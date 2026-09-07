---
title: "LeRobot v0.6.0 — Imagine, Evaluate, Improve"
created: 2026-08-14
updated: 2026-09-07
type: entity
tags: [lerobot, robotics, embodied-ai, world-model, vla, reward-model, huggingface, open-source]
sources: [raw/articles/lerobot-v060-imagine-evaluate-improve]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LeRobot v0.6.0 — Imagine, Evaluate, Improve

Hugging Face LeRobot 框架 2026 年发布 v0.6.0，主题是**闭合机器人学习回路**：策略先想象未来再行动（world model policies）、奖励模型判断机器人是否成功、部署 CLI 把失败转化为训练数据、六个新仿真基准统一度量。^[raw/articles/lerobot-v060-imagine-evaluate-improve.md]

## 核心新增

- **World model policies**：VLA-JEPA、FastWAM、LingBot-VA 三类学会"想象未来"的策略
- **新 VLA 家族**：GR00T N1.7、MolmoAct2、EO-1、EVO1、Multitask DiT
- **Reward models API**：Robometer、TOPReward，判断任务成功与否
- **`lerobot-eval`**：六个新仿真基准统一度量入口
- **`lerobot-rollout` CLI**：DAgger 式人在回路纠错，失败样本直接沉淀为训练数据
- **训练基建**：FSDP 训练、HF Jobs 云训练
- **数据管线**：深度图支持、VLM 自动语言标注、自定义视频编码、数据加载提速 2x

## 与 wiki 内相关主题的关系

与 机器人具身智能 领域直接相关：world model policies 是 [[entities/amap-abot-earth-0.5-3d-native-world-model|3D 原生世界模型]] 一族的策略侧延伸；reward models 方向与 [[entities/embodied-ai-data-market-landscape-97-players-44-billion-2026|具身数据市场]] 的评估环节互补。LeRobot 定位为开放机器人学习栈，与商业 VLA 方案形成开源对照。

→ [[raw/articles/lerobot-v060-imagine-evaluate-improve|原文存档]]
