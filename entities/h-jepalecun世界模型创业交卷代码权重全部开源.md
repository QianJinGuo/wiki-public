---
title: "H-JEPA：分层 JEPA 视觉规划世界模型"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [world-model, jepa, hierarchical-planning]
sources: [raw/articles/h-jepalecun世界模型创业交卷代码权重全部开源]
confidence: 0.75
---

# H-JEPA：分层 JEPA 视觉规划世界模型

## 深度分析

LeCun 世界模型公司 AMI 联合 NYU/INRIA/布朗大学发布 H-JEPA：多个 JEPA 逐层堆叠，各层在自身表征空间预测不同时间跨度的状态；规划时高层出粗计划，预测中间状态作为子目标交给低层接力细化。每层三组件（状态编码器/动作编码器/预测器），低层读图像、高层抽象。效果：Visual AntMaze 三层 73% vs 单层 18%，多项仿真任务以更少规划计算量取得更好表现。代码与预训练权重全部开源，兑现 LeCun 2022 年提出的分层 JEPA 构想。 ^[raw/articles/h-jepalecun世界模型创业交卷代码权重全部开源.md]

## 关联

- [[concepts/world-models|世界模型]]
- [[entities/causalwm-causal-chain-of-thought-embodied-world-model-2026|CausalWM 因果世界模型]]

→ [[raw/articles/h-jepalecun世界模型创业交卷代码权重全部开源|原文存档]]
