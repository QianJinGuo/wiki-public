---
title: "Cerebras 万亿参数部署账本：44GB 片上 SRAM 的层映射、流水线与 KV cache 并发上限"
created: 2026-10-24
updated: 2026-10-24
type: entity
tags: [inference, hardware, wafer-scale, model-serving, kv-cache, quantization, pipeline-parallel, cerebras]
sources: [raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026]
confidence: 0.8
provenance_state: extracted
---

# Cerebras 万亿参数部署账本：44GB 片上 SRAM 的层映射、流水线与 KV cache 并发上限

> Cerebras 官方博客（2026-10-22，Sarah Chieng）以 Kimi K3（约 3T 参数、69 层线性注意力 + 24 层 dense = 93 层）为例，完整走了一遍"把万亿参数模型铺到晶圆级芯片上"的容量计算：单张 WSE-3T 晶圆 44 GB 片上 SRAM，4-bit 权重，8-bit KV cache，假设平均请求长度 256k，最终得到 **59 张晶圆**（35 张线性层 + 24 张 dense 层），并在多芯片流水线上跑到 **1000–2000+ tokens/s**。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

## 容量账本：从"3T 参数"推到"59 张 wafer"

第一步是单层容量：3T 参数全部按 4 bit 存放，原始权重载荷约 1.5 TB；除以 93 层，平均每层约 **16 GB**。第二步是每张 44 GB 晶圆能装几层——答案取决于层类型：^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

| 层类型 | 单层占用 | 每张晶圆层数 | 理由 |
|---|---|---|---|
| Dense（全注意力） | ~16 GB | **1 层** | 必须为每个活跃请求预留 KV cache 空间（每 token 的 K/V 都要存） |
| Linear attention | ~16 GB | **2 层**（2 × 16GB = 32GB） | 不需要按 token 存 K/V，每请求只保留固定大小状态，与上下文长度无关 |

因此 Kimi K3 需要 69 层线性注意力 ÷ 2 ≈ **35 张**晶圆，加上 24 层 dense 各占 1 张 = **24 张**，合计 **59 张 WSE-3T**。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

这套算法的关键不是"塞得下"，而是**不做模型并行切分**：因为单层 16 GB 小于单片 44 GB SRAM，整个层可以直接落在一张晶圆上。权重一旦加载就**不再移动**，跨晶圆只传激活值——通过 Ethernet 上的 RDMA，每跳约 **2 微秒**。这与 GPU 集群里"每层都要 all-reduce / 张量并行切分"的通信模型是根本区别。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

## 流水线：批大小不是 1，而是"阶段数 × 微批"

按层切到 59 张晶圆之后，每张晶圆就是流水线上的一站：wafer 1 跑第 1–2 层，把激活传给 wafer 2，依此类推直到最后一站产出下一个 token。常见误解是"一个 token 要顺序穿过所有站，那每站不都在空等、等价于 batch size = 1"——实际上流水线被**不同请求**填满：stage 3 处理请求 A 的 token 时，stage 2 处理请求 B，stage 1 处理请求 C。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

在途请求数量 ≈ **阶段数 × 微批大小**。生产环境用的阶段比"一层一站"细粒度高得多、站数也多得多。微批的权衡很直接：微批越小单请求延迟越低，微批越大总吞吐越高。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

## 并发天花板由 KV cache 决定

并发上限不是算力，而是内存：每个在途请求都要在**每张 dense 层晶圆**上占一份自己的 KV cache。每张 44 GB 晶圆装下一个 16 GB dense 层后，剩余约 **28 GB** 留给 KV cache；用单请求 256k 上下文所需的 cache 大小去除，就得到该映射下支持的并发请求数。这些请求不像 GPU 的静态 batch——它们不等彼此完成，而是持续流式穿过流水线。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

作者强调并发数和 batch size 都不是效率指标：评估推理系统时更有意义的度量是**在相同输出速度下，单位功耗的 token 吞吐（tokens/s per megawatt）**。这一点在超长上下文 + 大模型场景尤其重要，因为 KV cache 的 SRAM 占用会同时压低并发和能效。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

## 与 GPU 部署范式的差异

| 维度 | Cerebras 晶圆级 | 典型 GPU 集群 |
|---|---|---|
| 权重位置 | 常驻片上 SRAM，不移动 | 显存常驻，但张量并行需跨卡通信 |
| 跨设备流量 | 仅激活值（~2μs/跳） | 激活 + 梯度/规约（all-reduce 等） |
| 层切分 | 层内不切（单层 < 单晶圆） | 常需层内张量并行切分 |
| 并发约束 | 每张 dense wafer 的 KV cache 余量 | 显存总量 / 分页 KV cache 管理 |
| 扩展方式 | 加晶圆 → 加算力 + 加片上内存 | 加卡 → 需重构并行策略 |

对推理架构师的直接含义：这类硬件的容量规划是"**先算权重载荷 → 再算单层尺寸 → 再算 KV 余量 → 最后得到并发**"的四步解析式，而不是 GPU 上靠经验配比的显存余量管理；反过来说，长上下文（256k 级）请求会先把 dense 层晶圆的 KV 余量吃满，成为比算力更早触顶的瓶颈。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

## 深度分析

### 一、容量账本压在三个可被推翻的假设上

