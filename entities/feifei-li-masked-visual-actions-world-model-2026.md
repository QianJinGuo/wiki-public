---
title: "掩码视觉动作（Masked Visual Actions）——李飞飞团队世界模型"
created: 2026-07-23
updated: 2026-09-07
type: entity
tags: [world-model, robotics, video-model, stanford, fei-fei-li, vision, embodiment, mask]
sources: [raw/articles/feifei-li-masked-visual-actions-world-model-2026]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 掩码视觉动作（Masked Visual Actions）——李飞飞团队世界模型

> **Background**：本文基于 PaperWeekly 对斯坦福李飞飞团队 Masked Visual Actions 论文的解读整理。论文由斯坦福大学联合马里兰大学、哈佛大学研究者共同完成，提出将机器人动作表示为视频中的像素级运动条件，用预训练视频模型统一世界预测与行为生成。^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

## 核心思路

传统机器人世界模型通常接收关节角、末端位姿或控制向量，这些信号与具体机械结构绑定。Masked Visual Actions（掩码视觉动作）将机器人动作表示为**视频中随时间逐帧变化的时空像素掩码**——保留实体在整段视频中的时空像素区域，其余区域统一置为灰色背景。外形、位置、尺度和接触过程都直接可见。^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

## 技术方案

- **底座模型**：Wan-Fun-Control 2.2 14B，采用秩为 256 的 LoRA 适配
- **训练数据**：约 1000 条 DROID 演示 + 4000 个 RoboCasa 样本，总时长约 **15 小时**机器人视频，同时纳入成功与失败轨迹
- **训练规模**：约 10000 步，使用 8 张 H200，耗时约 4 天^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

### 双重用法

在同一掩码表示下，模型始终在补全未显示的实体轨迹：
1. **前向预测**：机器人轨迹作为输入 → 预测物体和环境的变化
2. **逆向生成**：目标物体轨迹作为输入 → 补全相应的机器人行为

前向预测和逆向生成由**同一个模型**完成，无需分设两个网络。^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

### 跨具身泛化

在训练中未出现的夹爪和双臂机器人上，这套视觉接口的跨具身泛化表现出色。视觉表示天然不绑定特定机械结构，可迁移到不同形态的机器人平台。^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

## 与世界模型的关系

该方法可归类为[[concepts/ai-coding-agent-from-helloworld-to-production|基于视频的世界模型]]的新范式——不是用关节角或控制向量表示状态，而是直接用像素级时空掩码统一世界预测和行为生成。与 [[entities/baai-orca-next-state-prediction-world-model|BAAI ORCA 状态预测世界模型]]、[[entities/light-interaction-world-model-inference|轻交互世界模型推理]]、[[entities/loopwm-looped-world-models|循环世界模型]]等方案相比，其核心差异在于**动作表示由数值向量变为视觉掩码**，大幅降低了对具体机械结构的依赖。

## 论文资源

- **论文**：https://arxiv.org/abs/2607.19343
- **GitHub**：https://github.com/HadiZayer/masked-visual-actions
- **项目主页**：https://masked-visual-actions.github.io/

→ [[raw/articles/feifei-li-masked-visual-actions-world-model-2026|原文存档]]
