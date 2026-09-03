---
title: "4D-WAM — 面向世界动作模型的 3D 轨迹表征对齐（训练时辅助监督，推理零开销）"
created: 2026-08-17
updated: 2026-08-29
type: entity
tags: [world-action-model, 4d, trajectory, robot, sim2real, alignment, vla, world-model, auxiliary-supervision, robotics]
sources: [raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026]
confidence: 0.65
provenance_state: extracted
---

# 4D-WAM — 面向世界动作模型的 3D 轨迹表征对齐

## 概述

**4D-WAM**（4D World Action Model）是 4D-WAM 团队提出的面向**世界动作模型（World Action Models, WAMs）** 的 3D 轨迹表征对齐训练策略。它针对真实机器人「**看到的是 2D 投影，执行的却是 3D 交互**」的核心鸿沟，通过**只在训练阶段**引入 3D 轨迹场监督（Motion Alignment + Destination Alignment 双辅助目标），让 WAM 学会理解动态三维世界，而**不增加任何推理参数量、延迟或显存开销**。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]

## 核心问题与反直觉发现

- **问题**：主流的 WAM 方法在二维像素空间建模视觉预测与动作生成，但真实机器人执行的是三维空间、随时间演化的动作。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]
- **反直觉前置实验**：直接对齐 WAM 中间表征与 4D 基础模型特征，**不仅没有稳定收益，反而造成优化冲突**。典型 VLA 的视觉表征与 3D 特征正相关，但基于 DiT 的 WAM 中间表征与 4D 表征在多数层上不相关甚至负相关——两者用不同的「表征语言」描述同一世界，强行逐点一致如同让不同坐标系的地图数值对齐。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]

## 方法：一条轨迹，两种互补监督

4D-WAM 引入 **Trace Anything** 作为教师模型（提取 3D 轨迹场），在训练阶段加入两项互补辅助目标：^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]

- **Motion Alignment（运动对齐）**：对齐相邻帧 WAM token 与 4D 轨迹表征的帧间差分（token 级余弦距离），让模型学习与真实轨迹一致的运动趋势，而非复制绝对特征。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]
- **Destination Alignment（终点对齐）**：把首帧视为 source、末帧视为 destination，学习早期运动区域与最终目标区域的对应关系（source-conditioned destination 分布 + KL 散度），为局部动作加入全局目标感知，防止「走偏」。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]

**关键设计**：辅助监督只在训练阶段使用；训练结束后 4D 教师模型、投影层与两项对齐目标**完全移除**，部署时保留原始 WAM 推理架构。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]

## 效果

- **跨主干可迁移**：在 FastWAM-Joint 与 Lingbot-VA 两种主干的 RoboTwin 2.0 / LIBERO / LIBERO-Plus / RoboTwin Clean2Rand 上系统验证；迁移到 Lingbot-VA 后随机化场景成功率 34.6%→41.8%。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]
- **分布外扰动显著**：LIBERO-Plus 七类扰动相机扰动下成功率 27.89%→45.15%（+17.26pp），平均提升 +8.8pp。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]
- **真实机器人**：ARX LIFT2 双臂机器人四类任务（积木插入/动态灯光抓取/扳手定位/毛巾整理），550 条示范 + 7000 步训练，四任务平均进度 20.3%→31.8%，成功率 1.0%→5.5%。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]
- **推理零开销**：4D-WAM 仅以约 2% 训练时间增幅（51h→52h）+56MiB 存储，换取跨基准/跨扰动/跨主干/真实机器人的持续提升，推理阶段零额外开销。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]

## 方法论价值

4D-WAM 提供一套**可迁移、轻量、训练时专属**的 3D 感知配方：与其对齐绝对特征，不如对齐**动态规律**（运动趋势 + 最终目标）；额外的时空理解能力全部留在训练阶段，不把负担带到部署中。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md]

## 相关

- [[entities/feifei-li-masked-visual-actions-world-model-2026|Fei-Fei Li Masked Visual Actions World Model]]
- [[entities/loopwm-looped-world-models|LoopWM Looped World Models]]
- [[entities/video-world-model-hand-motion-capture-2026|Video World Model 手部运动捕捉]]
- [[entities/worldtrace-addressable-memory-video-world-models|WorldTrace 可寻址记忆视频世界模型]]
- Robotics & Embodied AI
- → [[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026|原文存档]]
