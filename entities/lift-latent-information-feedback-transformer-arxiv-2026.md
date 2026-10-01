---
title: "LIFT: Latent Information Feedback Transformer — 预训练期潜状态反馈架构"
created: 2026-10-02
updated: 2026-10-02
type: entity
tags: [transformer, architecture, pretraining, recurrent, state-tracking, arxiv]
sources: [raw/articles/arxiv-2609-38149-lift-latent-information-feedback-transformer]
confidence: 0.72
provenance_state: extracted
---

# LIFT: Latent Information Feedback Transformer

## 核心问题：Transformer 的前馈瓶颈

标准 Transformer LM 是纯前馈架构：深层表征从不反馈回浅层，跨生成步的信息下行通道只有已解码的 token 这一条窄带。这迫使模型反复重算中间结果、丢弃未选中的候选续写。LIFT（Latent Information Feedback Transformer）在预训练阶段移除这一瓶颈，使 LM 能跨生成步传播状态。^[raw/articles/arxiv-2609-38149-lift-latent-information-feedback-transformer.md]

## 方法：教师监督下的递归状态学习

LIFT 将递归状态学习转化为 teacher-forced 预测问题：每个输入 token 配对一个信息稠密的状态，该状态由现成预训练 LM 的 next-token 分布导出。模型（仅增加少量额外参数）被训练为同时预测下一个 token 和下一个状态。由于输入状态全部预计算，预训练在位置维度保持完全并行；推理时模型回喂自己预测的状态，计算开销随模型规模增大而递减。^[raw/articles/arxiv-2609-38149-lift-latent-information-feedback-transformer.md]

## 实验结果

- 135M–1B 参数规模的预训练实验中，LIFT 在 token-matched 预算下持续超越标准 Transformer 与基线（语言建模、下游推理、程序性任务），compute-matched 对比下持平或领先。^[raw/articles/arxiv-2609-38149-lift-latent-information-feedback-transformer.md]
- 状态追踪任务的受控研究：微型 LIFT 胜过用 8 倍数据训练的同尺寸 Transformer——即便其教师状态来自一个在该任务上失败的 Transformer。这表明模型可以学会利用 deep-to-shallow 反馈，而非仅复制教师状态。^[raw/articles/arxiv-2609-38149-lift-latent-information-feedback-transformer.md]

## 关联

- [[concepts/transformer-architecture-2025|Transformer 架构 2025]] — LIFT 是对标准前馈 Transformer 架构的直接修改
- [[entities/deepmind-recirculation-transformer-layer-activation-feedback-2026|DeepMind Recirculation 层激活反馈]] — 同为 layer-feedback 家族，Recirculation 在推理期做激活反馈，LIFT 在预训练期做潜状态反馈
- [[concepts/llm-pretraining-vs-sft|LLM 预训练 vs SFT]] — LIFT 的贡献在预训练方法层（teacher-forced 状态预测）

→ [[raw/articles/arxiv-2609-38149-lift-latent-information-feedback-transformer|原文存档]]
