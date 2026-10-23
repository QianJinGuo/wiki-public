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

## 关联

- 推理与部署：[[concepts/inference-optimization]]、[[entities/ai-infra-llm-efficient-inference-vllm]]、[[entities/agent-assisted-sglang-development-lmsys-2026-07]]
- KV cache 与量化：[[entities/deepseek-v41-flash-kv-cache-compression-limit-agiyanxishe-2026-09-22]]
- 投机解码等服务优化：[[entities/approximate-speculative-decoding-asd-relaxed-validation-2026]]

→ [[raw/articles/cerebras-trillion-parameter-wafer-scale-deployment-2026|原文存档]]
