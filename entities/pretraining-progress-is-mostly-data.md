---
title: "Pretraining Progress Is Mostly Data (Compute Efficiency by Axis)"
created: 2026-09-10
updated: 2026-11-18
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

## 深度分析

把六年进步拆成两条轴后，量级差距一目了然：数据轴累计增益约 12.0×，模型轴仅约 3.7×（相差 3.24 倍；对应年均 CEG 约 1.51× 与 1.24×）。更关键的是结构性发现：对 OLMES 得分做线性回归，88% 的方差可由模型效应与数据效应简单相加解释——两轴收益基本独立，预训练进步并不依赖"架构与数据的协同设计"，更像两条各自复利的独立曲线。但可加性有边界：作者实测的联合增益 1.57×/年是联合测量值而非两轴乘积，直接把 1.51× 与 1.24× 相乘外推会高估进步。^[raw/articles/pretraining-progress-is-mostly-data.md]

数据轴跑赢的原因是迭代成本的不对称。数据改进（从 OpenWebText 的约 9B tokens 到 UltraFineWeb 的全网抓取加分类器过滤）可以靠跑消融、看下游分来经验性快速验证，Greenblatt 指出这正是自动化 AI 研究最容易接管的改进形态；而模型轴多数创新（MoE、稳定化、FlashAttention）的真实贡献是让更大算力"变得可用"，而非同性能下省 FLOPs，在 CEG 口径下被系统性低估。这也解释了本文测得的 1.24–1.51×/年为何远低于 Ho 等人 3×/年的估计：差距主要来自规模依赖增益与推理侧优化（如 GQA）未被 ≤1e19 FLOPs 的小规模实验捕捉。^[raw/articles/pretraining-progress-is-mostly-data.md]

结论不能直接外推到前沿：实验规模封顶 1e19 FLOPs、评测锚定 OLMES。作者自己警告，小模型容量有限须"精选"，大模型容量过剩反而应"尽量多喂"，激进过滤加多 epoch 重放实证上更差，而前沿模型普遍超 Chinchilla 最优约 100× 训练——数据质量最优点随规模移动，1.51×/年在前沿可能缩水。Pile 异常值是同一教训的镜像：22 源多样性在英文 web-prose MCQ 上不讨喜，换更大规模或换评测可能反超——"什么数据好"本质是评测锚点的函数；且这些语料都是对同一份 Common Crawl 的策展，属有限存量消费（数据墙约束），与 [[entities/a-bitter-lesson-for-data-filtering-e8807d]]、[[entities/datacomp-for-language-models]] 处在同一条问题线上。^[raw/articles/pretraining-progress-is-mostly-data.md]

比数值更可复用的是误差处理范式：每个 scaling 点由多 seed 复跑取标准差，compute multiplier 的误差条来自对整条估计管线的 parametric bootstrap，外推得出的 NeoX/Pile 倍数被明确标注为额外误差源，作者并承认真实不确定性高于误差条（超参扫描不充分）。把不确定度一路传播到最终结论，使 scaling 声明变得可审计——这是该文除数字外的隐性方法论贡献。^[raw/articles/pretraining-progress-is-mostly-data.md]

## 实践启示

1. 算力预算固定时，优先把增量研发投入数据管线（抓取→清洗→过滤→配比）：六年 12.0× vs 3.7× 的差距说明数据工程是第一杠杆。
2. 改数据策略前先锁定评测锚点，并在至少一个 held-out loss 上复核：Pile 与 NeoX 的优劣结论会随 OLMES/held-out loss 的选择而反转。
3. 架构与数据投资近似可加（R²≈0.88），可并行推进、无需为特定语料定制模型；但不要按两轴相乘叠加预期收益。
4. 给模型侧创新计价时按"解锁了多大算力规模"而非"同性能省了多少 FLOPs"：稳定化、MoE、系统级优化多属前者。
5. 数据策略随模型容量分叉：小模型重质（精选过滤），大模型重量（广覆盖、少 epoch）；激进过滤加多轮重放在大算力与超训场景下实证为负。

## 关联

- 数据轴 vs 模型轴的 scaling 分解：[[entities/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|Scaling Domain Data Repetition]]
- 效率路径的战略含义：[[entities/magic-10x-more-efficient-pretraining|Magic 10x 高效预训练]]
- 总框架：[[concepts/scaling-laws|Scaling Laws]]

→ [[raw/articles/pretraining-progress-is-mostly-data|原文存档]]
