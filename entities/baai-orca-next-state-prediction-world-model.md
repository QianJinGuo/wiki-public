---
title: "BAAI Orca — 智源悟界 RoboBrain Next-State Prediction 世界模型"
created: 2026-07-08
updated: 2026-09-07
type: entity
tags: [world-model, baai, orca, next-state-prediction, robobrain, state-representation, foundation-model, flagscale]
provenance_state: extracted
confidence: 0.8
sources:
  - raw/articles/baai-orca-next-state-prediction-world-model
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# BAAI Orca — 智源悟界 RoboBrain Next-State Prediction 世界模型

智源研究院（BAAI）悟界·RoboBrain Orca Team 发布的技术报告 Orca: The World is in Your Mind，提出 **Next-State Prediction（下一状态预测）**范式。^[raw/articles/baai-orca-next-state-prediction-world-model.md]

## 核心哲学

> 先让模型学习统一的世界状态表征，再从这个表征中读出理解、预测和行动能力。

Orca 不追求更好的 token 预测、帧生成或动作模仿，而是关注：**当前世界处于什么状态，以及这个状态在自然演化、事件条件或外部干预下，会如何转移到另一个状态。**^[raw/articles/baai-orca-next-state-prediction-world-model.md]


比喻：不让 3 岁小孩进工厂打 10 万小时螺丝——先理解世界，再去做具体任务。^[raw/articles/baai-orca-next-state-prediction-world-model.md]


## 双学习机制

| 学习方式 | 信号 | 来源 | 特点 |
|---------|------|------|------|
| **无意识学习** | 连续视频 | 自然动态观察 | 稠密状态变化，无语言依赖 |
| **有意识学习** | 语言+事件 | 语义条件约束 | 稀疏但有意义的状态转移 |

两类学习共同构造 **world latent**——统一的世界状态表征空间。^[raw/articles/baai-orca-next-state-prediction-world-model.md]

## 数据规模

- 12.5 万小时视频
- 1.6 亿条事件标注
- 1150 万条 VQA
- 来源：第一/第三视角交互、机器人视频、自然动态场景、事件级转移

## 基础设施：FlagScale 框架

基于自研 FlagScale 实现 **4.4× 加速**（H100 集群：0.66 → 2.91 Samples/Sec/GPU）。关键优化：FSDP2 灵活分片、分块交叉熵损失、前向/后向预取通信重叠。^[raw/articles/baai-orca-next-state-prediction-world-model.md]

## Three-in-One Readout

冻结 backbone，仅训练轻量 readout 模块，验证 world latent 是否真正承载多维能力：^[raw/articles/baai-orca-next-state-prediction-world-model.md]


| Readout | 能力 | 关键发现 |
|---------|------|---------|
| **文本读出** | 理解与推理 | 4B规模综合评测更高，尤其状态转移/事件演化维度 |
| **图像读出** | 下一视觉状态预测 | 保持机器人形态/物体布局/物理约束 |
| **动作读出** | 真实机器人控制 | **预训练无 action label**，200条域内轨迹后训练即有效 |

## 关键发现

- **Scaling 有效**：预训练规模增加，三类 readout 同步提升
- **消融实验**：三类训练目标（无意识状态转移、有意识事件转移、VQA 监督）各自承担不同作用，缺一不可
- **对比基于同一套主干 ckpt**，未使用刷榜数据

→ [[raw/articles/baai-orca-next-state-prediction-world-model|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

