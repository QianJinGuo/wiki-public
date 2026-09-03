---
title: "Discretizing Reward Models"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/discretizing-reward-models]
provenance_state: extracted
---

> -> [[raw/articles/discretizing-reward-models.md|原文存档]]

sha256: e08d60ab3e144ad825821f33eefd79b45cb2dcaa5abb385279f4929f675b3f44 ^[raw/articles/discretizing-reward-models.md]

## 摘要

这是 arXiv 论文《Discretizing Reward Models》（编号 2606.21795，cs.LG，v1 于 2026-06-19 提交，提交人 Vijay Viswanathan）的摘要页。论文指出 reward model 在强化学习中虽被广泛使用，但其作用机理尚不清楚：与产生二值分数的 verifiable rewards 不同，reward model 输出连续分数、理应对响应间的细粒度差异敏感——但论文证明这种表面优势恰是严重弱点：许多流行的 reward model 是"过度敏感"（oversensitive）的，会给同样好的响应打出不同分数；理论上，看似完美的 reward model 也可能高度过度敏感，实证上这种过度敏感会导致糟糕的策略。作为替代现有"reward model accuracy"概念的评估框架，论文提出用"判别能力"（discriminative ability）与"特异度"（specificity，过度敏感的补集）两个独立维度来评估 reward model。解决方案是一个无需训练的算法：对任意神经 reward model 施加 Monte Carlo dropout 产生离散 reward 聚类；理论上证明了存在能在最小牺牲判别能力的代价下降低过度敏感的离散化方案，实证上在受控与自然 RL 两种设定下，对 reward 做离散化都比直接在原始 reward 上训练带来更少的 reward hacking 和更好的策略。^[raw/articles/discretizing-reward-models.md]

## 关键要点

- 核心发现：reward model 的连续分数带来的"细粒度敏感性"是严重弱点——oversensitive 的 reward model 会对 equally good 的响应赋予不同分数
- 理论结果：看似完美的 reward model 也可能高度过度敏感；过度敏感会实证性地导致坏策略
- 提出新的评估维度：以"discriminative ability"（判别能力）与"specificity"（特异度 = 过度敏感的补集）替代传统的"reward model accuracy"
- 解决方案：training-free 算法，用 Monte Carlo dropout 在任意神经 reward model 上产生离散 reward 聚类
- 理论保证：存在这样的离散化方案，能以最小的判别能力损失降低过度敏感
- 实证结论：在受控与自然 RL 设定下，离散化 reward 相比原始 reward 训练，reward hacking 更少、最终策略更好

## 来源

- 原文: [[raw/articles/discretizing-reward-models.md|Discretizing Reward Models]]
