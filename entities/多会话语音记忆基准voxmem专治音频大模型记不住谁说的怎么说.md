---
title: "VoxMem：多会话语音记忆基准"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [benchmark, audio, memory, lalm]
sources: [raw/articles/多会话语音记忆基准voxmem专治音频大模型记不住谁说的怎么说]
confidence: 0.75
---

# VoxMem：多会话语音记忆基准

## 深度分析

墨尔本大学+UNSW 团队的多会话语音记忆基准 VoxMem（arXiv:2609.32607）：核心洞察是「说话人是谁/语气如何/背景有什么」三类信息转写即丢失，现有基准只考词汇内容。设计：声学证据×记忆操作二维分类覆盖 15 种考察组合，3196 评测实例、34743 语音会话（177 小时），每题在 8K-64K token 历史长度下保持不变以分离历史长度影响。结果：15 个音频大模型 32K 下无一整体准确率超 40%，语气变化/背景声追踪平均仅 3.4%/1.2%——LALM 副语言与环境声记忆近乎失效。代码开源。 ^[raw/articles/多会话语音记忆基准voxmem专治音频大模型记不住谁说的怎么说.md]

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]
- [[entities/agent-memory-architecture|Agent 记忆架构]]

→ [[raw/articles/多会话语音记忆基准voxmem专治音频大模型记不住谁说的怎么说|原文存档]]
