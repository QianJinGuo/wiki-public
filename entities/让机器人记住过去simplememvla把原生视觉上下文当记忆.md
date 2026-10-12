---
title: "SimpleMemVLA：原生视觉上下文当记忆"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [vla, memory, robotics, long-horizon]
sources: [raw/articles/让机器人记住过去simplememvla把原生视觉上下文当记忆]
confidence: 0.75
---

# SimpleMemVLA：原生视觉上下文当记忆

## 深度分析

面壁智能联合华科/人大/清华/北大/智源（arXiv:2609.05533）：不加专用记忆模块，把带时间戳的视觉历史直接作为上下文。可行性账：60 秒历史约 5.6k token，主干 262k 上下文装得下分钟级历史。效果：四项记忆基准 SOTA + 两项通用操控保持高性能；RoboMemArena（26 任务、平均 1000+ 控制步）126 秒窗口全阶段成功率 63.6%（+17.4pp），计数 31.4%→71.4%、遮挡 39.1%→64.3%。概念贡献：「写入时承诺」——提前筛选/压缩/覆盖历史会丢掉未来需要的信息，原生上下文把保留决策推迟到读取时。实现沿用 VLM 预训练视频格式+明文时间戳。是 MiniCPM-RobotManip 的支撑技术。 ^[raw/articles/让机器人记住过去simplememvla把原生视觉上下文当记忆.md]

## 关联

- [[entities/agent-memory-architecture|Agent 记忆架构]]
- [[entities/motus-2-self-evolving-world-model-dexterous-manipulation-2026|Motus-2 自进化世界模型]]
- [[concepts/embodied-intelligence-frontier|具身智能前沿]]

→ [[raw/articles/让机器人记住过去simplememvla把原生视觉上下文当记忆|原文存档]]
