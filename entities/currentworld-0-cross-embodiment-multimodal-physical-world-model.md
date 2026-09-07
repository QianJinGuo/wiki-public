---
title: "CurrentWorld-0 — 跨本体多视角多模态物理世界模型"
created: 2026-08-19
updated: 2026-09-07
type: entity
tags: [world-model, robotics, physical-world-model, multimodal, embodiment, tactile, current-robotics, simulation, robot-learning]
sources: [raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model]
confidence: 0.78
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# CurrentWorld-0 — 跨本体多视角多模态物理世界模型

## 概述

Current Robotics 发布 **CurrentWorld-0**，一种**交互式世界仿真器（Interactive World Simulator）**，是其在发布人形全身精细操作 Curr-0 后的第二份工作。与依赖人工编写物理公式的传统物理引擎不同，它从真实世界数据中学习规律，形成数据驱动的仿真方式。^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]

## 核心创新：跨本体 + 多视角 + 力触觉统一

这是**首次将跨本体、多视角和力触预测整合进同一个世界仿真器**，同时用于模拟、学习以及与人类和机器人模型交互。此前类似方向如 1X World Model、Physical Intelligence 的 Ctrl-World 与 SC3-Eval、DeepMind WorldGym、英伟达 Cosmos Predict，但都未做到三者统一。^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]

- **跨本体**：移动机器人、配备灵巧手的双足人形机器人都可被建模与控制^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]
- **多视角 + 力触觉**：不只刚体，柔性物体与流体（叠袜子、拾枕头、削黄瓜）都能保持合理物理状态^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]
- 关键价值不在画面逼真度，而在于机器人动作变化后世界如何正确演化^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]

## 意义

CurrentWorld-0 与 [[entities/feifei-li-masked-visual-actions-world-model-2026|李飞飞世界模型]]、[[entities/nvidia-gamma-world-multi-agent-world-model|NVIDIA Gamma World]] 等同属世界模型前沿，但其「跨本体 + 力触觉」的数据驱动仿真路线在具身智能领域具有差异化方法论价值。

→ [[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model|原文存档]]
