---
title: "DeepMind Recirculation：冻结权重、深层激活回流释放性能"
created: 2026-08-21
updated: 2026-09-07
type: entity
tags: [transformer, architecture, inference, activation-feedback, deepmind, paperweekly, arxiv]
sources: [raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# DeepMind Recirculation：冻结权重、深层激活回流释放性能

> **Background**：PaperWeekly 对 DeepMind 论文《Recirculation》（arXiv:2608.17981）的解读。DeepMind 证明优化大模型未必需要堆参数——在推理阶段打通一条信息回路，让深层已形成的状态重新进入后续计算，即可释放新的性能空间，且基础版本无需修改原模型权重。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

## 核心洞见：深层状态无法被后续 token 复读是结构瓶颈

Transformer 的前馈结构决定了一个长期限制：语义消歧和状态更新往往要到较深的 layer 才逐渐稳定下来，但浅层计算不会重新读取这些后来形成的信息，后续 token 因而很难持续利用已在深层完成更新的状态。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

- 上下文语义消歧案例：含 `fishing pole` 的上下文中，模型在较深 layer 已把 `bank` 正确理解为"河岸"，但继续问附近有无 ATM 时仍会受 `bank`-金融机构强关联影响。此前研究通过激活干预，将已消歧的深层表示重新注入浅层，这类上下文化错误减少约 60%。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]
- 与 CoT 的区别：CoT 把中间结果显式写入 token 序列增加串行计算步骤，更适合复杂推理；Recirculation 关注的是内部状态本身如何在后续 token 中持续更新。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

## 机制：深浅层回流通路

基础 Recirculation 计算不复杂：处理当前 token 时，从较深的源层 (s) 读取激活，与较浅目标层 (d) 的原有状态混合，默认用凸组合。由于不同 layer 残差流尺度不一致，源层激活先对齐到目标层 L2 范数再相加。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

- Gemma3 1B/4B/12B 的较优源层→目标层组合分别是 11→4、18→9、35→16。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]
- 与 Looping（层循环，沿深度重复执行若干层）不同，Recirculation 同时跨越网络深度和 token 时间，同一 layer 可持续承载更新后的状态。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

## 实验结果

- Gemma3 1B/4B/12B 测试 10 个数据集，Recirculation 在其中 9 个上表现跨规模稳定改善；12B 的 PG19 困惑度下降 35.40%、BookSum 下降 32.91%、GovReport 下降 30.74%（Lambada 是主要例外）。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]
- 最稳定收益集中在语言建模；下游任务效果对具体设置更敏感。Qwen3 1.7B、Pythia 1B、Ministral3 3B、Phi2 2.7B 五家族均观察到有效源层—目标层组合。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]
- 回流收益随 token 间距衰减，但相隔 256 token 时仍可测到；副词/形容词/动词受益较多，数词/限定词/代词较少。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]
- 指令遵循：Gemma3 4B 准确率 82.3%→87.3%，12B 93.0%→98.4%。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]
- 工程代价主要在 Prefill（需按 token 顺序更新回流状态，长上下文拖慢）；自回归生成时两套 Transformer 计算可并行，额外生成延迟几乎可忽略。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

## Adaptive Recirculation：小模块反超全量微调

固定系数对所有 token/隐藏维度用相同控制策略，但不同 token 需融合程度不同。研究训练一个小型 MLP，根据当前 token 在源层与目标层的表示动态生成逐维回流系数（Gemma3 主体始终冻结，只训练控制模块）。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

- 逐维控制优于标量控制，token 条件动态生成又优于固定参数。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]
- Adaptive Recirculation 在 9 个语言建模数据集平均困惑度降幅 23.0%；固定基础版 8.5%；对加入回流结构的 Gemma3 1B 全量微调 21.6%。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]
- GSM8K 数学推理：Gemma3 4B pass@1 从 29.3%→35.5%（+21%），pass@128 94.9%→96.0%。效果依赖 MLP 控制器训练数据分布。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

## 意义

Recirculation 不是可直接替代微调的通用方案，但证明一个可能：冻结模型权重后，改变内部状态传播方式同样能释放新性能空间。与同属"垂直反馈/信息回路"家族但机制不同的 [[entities/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888|Full-Bandwidth Transformer]]（GLU latent feedback + scheduled multi-pass 训练）互补。^[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026.md]

## 相关实体

- [[entities/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888|Full-Bandwidth Transformer]] — latent feedback 拓宽垂直反馈通道（同家族不同机制）
- [[entities/discoformer-density-score-transformer-allen-ai|DiscoFormer]] — 密度评分稀疏化 transformer
- [[entities/topological-trouble-transformers-state-tracking-deepmind-2026-06-17|Transformer 状态追踪]]
- [[entities/attention-only-transformers-controlled-study-sans-arxiv-2607-18363|Attention-only transformer 对照研究]]
- [[entities/deepmind-ai-pointer|DeepMind]]

→ [[raw/articles/deepmind-recirculation-transformer-layer-activation-feedback-2026|原文存档]]
