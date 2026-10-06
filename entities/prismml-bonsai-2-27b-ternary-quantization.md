---
title: "Introducing Bonsai 2 27B: Near-Lossless Ternary Compression"
created: 2026-09-22
updated: 2026-10-06
type: entity
tags: [quantization, inference, efficiency, model]
sources: [raw/articles/prismml-bonsai-2-27b-ternary-quantization]
confidence: 0.7
---

# Introducing Bonsai 2 27B: Near-Lossless Ternary Compression

## 摘要

PrismML 基于 Qwen3.8 27B 的端到端三值量化（三值权重 + FP16 group scaling，1.76 有效 bits/weight，5.9GB），benchmark 保持 98.2%——近无损 9x 压缩的实证^[raw/articles/prismml-bonsai-2-27b-ternary-quantization.md]

## 核心要点

- v*c=49（value=7, confidence=7, stars=3），newsletter ingest 2026-09-22
- 详细分析见原文存档

## 深度分析

### 三值表示 + FP16 group scaling 的工程取舍

Ternary Bonsai 2 27B 的核心技术选择是把权重限制在 {−1, 0, +1} 三个值上，再以 FP16 group-wise scaling 恢复精度。1.76 effective bits/weight 意味着 27B 参数模型只需 5.9GB——比全精度小 9 倍以上。关键在于 low-bit 表示是端到端应用于整个语言模型，而不是只压缩部分层；这避免了混合精度方案中精度层成为 memory bottleneck 的问题。FP16 scaling 以组为单位共享，说明作者认为三值化后逐权重的幅度信息可以由小组共享的 scale 承担，这是"量化误差可控"的典型设计假设^[raw/articles/prismml-bonsai-2-27b-ternary-quantization.md]

### 从 95% 到 98.2%：跨过"近无损"门槛

对比上一代 Ternary Bonsai 27B 的 95% retention，Bonsai 2 把这个数字推到 98.2%（aggregate benchmark 得分 83.9，覆盖 reasoning、math、coding、instruction following、vision 与 agentic tool use）。PrismML 自己的判断是：到了这个水平，压缩已经"practically lossless"——量化不再被视为能力换体积的妥协，而是部署前提。两代之间 retention 提升 3+ 个百分点，主要驱动力是更强的 base model（Qwen3.8 27B）加上压缩流程本身的改进，这说明 base model 的鲁棒性直接决定了量化的能力天花板^[raw/articles/prismml-bonsai-2-27b-ternary-quantization.md]

### retention 的分布比 aggregate 更关键

原文特别强调：真正重要的不是 aggregate 分数，而是能力保留在哪些区域。Coding agents、tool-use 系统、multimodal workflow、long-horizon 任务对模型退化最敏感——因为小错误会在多步执行中 compounding。Bonsai 2 恰好在这些高敏感区域保留了大部分全精度性能，这正是它能支撑 Cline coding agent 与 computer-use demo 的原因。反观许多 low-bit 替代方案，往往是在 coding、vision 或 agentic tool use 上付出实质性代价后才变得可部署——aggregate 分数接近，敏感能力却已受损。这提示评估量化模型时必须按能力类别分解，而非只看一个总分^[raw/articles/prismml-bonsai-2-27b-ternary-quantization.md]

### intelligence density 与能耗：部署经济学的新维度

Bonsai 2 在同参数量级模型中呈现出 per-GB intelligence density 的 outlier 表现。吞吐方面：RTX 5090 上最高 143 tokens/s，M5 Max 上 46.8 tokens/s；能耗方面，RTX 4090 上 0.714 mWh/token，比全精度 8B 模型省电 40%。一个 27B 模型比 8B 模型更省电，说明量化改变了"模型大小 vs 能耗"的传统换算——memory bandwidth 是本地推理的主要瓶颈，权重越小带宽需求越低。PrismML 由此提出一个更一般的评价框架：问题不再是"模型有多强"，而是"给定 memory、compute、power 预算能交付多少有效智能"^[raw/articles/prismml-bonsai-2-27b-ternary-quantization.md]

## 实践启示

1. **评估量化模型先看敏感能力，再看 aggregate**：选型时应单独测 coding agent 循环、tool use、长任务等 compounding-sensitive 场景，aggregate 98% 不代表每个能力维度都近无损。
2. **5.9GB / 262K context 的组合适合"本地主力 + 云端升级"架构**：把高频、敏感或离线任务放在本地跑 Bonsai 2 级别的模型，仅在需要时升级到云端，可显著降低成本与延迟。
3. **量化收益随 base model 变强而放大**：更强、更鲁棒的 base model 在同样压缩流程下 retention 更高（95% → 98.2%），做量化部署时应优先考虑最新最强的开源底座。
4. **本地 agent 的吞吐瓶颈是 memory bandwidth 而非算力**：三值/低比特 kernel（CUDA + MLX 双平台支持）是解锁 143 tok/s 的关键，自建本地推理栈时应优先寻找针对低比特权重的定制 kernel 而非通用 FP16 引擎。
5. **能耗可以作为本地常驻 agent 的硬约束来设计**：0.714 mWh/token 意味着后台常驻的 assistant 在笔记本/移动设备上电池可持续，规划 always-on 本地推理时可直接用 mWh/token 做容量预算。
6. **Apache 2.0 开源权重降低了复现门槛**：结合完整 whitepaper 的压缩与评测流程，团队可以以它为 baseline 快速验证自家领域数据 post-training 后的量化鲁棒性。

相关：[[raw/articles/bonsai-image-4b-1-bit-ternary]]

→ [[raw/articles/prismml-bonsai-2-27b-ternary-quantization|原文存档]]
