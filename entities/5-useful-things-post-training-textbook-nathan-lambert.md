---
title: "Nathan Lambert RLHF 教科书：后训练五大核心知识点"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: [rlhf, post-training, reinforcement-learning, alignment, llm, nathan-lambert, textbook]
sources: [raw/articles/5-useful-things-youll-learn-in-my-new-post-training-textbook]
confidence: 0.75
---

# Nathan Lambert RLHF 教科书：后训练五大核心知识点

Nathan Lambert（Interconnects 作者、前 HuggingFace 研究员）出版了 Manning 新书 _Reinforcement Learning from Human Feedback: Aligning and Post-training LLMs_。本文介绍了书中五个最值得学习的核心知识点。^[raw/articles/5-useful-things-youll-learn-in-my-new-post-training-textbook.md]

## 核心内容

### 1. Rejection Sampling（拒绝采样）

后训练中的关键采样策略——从模型生成多个候选回答，用奖励模型筛选最佳，再用于 SFT 微调。Lambert 指出这一方法在 2024 年后被广泛采用，但网上缺乏系统性的入门解释。^[raw/articles/5-useful-things-youll-learn-in-my-new-post-training-textbook.md]

### 2. Outcome Reward Models（结果奖励模型）

与传统的过程奖励模型（PRM）不同，ORM 只对最终结果打分，不追踪中间步骤。在推理密集型任务（数学证明、代码生成）中，ORM 的训练成本更低且效果相当。^[raw/articles/5-useful-things-youll-learn-in-my-new-post-training-textbook.md]

### 3. Character Training（角色训练）

教 LLM 表现特定"人格"或"角色"的微调方法。这本书是少数系统讨论这一技术的资源之一，对产品化 AI 助手（如 Claude 的 personality）有直接参考价值。^[raw/articles/5-useful-things-youll-learn-in-my-new-post-training-textbook.md]

### 4. 后训练的核心 trade-off

Lambert 强调后训练的三个关键权衡：对齐 vs 智能、安全 vs 有用性、通用性 vs 专业性。大部分 LLM 厂商面临的选择本质上是这些维度的平衡。^[raw/articles/5-useful-things-youll-learn-in-my-new-post-training-textbook.md]

### 5. 历史脉络与常见误解

书的很大篇幅用于解释后训练为什么有效，以及人们常犯的误解。许多核心技术在过去几年并未改变，但理解其"为什么"比"怎么做"更重要。^[raw/articles/5-useful-things-youll-learn-in-my-new-post-training-textbook.md]

## 相关实体

- [[entities/kimi-k3-2.8t-open-source-model-2026|Kimi K3 开源模型]]
- [[entities/deploying-kimi-k3-on-aws|Kimi K3 部署]]

→ [[raw/articles/5-useful-things-youll-learn-in-my-new-post-training-textbook|原文存档]]
