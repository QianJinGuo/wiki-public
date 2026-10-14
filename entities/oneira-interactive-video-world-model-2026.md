---
title: "Oneira：状态表驱动的交互式视频世界模型"
created: 2026-10-14
updated: 2026-10-14
type: entity
tags: [world-model, video-generation, interactive, coding-agent]
sources: [raw/articles/oneira-interactive-video-world-model-2026]
confidence: 0.7
---

# Oneira：状态表驱动的交互式视频世界模型

arxiv 2610.01614。针对视频世界模型的两个顽疾——交互作用在错误物体上、镜头移开再返回后物体变化消失——提出「世界中发生了什么」与「画面如何呈现」分离的架构 ^[raw/articles/oneira-interactive-video-world-model-2026.md]。

## 核心机制：World State Table + Coding Agent

- **Coding Agent** 通过 World State Table 记录场景中每个物体的位置与状态；
- 给定动作/目标后，Agent 先读表确定交互对象并规划交互，再把结果**写回表**——状态持久化在符号层，不依赖生成画面的连续性；
- 交互准确性与长时一致性上明显优于现有方法。

这与游戏引擎的显式状态管理同构，但状态表由 agent 在生成过程中维护，场景可随生成内容扩展。与 [[entities/alaya-evoke-hour-level-video-world-model]]（长时视频世界模型）相比，Oneira 把一致性从生成侧转移到状态侧；与 [[entities/code-world-model-coding-agent-as-world-brain-2026]]（Coding Agent 作为世界大脑）共享「代码/符号层承载世界状态」的思想。

- 代码：github.com/Madaoer/Oneira

## 关联
- → [[raw/articles/oneira-interactive-video-world-model-2026|原文存档]]
