---
title: "DataFlex-RL：北大开源强化学习数据策略框架，接入更简单、对比更公平"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: [post-training, rl-data, data-strategy, grpo, rlvr, open-source, verl]
sources: [raw/articles/dataflex-rl-pku-rl-data-strategy-framework]
confidence: 0.7
---

# DataFlex-RL：北大开源强化学习数据策略框架，接入更简单、对比更公平

> **Background**：本文基于新智元对北大 DCAI 团队 DataFlex-RL 发布的报道整理。DataFlex-RL 是北大团队联合 UCAS、上海算法创新研究院、中关村学院发布的开源 RL 数据策略框架（GitHub: haolpku/DataFlex-RL，论文 arXiv 2609.06107），登上 HuggingFace #2 Paper of the day。

> → [[raw/articles/dataflex-rl-pku-rl-data-strategy-framework|原文存档]]

## 核心要点

- **DataFlex-RL** 把 RL 后训练中的数据策略（选择/加权/混合）做成 verl 可配置组件，开发者无需改训练器即可接入 ^[raw/articles/dataflex-rl-pku-rl-data-strategy-framework.md]
- 复用 GRPO/RLVR 循环已有的奖励、优势、token 概率、prompt 分组信号，rollout/验证/策略更新代码基本不变 ^[raw/articles/dataflex-rl-pku-rl-data-strategy-framework.md]
- 三类动态数据策略维度：**选择**（哪些 rollout 参与更新：组内解出率、advantage 排名、奖励方差、回答效率筛选）、**加权**（样本/token 学习权重）、**混合**（按历史反馈调整后续批次领域构成）^一^[raw/articles/dataflex-rl-pku-rl-data-strategy-framework.md]
- 解决的核心痛点：不同数据策略各有一套实现、比较效果时模型/奖励函数/训练预算不一致，分数提升难以归因于数据策略本身 ^[raw/articles/dataflex-rl-pku-rl-data-strategy-framework.md]
- 论文组织 591 次实验，在多种模型与数学/逻辑/科学任务上检验数据策略效果 ^[raw/articles/dataflex-rl-pku-rl-data-strategy-framework.md]

## 为什么重要

随着推理模型与 RLVR 的发展，"模型练什么、怎么练"（数据选择、权重、领域配比）成为后训练的核心杠杆。此前换一个筛选规则就要改一遍训练器，且效果比较缺乏控制变量。DataFlex-RL 把这三类动作标准化为可配置组件，降低了数据策略实验的工程成本，也让"收益来自数据策略还是训练设置"这一归因问题变得可解。

## 相关

- [[entities/good-qc-for-rl-data|Good QC for RL Data]] — 同属 RL 数据工程议题：QC 标准框架（准入审查 + 主动测试）与数据策略接入正交且互补
- 源码：github.com/haolpku/DataFlex-RL · 论文：arXiv 2609.06107
