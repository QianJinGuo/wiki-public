---
title: "Benchmarking Small LLM Inference on SageMaker：G7 vs G5/G6（30B MoE 跨代 GPU 评测）"
created: 2026-09-09
updated: 2026-10-26
type: entity
tags: [llm, inference, benchmark, moe, quantization, gpu, aws, inference-optimization]
sources: [raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026]
confidence: 0.7
provenance_state: extracted
---

# Benchmarking Small LLM Inference：G7 vs G5/G6

> AWS ML Blog 对两个代表性 30B MoE 模型跨 GPU 实例家族的基准评测（G7 Blackwell / G5 A10G / G6 L4 / G6e L40S），展示 G7 在吞吐/延迟/单 token 成本的可量化收益。核心价值在**跨代 GPU 选型的评测方法论与 MoE 推理硬件权衡**，而非平台绑定。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 为什么 MoE 推理对内存带宽敏感

MoE 架构在 token 生成（decode）阶段内存带宽受限：每个 token 只激活一小部分 expert，驻留权重远超单次激活量。**更高内存带宽直接降低 inter-token 延迟、提升吞吐**——这是选型判断比单纯看参数量更关键的维度。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## NVFP4 与低精度原生支持

- **NVFP4**：NVIDIA Blackwell 引入的 4-bit 浮点格式，Tensor Core 原生支持，权重压到约每权重 4 bit 且质量损失最小。
- **结构性优势**：只有 G7 有原生 FP4 Tensor Core；G5/G6 无硬件加速跑 NVFP4 权重，导致 G7 在 MoE 部署上有结构性优势。这是「量化格式 × GPU 代次」协同选型的典型案例。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 使用案例1：Qwen3-Coder-30B（编码助手）

同模型 + DJL LMI 28.0 容器 + 同 workload 对比 ml.g5/g6/g7.12xlarge（G5/G6 各 4 GPU×96GB，G7 2 GPU×64GB，**G7 用一半加速器和更少显存胜出**）：

| 指标 | G5 | G6 | G7 |
|---|---|---|---|
| 输出 token 吞吐 | 346.3 tok/s | 243.4 | **391.3** |
| 请求吞吐 | 2.69 req/s | 1.89 | **3.04** |
| 平均请求延迟 | 1475 ms | 2110 | **1316** |
| P99 延迟 | 1881 ms | 3316 | **1501** |

G7 vs G6：吞吐 +60.8%、平均延迟降 37.6%、P99 降 54.7%；流式场景 G7 中位 TTFT≈118ms、平均 ITL 8.9ms。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 评估方法论（可迁移的框架）

- **基准流程**：单模型 + 多实例 + 同 workload 的 apples-to-apples 对比；用 `Workload.synthetic` 定义 token 分布/并发/请求数；AIPerf 采集吞吐、请求延迟、P50/P90/P99；交互应用再测流式 TTFT 与 ITL。
- **结论皆 workload 特定**：结果只对所用模型/服务配置/token 分布/并发有效。生产应按期望流量特征度量。
- **推荐流程（生成式 AI Inference Recommendations）**：给定模型/候选实例/目标（吞吐优化），系统评测候选配置并返回**排名的成本-延迟-吞吐推荐**——比人工逐实例 benchmark 更接近真实生产最优。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 与 wiki 焦点框架的连接

属 [[concepts/inference-optimization|推理优化]] 与 [[concepts/moe-mixture-of-experts-2025|MoE]] 的实证层：把 [[concepts/harness-engineering-framework|Harness Engineering]] 的「评测要区分能力vs预算」思路落到推理基础设施选型（G7 用一半显存赢，正是 [[entities/agent-harness-evolution-from-llm-call-to-harness-tencent-2026|Harness 评测对照]] 里『少资源多办事』的硬件版）。与 [[entities/ai-infra-llm-efficient-inference-vllm|vLLM 高效推理]]、[[entities/750b-moe-pd-disaggregation-aws-efa-vs-roce|750B MoE 分载]]、[[entities/amazon-bedrock-multimodal-model-benchmark-gaming-qa-2026|Bedrock 多模态基准]] 互补——本篇聚焦「小模型（30B）+ 大批量承载」的低成本推理选型方法论。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 深度分析

### 一、内存带宽而非 FLOPS 才是小 MoE 解码的天花板

