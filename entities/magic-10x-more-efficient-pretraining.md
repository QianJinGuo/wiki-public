---
title: "Magic 10x More Efficient Pretraining (Magic Lab)"
created: 2026-09-10
updated: 2026-09-10
type: entity
tags: [pretraining, scaling-laws, training-efficiency, llm, magic, frontier-lab]
sources: [raw/articles/magic-10x-more-efficient-pretraining]
confidence: 0.7
---

# Magic: 10x More Efficient Pretraining

Magic（2026-09-08 研究更新）宣称其预训练配方已实现比主流开源 base model 高 >10x 的算力效率：以约 DeepSeek V4 Pro Base 1/50 的 FLOPs 匹配其能力（约等于 GPT-3 预训练算力的一半，GB200 上约 $0.5M），继续 scale 10x（~$4M）后在 perplexity evals 上明显超越所有公开开源 base model。按图 1 的 scaling laws，同能力在 DeepSeek V4 Pro 配方下成本 >$100M。^[raw/articles/magic-10x-more-efficient-pretraining.md]

## 方法论可信度信号

文章给出多个支撑其效率声明的工程细节：167 个域上的 scaling-law 拟合、每次改动 3 个模型的多尺度评估、heldout 去污（decontamination）协议、以及跨引擎 logprob 验证（防止评估引擎差异污染对比）。模型与评测均为私有，无法独立复现——这是实验室研究公告与可复现论文之间的固有差距。^[raw/articles/magic-10x-more-efficient-pretraining.md]

## 战略定位

Magic 的路线声明是"预训练 + agentic RL + long-context 足以构建超人 coding agent 并自动化 AI 研究"，本文聚焦预训练一环。其效率优先路径（算法效率替代 10 万卡集群）对算力受限实验室有参考意义，与 [[concepts/scaling-laws|Scaling Laws]] 的"算力即一切"叙事形成对照：效率改进可以数量级地移动成本曲线。^[raw/articles/magic-10x-more-efficient-pretraining.md]

## 关联

- 预训练数据/规模研究：[[entities/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|Scaling Domain Data Repetition]]
- 预训练 vs 后训练分工：[[concepts/llm-pretraining-vs-sft]]
- 数据 vs 模型轴的效率分解：[[entities/pretraining-progress-is-mostly-data|Pretraining Progress Is Mostly Data]]

→ [[raw/articles/magic-10x-more-efficient-pretraining|原文存档]]
