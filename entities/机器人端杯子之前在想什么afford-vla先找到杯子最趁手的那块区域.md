---
title: "机器人端杯子之前在想什么？Afford-VLA：先找到杯子最趁手的那块区域"
created: 2026-07-09
updated: 2026-07-09
type: entity
tags: [ai, embodied-ai, vla, affordance, robot, manipulation, computer-vision, visual-planning]
sources: [raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域]
---

# 机器人端杯子之前在想什么？Afford-VLA：先找到杯子最趁手的那块区域

> **Afford-VLA** 是由复旦大学、阿卜杜拉国王科技大学、上海交通大学和华东师范大学联合提出的视觉规划框架。它将可供性（affordance）内化进 VLA 系统内部，让模型自己生成任务相关的交互区域，并将这些局部视觉线索直接送入动作生成模块，使机器人从「看完整张图再猜动作」走向「先找出当前任务该交互的位置，再生成动作」。^[raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域.md]

当前许多 VLA 模型语义理解虽强，但空间交互理解不够精细。机器人「知道物体是什么，不等于知道该在哪里交互」。Afford-VLA 将这个问题放在视觉规划框架中重新审视，提出适合 VLA 的视觉规划应具备四个特性：Local（聚焦任务相关局部区域）、Visually Grounded（直接围绕图像视觉证据）、Internally Generated（由 VLA 内部生成而非级联外部模型）、Action-aligned（直接服务于下游动作决策）。^[raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域.md]

## 核心设计

Afford-VLA 包含三个关键步骤：

**第一步**，引入可学习的 query 作为「交互区域探针」，与视觉和语言信息一起进入 VLM backbone 融合，随后由 Affordance Head 解码出 patch 级 affordance logits。^[raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域.md]

**第二步**，通过 mask pooling 根据预测的 affordance logits 选出最相关的局部视觉 patch，聚合为紧凑的 affordance embedding，拼接进动作生成头——让动作头同时获得全局语义表示和局部交互提示。^[raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域.md]

**第三步**，使用 Straight-Through 风格的 Top-K mask pooling，前向传播保持稀疏 Top-K 选择，反向传播使用可微的软替代梯度，让动作预测损失能够反向更新 affordance logits——模型学到的 affordance 不只是「哪里像交互区域」，而是「哪些区域真的能帮助机器人把动作做好」。^[raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域.md]

在 LIBERO、LIBERO-Plus、SimplerEnv 等多个基准上，Afford-VLA 取得了 SOTA 表现，在空间关系、目标定位和分布偏移场景中展现出优秀的鲁棒性。^[raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域.md]

→ [[raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域|原文存档]] ^[raw/articles/机器人端杯子之前在想什么afford-vla先找到杯子最趁手的那块区域.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

