---
title: "Full-Bandwidth Transformer：Latent Feedback 拓宽解码垂直反馈通道"
created: 2026-08-17
updated: 2026-09-07
type: entity
tags: [transformer, architecture, latent-feedback, arxiv, autoregressive, efficiency]
sources: [raw/articles/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Full-Bandwidth Transformer：Latent Feedback 拓宽解码垂直反馈通道

> **Background**：arXiv:2608.08888（2026-08-09 提交，Xi Wang 等 8 人，含 John Langford）。自回归 Transformer 架构侧的解码反馈带宽补全研究——将顶层隐状态经 GLU 融合回注入输入，让非言语化计算重新进入计算栈。^[raw/articles/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888.md]

## 核心洞见：垂直反馈带宽是自回归架构的隐藏瓶颈

自回归 Transformer 的两条计算轴：**水平方向**（生成的 token 序列，dense attention 提供对过去的宽接入）与**垂直方向**（模型深度）。垂直反馈通道在解码步之间是狭窄的——只有采样 token 回到栈底，而顶层隐状态被丢弃。这意味着模型「想清楚了但没说出口」的部分（latent computation）无法回流参与后续推理。^[raw/articles/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888.md]

full-bandwidth transformer 用 **latent feedback** 拓宽该通道：每个解码步，上一层的顶层隐状态与采样 token embedding 经门控线性单元（GLU）融合，作为下一输入反馈回栈底。非言语化的计算得以带着「更新的深度预算」重新进入计算栈，同时保持标准 Transformer 架构、KV cache 与 LM 目标完全不变。^[raw/articles/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888.md]

## 训练方法与实验

- **Scheduled multi-pass objective**：为不损失并行 teacher forcing，预训练后期才引入 latent feedback，并混合一小部分更深反馈 pass 保证稳定性。^[raw/articles/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888.md]
- **规模**：1B 参数训练至 400B tokens。^[raw/articles/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888.md]
- **结果**：latent feedback 改善验证损失、5-shot LM 评估、数学与代码生成、指令微调性能；以可忽略的每 token 解码开销，达到或接近用约 1.5× 更多 token 训练的标准 Transformer，并产生更短推理轨迹（同等或更好精度）。^[raw/articles/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888.md]

## 与既有研究的关系

- 与 [[entities/topological-trouble-transformers-state-tracking-deepmind-2026-06-17|Transformer 状态跟踪局限]] 同属「修改自回归架构以补足机制能力」家族，但本作从反馈带宽（垂直流）切入而非状态表示。
- 与 [[entities/attention-only-transformers-controlled-study-sans-arxiv-2607-18363|Attention-only Transformer 受控研究]] 同属「给 Transformer 变形并实证」的方法论族——本作保留标准 attention，只改反馈通道。
- 与 [[concepts/attention-mechanism|注意力机制]] 概念页互补：attention 决定水平接入，latent feedback 决定垂直回流。

## 实践启示

推理效率视角：缩短 reasoning trace 而不损失精度，对长推理链 Agent/思维链场景有直接价值——模型可以把「想的过程」留在隐状态流中，而不必全部言语化。训练效率视角：1.5× token 等价收益意味着同样预算下可用更小模型达到相近能力。

→ [[raw/articles/full-bandwidth-transformer-latent-feedback-arxiv-2608-08888|原文存档]]