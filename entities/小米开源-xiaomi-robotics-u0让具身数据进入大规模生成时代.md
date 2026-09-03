---
title: "小米开源 Xiaomi-Robotics-U0：让具身数据进入大规模生成时代"
created: 2026-08-13
updated: 2026-08-13
type: entity
tags: [ai, research, rl, reinforcement-learning, post-training, evaluation, benchmark, agent-eval, world-model, multimodal, vlm, vision, robotics, embodied-ai, inference, llm-inference]
sources: [raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代.md]
confidence: 0.6
provenance_state: extracted
---

# 小米开源 Xiaomi-Robotics-U0：让具身数据进入大规模生成时代

> WeChat-小米技术 | 发布于 2026-07-15 | 评分入库 v×c≥49

## 核心内容

原创 小米机器人事业部 2026-07-15 14:42 北京 今天，小米正式发布 Xiaomi-Robotics-U0 ——一个拥有 380 亿参数的多模态自回归具身生成基础模型，是具身领域首个“通吃”四类任务的统一生成模型， 打通了机器人图片和视频数据的生成与编辑链路。 它既能在保持几何一致性的前提下，对已有数据做增强——换物体、换光照、换背景、加干扰，无需重新采集；也能从零生成全新场景，覆盖危险、极端、长尾等真机难以触达的环境。此外，通过 FlashAR+ 推理加速方案，它的生成效率较原始自回归范式提升近 83 倍，大幅加快工程落地速度。规模化生成具身训练数据用于增益模型效果，从此有了可控且高效的解决方案。 在 WorldArena 评测基准上，Xiaomi-Robotics-U0 取得总分第一名 （全球 126 个模型参评）。此外，真机评测中，在未知光照、陌生背景等 Out of Distribution 场景下，使用 Xiaomi-Robotics-U0 扩增数据训练的策略任务完成进度平均提升超 26% 。 相关代码与模型权重已全量开源：<https://robotics.xiaomi.com/xiaomi-robotics-u0.html 01 一个通用模型，覆盖四类生成任务 过去，具身生成往往是“一个任务一个模型”：场景生成用一个模型，轨迹迁移用一个模型，视频生成又是另一个模型。模型之间相互割裂，导致具身生成很难规模化应用。 Xiaomi-Robotics-U0 选择了另一条路：用统一的多模态自回归框架，覆盖四类核心任务。 具身场景生成（Scene Generat。^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代.md.md]

## 关键要点

- 原文完整记录：[[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代.md|原文存档]]
- 关联主题："Agent 评估基准体系"、[[concepts/evaluation-harness-design]]、[[concepts/embodied-intelligence-frontier]]

## 相关实体

"Agent 评估基准体系" [[concepts/evaluation-harness-design]] [[concepts/embodied-intelligence-frontier]]