3T 参数按 4 bit 存放约 1.5 TB，除以 93 层得每层约 16 GB——这条除法是推演起点，其实是三个假设的乘积。第一是位数：4 bit 才让单层落进 44 GB 晶圆；换 8 bit，线性层从"两片一层"退化成"一片一层"，晶圆数从 59 跳到 93。量化在这里的意义不是几倍吞吐，而是映射是否成立的开关。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

第二是层间容量齐整：16 GB 只是均值，编译器拿到的是逐层实际张量尺寸。第三是 dense 与 linear 的 24:69 结构比，59 由它直接决定——若模型全为全注意力层，晶圆数就等于层数，且每片都被 KV 挤压，那不是更贵，而是并发趋零。

### 二、不做张量并行，靠"单层小于单卡"这个不等式

GPU 上服务万亿参数，层内张量并行几乎是默认动作：一层放不进一张卡就得切开权重，此后每层前向都要跨卡规约，通信量与被切矩阵同阶。晶圆级把这条因果链从源头掐断——44 GB SRAM 大于任何单层 16 GB，一层完整落在同一片晶圆上，层内没有需要跨设备求和的维度，切分策略、通信重叠、拓扑感知一并失去对象，与 [[entities/deepseek-moe-parallel-strategy]] 里按层按专家切分的取舍恰成对照。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

由此权重静态、只有激活跨片，RDMA over Ethernet 约 2 微秒一跳；59 站串起来，累计跳延迟仍在微秒量级，与跨节点 all-reduce 的百微秒不在一个问题上。代价是用硅片面积买来的简化：单层一旦越过 44 GB，层内切分就会回流，方案退化为"晶圆级 + 张量并行"。

### 三、流水线的 batch 是"阶段数 × 微批"

"token 顺序穿过 59 站，每站都在空等"错在把流水线当串行批处理：第 3 站在算请求 A 时，第 2 站在算 B，第 1 站在算 C。在途请求数约等于阶段数与微批之积；生产环境阶段比"一层一站"细得多、站数也多，这就是 [[entities/llm-inference-pipeline-internals]] 所说"流水线要被多请求填满"的容量基础。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

两个旋钮方向相反：微批小则时延低，微批大则吞吐高而单请求慢；batch 在这里是调度结果而非预设参数——GPU 侧靠调度填气泡，这里的气泡被"不同请求占不同阶段"天然填掉。

### 四、天花板在 KV cache，而不在算力

每张 dense 晶圆装下 16 GB 层后剩约 28 GB，这就是并发预算：每个在途请求都要在每张 dense 晶圆上占一份 KV，按 8 bit、256k 平均上下文计的占用可观，28 GB 除以它即得并发数。约束由 24 张 dense 晶圆共同给出，与总算力无关。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

反直觉之处在于：dense 层绝对条数比总参数更能预测并发。线性注意力每请求只留固定状态、与上下文无关，是免费容量；于是上下文由 32k 涨到 256k，KV 占 8 倍、并发降到八分之一，权重侧分文未省。作者还纠正了评价口径：并发与 batch 都不是效率指标，可比的是同速下的 tokens/s per MW。

### 五、这套账本的迁移边界

四步解析式可搬，不可搬的是三个条件：单层小于单片 SRAM、模型能切成足够多站、负载能提供足够多在途请求。前两条由硬件与架构给出，第三条由负载给出——低并发强交互下，59 站会因填不满而退回低效区间。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]

故选型是三集合的交集：单层可控、dense 占比够低、负载能提供高并发或超长上下文。三者齐备时，解析式规划胜过 GPU 上凭经验配比显存；缺一条，账本上漂亮的数字就会在真实负载下缩水。[[concepts/attention-mechanism]] 向线性与稀疏化演进使 dense 占比走低，对晶圆级长期有利。

## 实践启示

1. 先把容量账算成解析式再比硬件：权重载荷 → 单层尺寸 → 单卡可容纳层数 → KV 余量 → 并发；任一环假设变化都会放大到最终数字。
2. 把"最宽那层能否整体放下"当硬门槛：放得下，层内并行与通信开销整体消失；放不下就要备齐张量并行全套工程。先量最宽层，不看总参数。^[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026.md]
3. 用 dense 层条数而非总参数量预估并发：线性与稀疏层几乎不耗 KV 预算，dense 才是收费口。
4. 把平均上下文长度当一阶容量参数：256k 对 32k 是 8 倍 KV 与 8 倍并发折损；若上下文还要变长，预先规划 [[entities/750b-moe-pd-disaggregation-aws-efa-vs-roce]] 那类 prefill/decode 分离路径。
5. 先看负载能否填满流水线：效率来自"阶段数 × 微批"的在途请求，低 QPS 或强交互会让站空转；先看 QPS 分布与 P99。

## 关联

- 推理与部署：[[concepts/inference-optimization]]、[[entities/ai-infra-llm-efficient-inference-vllm]]、[[entities/agent-assisted-sglang-development-lmsys-2026-07]]
- KV cache 与量化：[[entities/deepseek-v41-flash-kv-cache-compression-limit-agiyanxishe-2026-09-22]]
- 投机解码等服务优化：[[entities/approximate-speculative-decoding-asd-relaxed-validation-2026]]

→ [[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026|原文存档]]