解码阶段每生成一个 token，都要把本次路由激活的 expert 权重从 HBM 完整读一遍；batch 小时算术强度极低，整个 kernel 落在 roofline 的带宽斜坡上而非算力平台上。30B-A3B 名义 30B 参数、每 token 只激活约 3B，但激活的是**分散在不同 expert 里的权重切片**，读取字节数仍远大于计算量——于是「权重字节 ÷ 有效 HBM 带宽」直接给出 ITL 的理论下界，实测值（G7 平均 8.9ms）与下界的差额才是 kernel 与调度的损失。

这解释了一个反直觉结果：G7 用 **2 张 GPU、64GB** 打赢 G5/G6 的 **4 张 GPU、96GB**。瓶颈既不在显存容量（装得下即可），也不在聚合算力（解码用不上），而在有效带宽与低精度路径。选型第一步因此不是比 TOPS，而是估带宽下界再看实测差多少：差距大说明该优化 serving 栈。

但这只是半场。prefill 是 compute-bound，长上下文时计算占比陡增：同一批配置下，512/256 的对话负载跑到 2,397 tok/s，3,500/200 的 RAG 负载掉到 816 tok/s，成本从 $0.90 涨到 $2.29/1M。input:output 比例决定了你站在带宽斜坡还是算力平台，而这两段的胜负手是不同硬件。容量仍是硬门槛（FP8 30B 要塞进 64GB 才使比较成立），batch 增大后带宽优势也会被摊薄。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

### 二、NVFP4 的收益边界：哪里真付账，哪里不付

NVFP4 是 Blackwell 引入的 4-bit 浮点格式，Tensor Core 原生支持。收益有三处，强度不同：**权重体积**压到约 4 bit/weight，同容量可装更大模型或留更多 KV 空间；**读取字节减半**，把上一节的带宽下界直接往下推——这是与小 MoE 解码同源的两倍级杠杆；**避免反量化开销**，原生路径省掉权重解包与格式转换。

不付账的地方同样明确。**精度**：原文只说 minimal quality loss，没有任何任务级证据——没有困惑度、没有代码通过率、没有长上下文保真度。量化误差对不同任务杀伤力差异极大，编码与检索问答恰是对数值扰动敏感的两类。**kernel 成熟度**：FP4 在 attention、MoE 路由、自定义算子上覆盖远不如 FP8 完整，某些栈会静默回落到 FP8/FP16，名义 4bit 却拿不到端到端收益。**路径错配**：G5/G6 能加载 NVFP4 权重但无硬件加速，这不是「慢一点」而是走错了路——量化格式与 GPU 代次必须成对选，参见 [[entities/quantization-techniques|量化技术]] 与 [[concepts/inference-optimization|推理优化]]。

判断规则可收敛成一句：只有当 serving 栈对该架构有原生 FP4 kernel、且自己的 eval 集通过时，FP4 才是真收益；否则 FP8 这条全代可用的路径更稳。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

### 三、怎么读 G7 vs G5/G6 的数字

第一个陷阱藏在基线里：**G6 比 G5 慢**（243.4 vs 346.3 tok/s）。「代次更新 = 更快」直接失效——同为 12xlarge，G6 是 4×L4，单卡带宽与算力都低于 A10G。SKU 名字对齐不代表资源对齐，跨代比较必须先拉平「GPU 型号 × 数量 × 显存 × 带宽 × 单价」五元组。基线选谁本身参与制造结论：对 G5 领先 13.0%，对 G6 领先 60.8%，而 13% 完全可能被一次 kernel 升级吃掉。

第二个陷阱是**把吞吐当成本**。Use case 2 内部就分裂了：g7.48xlarge 吞吐最高（2,397 tok/s），g7.2xlarge 单 token 成本最低（$0.90/1M）。并行效率不随卡数线性增长，单价却线性上涨。真正的决策量是 $/1M output tokens =（$/hr × 1e6）÷（tok/s × 3,600），吞吐只是它的分子之一，换算见 [[entities/how-to-calculate-the-inference-efficiency-ratio|推理效率比]] 与 [[concepts/ai-cost-optimization-framework|成本优化框架]]。

第三个陷阱是**跨 use case 相乘**。「吞吐高 60.8%」与「比 G6 便宜 4.9×」不是同一实验的两个切面：前者是 Qwen3-Coder FP8 + LMI + 并发 4 + 100 请求 + 128/128，后者是 Nemotron NVFP4 + vLLM + 并发 1 + 10 请求 + 512/256。模型、栈、并发、规格全变，只能做方向性归纳，不能算术合并。

