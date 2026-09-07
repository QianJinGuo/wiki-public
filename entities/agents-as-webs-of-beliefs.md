---
title: "Agents as Webs of Beliefs"
created: 2026-08-01
updated: 2026-09-07
type: entity
tags: ['raw', 'article']
sources: [raw/articles/agents-as-webs-of-beliefs]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/agents-as-webs-of-beliefs.md|原文存档]]

sha256: e8a6797c7c87ca37f0611da0d636b009704aea4094225d3a2dfd4370cb4aeef7 ^[raw/articles/agents-as-webs-of-beliefs.md]

## 摘要

这是一篇 2026-06-27 发布于 LessWrong 的智能体理论基础文章，提出"信念网（belief webs）"的非形式化框架，综合 active inference、agent foundations 与机器学习三方面思想，试图把信念、目标与行动统一为同一现象的三个侧面。核心前提是：智能体的信念只在局部与邻近信念一致，不必然全局一致——这挑战了用单一概率分布描述智能体的框架（因果图、Solomonoff 归纳、active inference），因此作者借用能处理全局不一致的 Probabilistic Dependency Graphs 与 Garrabrant 归纳（超边/traders 被视为"概念"形式化的雏形），并主张需要支持层级概念形成的扩展。文章进一步提出行动的自预测模型：行动是"因外部执行器介入而自我实现的信念"的子集，由此解释自我/身份作为承诺机制、拖延与自我破坏等内部冲突；目标方面，作者批评 active inference"把目标固定为高置信度信念"的做法会在信念网中传播虚假，转而提出驱力（drives，向上拉目标的力）与锚（anchors，经验证据的固定力）平衡的均衡模型，并引用 Davidad 的"幂零偏好"（任意小幅度驱力的极限情形）。文末列出四个开放问题（最优均衡可达性、EDT/FDT 涌现性质、自引用的精细化、高层概念命题的真值定义）。^[raw/articles/agents-as-webs-of-beliefs.md]

## 关键要点

- 信念网核心前提：信念局部一致而非全局一致，单一概率分布框架（因果图、Solomonoff 归纳、active inference）无法处理这种全局不一致
- 两个可处理不一致的框架：Richardson 的 Probabilistic Dependency Graphs（经验不一致）与 Garrabrant 归纳（逻辑不一致）；超边/traders 对应"概念"的形式化雏形，但两层结构过于人工——深度学习的成功暗示需要层级概念形成
- 行动的自预测模型：行动是（预期）因外部执行器而自我实现的信念子集，与 active inference"行动即预测"相关，但 Garrabrant 归纳能处理自引用悖论，可稳定相信"如果我相信 X 则 X 会成真"
- 自预测模型解释心理现象： believing an action is good 与 actually taking it 之间的鸿沟需要"我是行动者"的身份信念；身份是承诺机制（如戒酒者身份），身份形成不能用贝叶斯更新描述，因为它是在多个自我实现信念间做选择
- 目标即驱力而非固定信念：目标不应固定为人工高置信度，而是被驱力向上拉、被经验证据锚定的信念；完全理性智能体定义为驱力幅度趋于零的极限（Davidad 的 nilpotent preferences）
- 长期愿景：智能体作为涌现现象——所有智能体同属一个巨大未均衡信念网，"agent"是内部更新信任度远高于外部的稠密连通区域

## 来源

- 原文: [[raw/articles/agents-as-webs-of-beliefs.md|Agents as Webs of Beliefs]]
