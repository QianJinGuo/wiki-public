---
title: "Scaling Domain Data Repetition in LLM Pretraining — 域数据重复率随规模扩展"
created: 2026-08-19
updated: 2026-11-18
type: entity
tags: [llm, pretraining, data-repetition, data-scaling, tokens-per-parameter, training, ai2, domain-data]
sources: [raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Scaling Domain Data Repetition in LLM Pretraining — 域数据重复率随规模扩展

## 概述

arxiv 2608.14071（2026-08-14 提交，Jingwei Li 等 8 位作者）研究 LLM 预训练中**域数据重复**（domain data repetition）如何随模型规模扩展。核心背景：LLM 规模扩大时训练 token 预算必须同步增长以维持合适的 tokens-per-parameter（TPP）比，但**高质量域数据远比通用网页数据难扩展**——这使「重复使用稀缺域数据」成为预训练的关键权衡。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md]

论文聚焦一个反直觉的核心发现趋势：最优重复率如何随模型规模 / 训练 token 预算的变化而缩放，为「数据重复多用」这一业界常见但缺乏理论锚定的做法提供实证尺度。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md]

## 深度分析

### 问题设定：TPP 约束下的域数据稀释与重复权衡

论文的出发点是一个规模化的结构性矛盾：模型规模扩大时，训练 token 预算必须同步增长才能维持合适的 tokens-per-parameter（TPP）比；但高质量域数据的扩展难度远高于通用网页数据，于是随着模型与 token 预算一起变大，域数据在训练混合中的占比趋于下降。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 这与 Chinchilla 传统里「token 预算跟着参数走」的配比逻辑一致，只是作者们把矛盾推进一步：分母（参数）可以按算力规划扩张，分子（高质量域 token）却受制于现实世界的供给侧，稀释因此是规模化的内生现象，而非数据工程失误。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 论文检验的对冲手段是最直接的重复使用：重复能对抗稀释，但过度重复会滑向过拟合——于是问题被精确化为「在 token 预算随模型大小成比例增长的实际扩展路径下，每个域的最优重复次数如何随规模缩放」。这一实验条件设定使结论直接对应真实预训练决策，而非固定算力下的抽象最优；也让该工作与 [[concepts/scaling-laws|Scaling Laws]] 的 proxy-model 方法论同属一个谱系，只是被缩放的对象从损失换成了重复次数。论文作者为 Jingwei Li、Xinran Gu 等 8 人（2026-08-14 提交，cs.AI）。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:19-41]

### 发现一：固定 TPP 下最优重复次数随模型规模温和上升

核心结果之一出人意料：对固定域，在固定 TPP 的条件下，最优重复次数随模型规模的增大而温和上升——论文自述这一方向性结果 "surprisingly"。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 直觉上更大的模型反复见识同批数据似乎更易过拟合，该结果说明在 TPP 被钉住的设定里恰恰相反：容量增长带来的表征与优化冗余足以「消化」更多次的重复曝光，边际上多重复一轮的过拟合风险低于其抗稀释收益。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 「温和」（mildly）一词是刻意的限定：它既否定了「重复率应随规模不变」的静态直觉，也否定了「规模越大越该翻倍重复」的激进外推——摘要只给出方向性结论，并未给出可精确计算的显式标度律形式，工程上不宜把它读成一条公式。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 这与 [[entities/generalization-dynamics-lm-pretraining|LM 预训练泛化动力学]] 刻画的泛化-记忆张力是同一枚硬币的两面：规模同时改变了记忆倾向与表征容量，此处的净效应是重复容忍度被推高了一格。

### 发现二：域验证损失强负相关，唯一数据量仅弱相关

跨域比较给出第二个、也是更具操作价值的规律：最优重复次数与该域的最终验证损失强负相关——损失更低的域可以从更多次重复中持续获益，损失偏高的域则更早触顶。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 与之形成鲜明对照的是，一个域唯一数据的绝对数量与最优重复次数只有弱相关。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 这直接否定了「数据集越大越该多重复」的朴素直觉：主导变量是域数据的可学性（以验证损失为代理），而非其体量。对多域混合预训练而言，这意味着重复预算应当按域逐一校准，而不是按数据规模统一摊派——同一个 4-epoch 策略在低损失域和高损失域上会分别落在欠重复与过重复的两侧。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 使用这一定律时要留意代理变量的边界：验证损失同时受数据质量与评测集构成影响，跨域比较时损失口径需保持一致，否则「强负相关」可能被评测集差异污染。该规律与 [[concepts/moe-mixture-of-experts-2025|MoE 混合专家]] 按容量分而治之的思路形成互文——两者都拒绝「一刀切」的资源分配，把异质性当作一等公民来建模。

### 方法论红利：小代理模型调参的迁移可行性

三条规律合起来的工程推论是：在与目标模型保持相同 TPP 的小代理模型上扫出的最优重复次数，可以为更大模型提供实用估计。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 其成立条件值得拆开看：固定 TPP 正是让「小模型最优重复率」对大模型保持有效的前提——若代理实验的 TPP 与目标训练不一致，最优重复次数的估计会随规模漂移而失真。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 这与「在小模型上算最优参数/token 配比再外推」的经典 scaling-law 做法同构，但外推对象换成了每域的重复次数：重复率 sweep 只需在代理模型上进行，无需反复启动大模型训练，算力成本从 O(全量训练 × sweep 网格) 降到 O(代理训练 × sweep 网格)。数据工程团队因此可以把「重复次数」纳入小模型消融的固定清单，与学习率、batch size 一起批量定标，随每次数据配比更新同步刷新。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41] 对预训练与后训练分工的团队，这也再次划清边界：预训练阶段的重复针对的是 TPP 稀释下的曝光不足，与 SFT 阶段多 epoch 复读的任务适配机制不同（参见 [[concepts/llm-pretraining-vs-sft|LLM 预训练 vs SFT]]），二者的收益曲线与过拟合风险不可互相借用。

## 实践启示

1. **按损失定重复，不按体量**：为每个域建立「验证损失 → 重复次数」的校准关系，损失更低的域分配更多重复预算；不要以数据集大小作为重复依据。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41]
2. **代理模型先行**：在与目标同 TPP 的小模型上 sweep 重复次数再迁移到大模型，把重复率消融纳入小模型实验标准清单，节省大模型试错算力。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41]
3. **锁定 TPP 口径**：代理实验与目标训练保持相同 TPP 是迁移有效的前提，配比口径不一致时外推会失真，先对齐再外推。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41]
4. **主动补偿稀释**：域占比随规模下降是规模化的内生现象，混合配比设计应显式预留重复额度对冲，而非被动接受稀释。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md:41]
5. **区分两种复读机制**：预训练重复解决 TPP 稀释下的曝光不足，SFT 多 epoch 是任务适配手段，二者收益曲线与风险不同，不要混用同一套重复参数。
6. **给数据护城河重新定价**：唯一数据量与最优重复仅弱相关，稀缺但高质量的小型域数据比庞大的中庸语料更值得购入与清洗——数据壁垒的质量密度重于规模。

## 意义

该研究直接回应 LLM 训练中的数据工程难题——当高质量域数据供给不足时，重复率该如何设定。它与 [[entities/generalization-dynamics-lm-pretraining|LM 预训练泛化动力学]]、[[entities/notes-on-pretraining-parallelisms-and-failed-training-runs|预训练并行与失败记录]] 同属 LLM 预训练实证研究前沿，为数据配比与训练预算设计提供可迁移的 scale 规律。

→ [[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|原文存档]]