第四个是**并发与尾部的敏感度**。Use case 1 是轻载短序列，G7 对 G5 的 13% 领先在此区间很薄；换到高并发或 prefill 更重的负载，4 卡聚合算力在 compute-bound 段可能反超。尾部信号更值得信：对 G5 平均延迟只降 10.8%，P99 降 20.2%——代际收益主要在排队与抖动，这正是 SLO 该盯的地方。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

### 四、评测方法论里真正可迁移的部分

- **单变量原则**：同模型、同容器（LMI 28.0）、同 workload，只让实例类型变。任何「换卡 + 换框架 + 换量化」一起上的对比都不可归因。
- **workload 显式参数化**：`Workload.synthetic(tokenizer, concurrency, request_count, prompt_input_tokens_mean, output_tokens_mean, streaming)` 把「你的流量」写成可复现参数；且至少跑两个 profile（512/256 对话、3,500/200 RAG），因为相对排序会随 input:output 比例变化——这是本次评测最有价值的设计。
- **指标分层不可互替**：吞吐（tok/s、req/s）→ 延迟分布（avg/P50/P90/P99）→ 交互指标（TTFT/ITL，需 `streaming=True`）。非流式测不出 TTFT，而 TTFT 才是「用户多久看到第一个字」；ITL 决定打字机是否流畅。
- **流程两分**：已有 endpoint 用 benchmarking 工作流；只有模型与候选时用 recommendation 工作流（给目标函数，返回排序配置）。本质是把选型从人工试错变成带目标函数的搜索，与 [[concepts/agent-evaluation-benchmark-frameworks|评测基准框架]] 同构：固定任务集、固定评分口径、只换被测对象。
- **该补而没报的**：warmup 与稳态判定、冷启动、多轮与前缀缓存命中率、扩容/故障行为、精度回归。这些缺口说明它是**性能 benchmark** 而非生产就绪评估。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

### 五、单一厂商自报基准的边界

AWS 卖 G7，结论必然是 G7 胜——全文没有一个 G7 落败或「某负载下 G5 更划算」的反例，单向结果本身该打折。可复现性倒是加分项：notebook 公开、参数全列、成本公式给出、区域也写了（G7 仅 us-east-2 / us-west-2 GA）。

但它仍是**单栈单区**结论：没有同价位非 AWS 方案对照；没有自建 vs 托管的 TCO（管理溢价没进 $/hr）；没有把 quota 与容量风险折价进去。而基线选择的可疑之处已在上节说明——一个比上代更慢的 G6 会系统性放大「跨代收益」的观感。

正确用法是把它当**假设生成器加方法论模板**，而非采购结论：数字只对它那套 workload 成立，可迁移的是流程（单变量、双 profile、三层指标、成本换算），不是排名。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 实践启示

1. **先算带宽下界，再谈换卡。** 用「激活权重字节 ÷ 单卡有效 HBM 带宽」估 ITL 下界并与实测对比：远高于下界说明瓶颈在 serving 栈，换卡无用；接近下界则换带宽更高的卡才有确定收益。
2. **用自己的并发与序列长度重跑，绝不引用别人的 tok/s。** 至少两组 profile（短对话 512/256、长上下文 RAG 3,500/200），并发覆盖峰值而非 4，请求数上千以支撑 P99。原文的并发 4 只能证明方向。
3. **以 $/1M output tokens 作决策指标，吞吐只当中间量。** 公式 ($/hr × 1e6) ÷ (tok/s × 3,600)。同家族内「吞吐最高」与「成本最低」通常不是同一规格（g7.48xl vs g7.2xl），按业务目标二选一，别默认选最大。
4. **量化上线前必须做精度回归。** NVFP4 的 minimal quality loss 没有任务级证据；在代码生成、检索问答上跑自己的 eval 集并设退化阈值，超阈值退回 FP8。
5. **买 Blackwell 前先确认 serving 栈有原生 FP4 kernel。** 否则 4bit 权重跑在无加速路径上，收益为负还多付反量化开销；量化格式与 GPU 代次要成对验证。
6. **按 P95/P99、TTFT/ITL 定 SLO，把「下一代会更快」当假设而非计划。** 代际收益集中在尾部与排队；同时核对目标区域是否 GA、quota 是否到位（G7 目前仅两个 Region），容量与刷新周期的不确定性要算进迁移成本。

→ [[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026|原文存档]]