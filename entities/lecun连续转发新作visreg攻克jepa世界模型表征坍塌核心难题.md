---
title: "VISReg：Variance-Invariance-Sketching Regularization 攻克表征坍塌"
created: 2026-08-06
updated: 2026-08-06
type: entity
tags: [ssl, self-supervised-learning, representation-collapse, jepa, world-model, regularization]
sources: [raw/articles/lecun连续转发新作visreg攻克jepa世界模型表征坍塌核心难题]
confidence: 0.8
provenance_state: extracted
---

# VISReg：Variance-Invariance-Sketching Regularization 攻克表征坍塌

## 背景：SSL 表征坍塌难题

自监督学习（SSL）无需人工标注即可从海量数据中学习通用表征，但普遍面临**表征坍塌（representation collapse）**：模型倾向于把不同输入映射到相同或极少数几个向量上，看似完成了训练，实则未学到有判别力的表征。主流抑制方法依赖启发式技巧（EMA、教师-学生网络、停止梯度、冻结层等），使训练脆弱、难以调参、削弱可解释性与可扩展性。^[raw/articles/lecun连续转发新作visreg攻克jepa世界模型表征坍塌核心难题.md]

## 从 VICReg → SIGReg → VISReg 的演化

- **VICReg**（LeCun 团队）：把学习目标拆为方差、不变性、协方差三项，用协方差约束各维度相关性——但协方差仅刻画二阶统计量，无法区分"均值、方差相同但分布形状迥异"的表征。
- **SIGReg**：基于 Cramér–Wold 定理，用 sketching 技术把整个嵌入分布对齐到标准高斯，约束完整分布形状。但存在两个缺陷：① **坍塌时梯度消失**——表征越坍塌、修正信号越弱，模型难以自行恢复；② **尺度与形状耦合**——未分离"幅度大小"与"分布形态"两个独立属性，在长尾、低质量、低秩数据上适配性差。
- **VISReg**（Variance-Invariance-Sketching Regularization）：正是为解决 SIGReg 的梯度消失与尺度/形状耦合问题而生。LeCun 连续转发并评价"VICReg begat SIGReg which begat VISReg"。^[raw/articles/lecun连续转发新作visreg攻克jepa世界模型表征坍塌核心难题.md]

## 意义：JEPA 世界模型的关键拼图

表征坍塌是 JEPA（联合嵌入预测架构）世界模型的核心难题之一——VISReg 提供了一条不依赖启发式技巧的正则化路线。这与 LeCun 关于世界模型的整体构想直接相关。^[raw/articles/lecun连续转发新作visreg攻克jepa世界模型表征坍塌核心难题.md]

## 与 Wiki 现有知识的关联

- JEPA 世界模型背景：[[entities/yann-lecun-jepa-world-model|Yann LeCun JEPA World Model]]
- 世界模型前沿：[[entities/baai-orca-next-state-prediction-world-model|BAAI ORCA 世界模型]]、[[entities/feifei-li-masked-visual-actions-world-model-2026|李飞飞 Masked Visual Actions]]
- 表征/熵坍塌相关：[[entities/llm-rl-entropy-collapse-acl-2026-outstanding-paper|LLM RL 熵坍塌 ACL 2026]]

→ [[raw/articles/lecun连续转发新作visreg攻克jepa世界模型表征坍塌核心难题|原文存档]]
