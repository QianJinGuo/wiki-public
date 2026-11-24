---
title: "4D-WAM — 面向世界动作模型的 3D 轨迹表征对齐（训练时辅助监督，推理零开销）"
created: 2026-08-17
updated: 2026-11-16
type: entity
tags: [world-action-model, 4d, trajectory, robot, sim2real, alignment, vla, world-model, auxiliary-supervision, robotics]
sources: [raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026]
confidence: 0.65
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
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

## 深度分析

### 差分对齐：绕开"表征语言不兼容"

前置实验的负相关结论是整个方法的地基：VLA 的视觉表征与 3D 特征通常正相关，但 DiT 架构 WAM 的中间表征与 Trace Anything 的 4D 表征在多数层上不相关甚至负相关——同一个世界，两套"表征语言"，逐点对齐注定优化冲突。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md] 4D-WAM 的转向是**对齐变化规律而非数值本身**：Motion Alignment 取相邻帧 WAM token 与 4D 轨迹表征的帧间差分，在共享空间用 token 级余弦距离对齐运动方向；Destination Alignment 把首帧视为 source、末帧视为 destination，用带温度系数的 source-conditioned destination 分布之间的 KL 散度迁移目标对应关系。差分与分布对表征坐标系的整体偏移不敏感，等于换了对齐的量而非换了地图，从结构上绕开了"两张坐标系不同的地图强行数值对齐"的死局。

### 教师-学生结构：具身版的表征蒸馏

Trace Anything 在这里扮演冻结教师：提取的 3D 轨迹场提供 4D 表征监督信号，WAM 作为学生只经一个轻量投影层接入。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md] 这正是蒸馏范式的具身化变体——与 [[concepts/model-distillation-compression|模型蒸馏压缩]] 中传递 logit 分布而非硬标签同理，4D-WAM 传递的是**运动趋势与目标对应关系**这类关系性信号，而不是教师的绝对特征；训练时冻结教师、只训练投影层与对齐目标，教师侧无梯度负担。与 [[concepts/world-models|世界模型]] 的关系上，它把"想象未来"（视频预测）与"执行未来"（动作生成）之间的鸿沟归因为 3D 空间理解缺位，并用监督信号而非结构改动来补。

### 零开销的精确边界：什么被移除，什么留下

"推理零开销"有明确构成：训练结束后 4D 教师模型、投影层与两项对齐目标**全部移除**，部署时保留原始 WAM 推理架构，额外推理参数、延迟与显存均为 0。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md] 代价全部落在训练侧——预缓存轨迹表征（离线）下每步训练 7.3s→7.5s，8 卡总时长 51h→52h，可训练参数存储 56MiB，约 2% 训练时间增幅。这一"训练时买理解、推理时免费"的交换，与 [[entities/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026|Lingbot-VA 具身视频动作预训练]] 等重结构路线形成对照：4D-WAM 不重新设计 WAM 架构，只改训练目标，因此才能在 FastWAM-Joint 与 Lingbot-VA 两种主干上直接迁移。

### 真实机器人的"低起点高增益"信号

真实世界数据最值得注意：ARX LIFT2 双臂机器人四类任务（积木插入、动态灯光抓取、扳手定位、毛巾整理）仅 550 条示范 + 7000 步训练，平均进度 20.3%→31.8%，成功率 1.0%→5.5%，其中积木与扳手任务推进最明显。^[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026.md] 绝对值仍然很低，但增益发生在**真实数据稀缺、基线接近失败**的区间——恰恰是 3D 目标感知最有价值的地方。与 [[entities/feifei-li-tract-visual-tracks-robot-world-model|Fei-Fei Li Track 视觉轨迹世界模型]] 一脉相承的是"轨迹即监督"的判断：少记绝对表征、多理解动态规律，蒸馏的是轨迹结构而非像素外观。

## 实践启示

1. **辅助监督尽量做成"训练时专属"**：把新能力封装进训练阶段、推理时整体移除，新增能力的评审标准就多一条——推理预算是否为零。带部署开销的方案应先论证为什么不能后移。
2. **做表征对齐前先测相关性**：先像前置实验那样统计中间表征与目标表征空间的相关性；负相关就放弃逐点对齐，改对齐差分、方向或分布等对坐标系不敏感的统计量。
3. **局部与全局双尺度对齐**：只对齐相邻帧变化会短时跟踪良好但目标漂移，只对齐目标又丢失运动细节；Motion + Destination 的组合（"下一小段路怎么走"+"最终目的地在哪"）是可复用的配方。
4. **优先改训练目标而非架构**：结构无关的策略才具备跨主干可迁移性（FastWAM-Joint、Lingbot-VA 均验证有效）；冻结教师 + 轻量投影层的接入方式把改动面压到最小。
5. **仿真增益用分布外扰动检验**：标准基准接近饱和时，LIBERO-Plus 七类扰动（相机扰动下 27.89%→45.15%，平均 +8.8pp）才暴露空间理解的真实差异——评测扰动维度应覆盖视角、光照、布局。
6. **算清"训练换推理"的账**：约 2% 训练时间增幅 + 56MiB 存储换取跨基准、跨扰动、跨主干与真机的持续提升；这笔账（训练增量 vs 推理节省）应成为轻量训练配方报告的标准格式。

## 相关

- [[entities/feifei-li-masked-visual-actions-world-model-2026|Fei-Fei Li Masked Visual Actions World Model]]
- [[entities/loopwm-looped-world-models|LoopWM Looped World Models]]
- [[entities/video-world-model-hand-motion-capture-2026|Video World Model 手部运动捕捉]]
- [[entities/worldtrace-addressable-memory-video-world-models|WorldTrace 可寻址记忆视频世界模型]]
- Robotics & Embodied AI
- → [[raw/articles/4d-wam-world-action-model-3d-trajectory-alignment-2026|原文存档]]
