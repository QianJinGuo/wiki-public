---
title: "Handroid：同一套硬件在人形机器人与灵巧手之间重构（UNC+Stanford）"
created: 2026-08-14
updated: 2026-09-07
type: entity
tags: [robotics, embodied-ai, reconfigurable-robot, dexterous-manipulation, diffusion-policy, reinforcement-learning]
sources: [raw/articles/handroid-reconfigurable-robot-dexterous-hand-humanoid-unc-stanford-2026]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Handroid：同一套硬件在人形机器人与灵巧手之间重构

## 核心创新

Handroid 是北卡罗来纳大学教堂山分校与斯坦福大学提出的可重构桌面级机器人（高 0.33m、重 2.05kg、27 自由度），核心思路是**形态复用**：同一套机电系统在灵巧手形态（20 DOF 五指）与人形形态（25 DOF 头+双臂+双腿）之间切换，而非叠加独立灵巧手。^[raw/articles/handroid-reconfigurable-robot-dexterous-hand-humanoid-unc-stanford-2026.md]

## 关键技术

- **机械设计**：手指模块 ↔ 头部/四肢 的关节映射；滑轨 + 齿轮齿条传动完成形态切换（无需拆卸）
- **电磁法兰**：与 Franka Research 3 机械臂快速连接/分离，提供 ~180N 保持力
- **遥操作**：Apple Vision Pro 手部关键点追踪 → 动作重定向 + 数据采集
- **学习策略**：物体条件扩散策略（10 类物体 100 条示教，平均 72% 真实抓取成功率）；仿真 RL 立方体重定向策略部署到真机；人形形态下 ZMP 步态规划 + 参考轨迹 RL 跟踪

## 意义

证明机器人能力扩展可以不依赖增加新部件，而是通过**同一套硬件的形态重组**承担不同功能——把"移动能力"与"精细操作"装进同一副躯体，并支撑从示教、策略学习到真实部署的完整流程。^[raw/articles/handroid-reconfigurable-robot-dexterous-hand-humanoid-unc-stanford-2026.md]

## 与既有实体的关系

与 [[entities/xiaomi-robotics-1-embodied-base-model-scaling-2026|Xiaomi Robotics-1 具身基座模型]] 互补——后者关注数据规模与 Scaling 效应，本文关注单平台硬件复用与策略部署。同属 [[entities/embodied-intelligence-sim-to-real-active-inference-behavior-tree-intrinsic-motivation-chenzhiyan-2026-06-17|具身智能 Sim-to-Real]] 家族的硬件侧实践。

## 引用来源

→ [[raw/articles/handroid-reconfigurable-robot-dexterous-hand-humanoid-unc-stanford-2026|原文存档]]
