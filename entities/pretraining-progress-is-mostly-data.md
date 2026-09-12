---
title: "Pretraining Progress Is Mostly Data (Compute Efficiency by Axis)"
created: 2026-09-10
updated: 2026-09-10
type: entity
tags: [scaling-laws, pretraining, data-quality, compute-efficiency, empirical]
sources: [raw/articles/pretraining-progress-is-mostly-data]
confidence: 0.7
---

# Pretraining Progress Is Mostly Data

一篇实证分析（Dwarkesh 频道发布，2026）：把预训练 compute-efficiency gain（CEG）逐年按 model 轴与 data 轴分解，发现 2019-2025 年间 data 轴效率提升（~1.51×/年）显著快于 model 轴（~1.24×/年）——预训练"进步"主要来自数据而非架构。结论指向：当算力固定时，数据配比/过滤/课程的投资回报高于模型结构改动。^[raw/articles/pretraining-progress-is-mostly-data.md]

## 方法与证据

- 每个 scaling curve 数据点来自多次独立 seed 的训练 run，误差条用 parametric bootstrap 计算
- 在 held-out 的 FineWeb-Edu 预训练 loss 上复核（防 eval 集污染）
- 异常值分析：NeoX 在 OLMES 上 1e19 处差于 GPT-2（但 FineWeb-Edu held-out loss 更好，疑似 OLMES 噪声）；The Pile 在 OLMES 上差于 OpenWebText——作者解释为 The Pile 的 22 源多样性（PubMed/arXiv/GitHub code/法律/专利等）跨域迁移到英文 web-prose MCQ 有限，且规模更大时可能反超^[raw/articles/pretraining-progress-is-mostly-data.md]

compute multiplier 的获取方式为外推，作者明确标注了由此引入的额外误差——对 scaling 外推的诚实性是该文的隐性方法论贡献。^[raw/articles/pretraining-progress-is-mostly-data.md]

## 关联

- 数据轴 vs 模型轴的 scaling 分解：[[entities/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|Scaling Domain Data Repetition]]
- 效率路径的战略含义：[[entities/magic-10x-more-efficient-pretraining|Magic 10x 高效预训练]]
- 总框架：[[concepts/scaling-laws|Scaling Laws]]

→ [[raw/articles/pretraining-progress-is-mostly-data|原文存档]]
