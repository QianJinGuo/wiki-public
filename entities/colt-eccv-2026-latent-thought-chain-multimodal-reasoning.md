---
title: "CoLT (Chain of Latent Thoughts): ECCV 2026 — 3步潜思维链加速多模态推理20+倍"
created: 2026-07-15
updated: 2026-09-07
type: entity
tags: [eccv-2026, latent-reasoning, multimodal, chain-of-thought, efficiency, mllm]
sources: [raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# CoLT (Chain of Latent Thoughts): ECCV 2026 — 3步潜思维链加速多模态推理20+倍

> **Background**: 本文基于机器之心对 ECCV 2026 论文 CoLT 的报道，该系统由南洋理工大学、上海交通大学、南开大学、天津大学等机构联合提出，首次系统性地将文本思维链替换为仅 3 步连续潜向量，实现多模态大模型的高效推理。^[raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning.md]

## 核心思想

CoLT（Chain of Latent Thoughts，潜思维链）将多模态大模型（MLLM）推理中的显式文本 CoT 替换为仅 K 步潜思维向量（默认 K=3），每个向量对应语言模型在指定推理位置的最后一层隐状态，直接作为下一步的输入嵌入反馈回模型，无需扩展词表或引入特殊 token。^[raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning.md]

## 效率提升

相比 Text CoT 平均约 142 个推理 token，CoLT 仅需 3 个连续向量即可完成思考，生成阶段耗时从约 7.24 秒降至 0.32 秒（MMStar benchmark，单卡 H200），文本解码加速达 22.6x，端到端推理加速 10.1x。^[raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning.md]

在八个多模态基准上的平均准确率达到 79.1%，超越同骨干上的 Text CoT（75.7%，+3.4%），也显著领先现有潜推理与潜视觉推理方法。^[raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning.md]

## 三重步级监督机制

无约束的潜空间缺乏结构化归纳偏置，模型极易学到语义空洞的表征。CoLT 引入三重监督机制：^[raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning.md]

- **外部前向解码**：以当前潜思维 h_k 为条件，小解码器自回归生成下一步文本推理，确保每个潜向量编码足够信息。
- **外部后向解码**：以前序文本推理的隐状态与潜向量做方向对齐（归一化余弦距离），与前向形成闭环。
- **内部步间连贯性预测**：由 h_k 预测下一步表征，以余弦相似度约束步间逻辑递进。

三者共同构成完整训练目标，从内容与结构两个维度规约潜空间。^[raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning.md]

## 论文信息

- 论文：CoLT: Teaching Multi-Modal Models to Think with Chain of Latent Thoughts
- 会议：ECCV 2026
- 机构：南洋理工大学 | 上海交通大学 | 南开大学 | 天津大学
- 代码：https://github.com/hulianyuyy/CoLT
- arXiv：https://arxiv.org/abs/2606.31986^[raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning.md]

## 相关实体

- [[entities/laser-acl2026-latent-superposition-visual-reasoning|LASER: ACL 2026 视觉推理潜叠加]]
- [[entities/flat-feedforward-latent-triangle-splatting|Flat Feedforward Latent Triangle Splatting]]

→ [[raw/articles/colt-eccv-2026-latent-thought-chain-multimodal-reasoning|原文存档]]
