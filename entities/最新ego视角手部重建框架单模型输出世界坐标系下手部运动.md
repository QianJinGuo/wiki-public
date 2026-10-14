---
title: "最新ego视角手部重建框架，单模型输出世界坐标系下手部运动"
created: 2026-10-15
updated: 2026-10-15
type: entity
tags: [embodied, vision, hand-reconstruction, ego-centric, shanghai-ai-lab]
sources: [raw/articles/最新ego视角手部重建框架单模型输出世界坐标系下手部运动]
confidence: 0.7
---

# 最新ego视角手部重建框架，单模型输出世界坐标系下手部运动

## 核心贡献

InfiniHand（上海人工智能实验室联合上海交大等）是一个流式模型，联合估计**手部位置、手部形态与相机轨迹**，把连续 ego 视频中的动作恢复到共享世界坐标系。传统方案拆成检测器 + 手部姿态网络 + SLAM 三套系统再对齐，检测抖动、轨迹漂移与尺度偏差逐级传递；InfiniHand 让全局几何上下文与局部外观共享，预训练 LingBot-Map 骨干从完整画面提特征，输出 MANO 参数化手模型并约束 2D 投影一致 ^[raw/articles/最新ego视角手部重建框架单模型输出世界坐标系下手部运动.md]。

## 数据与训练

汇集 ARCTIC、HOT3D、EgoDex 等 9 个公开数据集约 **5000 小时**第一视角视频，统一图像/手部参数/关节/掩码/边界框；用已有 3D 关键点补齐缺失 MANO 参数，有相机轨迹的序列保留世界坐标系监督。两阶段训练先学精细手势、再连接跨帧空间关系；后台稀疏束调整修正长序列残余漂移，二值关键帧池兼顾近期与较早观测 ^[raw/articles/最新ego视角手部重建框架单模型输出世界坐标系下手部运动.md]。

## 推理性能

双路并行（一路提取下一窗口骨干特征、一路解码上一窗口结果）减少串行等待，预测吞吐 **11.19FPS ≈ HaWoR 的 2.04 倍**，重建精度在多测试集（含 Ego4D 自然场景 vs ViDiHand/HaWoR/WiLoR）提升；11.19FPS 针对预测吞吐，束调整属额外离线开销。论文 arXiv:2609.35743（2026-09-28）^[raw/articles/最新ego视角手部重建框架单模型输出世界坐标系下手部运动.md]。

## Wiki 视角

对具身智能 pipeline 而言，InfiniHand 解决的是 ego 示范视频 → 3D 动作监督的**转换层**：头戴相机录像便宜、3D 手部动作昂贵，单模型世界坐标系输出让海量人类操作视频可直接变成训练信号。与 Real2Gym（视频 → 可交互仿真）同属"把人类视频变成机器人可用的结构化经验"家族 ^[raw/articles/最新ego视角手部重建框架单模型输出世界坐标系下手部运动.md]。

## 关联

- [[entities/egosuite-open100k-embodied-human-video-data-open-source-2026|EgoSuite/Open100K]] — ego 视频数据侧的互补层
- [[entities/self-harness-shanghai-ai-lab-agent-improves-harness|Self-Harness（上海AI Lab）]] — 同机构的 Agent 自改进路线
- [[entities/roboflywheel-embodied-simulation-assets-2026|RoboFlywheel 百万级仿真资产]] — 仿真资产侧的同一管线邻居

→ [[raw/articles/最新ego视角手部重建框架单模型输出世界坐标系下手部运动|原文存档]]
