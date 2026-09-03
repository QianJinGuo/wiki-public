---
title: "MineExplorer：开放世界多模态评测基准"
created: 2026-08-18
updated: 2026-08-18
type: entity
tags: [ai, agent, benchmark, multimodal, evaluation, world-model, long-horizon, meituan, video-generation]
sources: [raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026]
confidence: 0.75
provenance_state: extracted
score: 72
---

# MineExplorer：开放世界多模态评测基准

## 核心洞察

多模态大模型能看懂图像、解析视频、在复杂场景里做静态推理，但一旦被放进一个**实时变化、需要持续探索的开放世界**（如《我的世界》），其真实能力出现系统性断层——这是静态 benchmark（截图问答）掩盖的长程能力短板。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md]

## 评测设计

美团 LongCat 团队构建了 **MineExplorer**——首个在开放世界中做到**分钟级长程任务**的评测基准，系统评测多模态大模型在需要**长程规划**、且包含**隐藏前置条件**（如先砍树才能获得木头、先熔炼才能得到工具）的任务中的真实能力。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md]

- 与静态截图问答的区别：模型必须在环境中持续执行、感知反馈、修正规划
- 任务结构：长时间跨度、多步骤因果链、隐含前置条件
- 目标：暴露"温室"（静态数据集）与"开放世界"（动态执行）之间的能力落差

## 与现有评测体系的关系

MineExplorer 属于 agent/能力评测家族的新分支——面向**执行型长程任务**而非单轮问答。它与 [[entities/meituan-longcat-vitabench-20-long-term-dynamic-agent-benchmark|VitaBench 2.0]]（长期动态智能体基准）、Agent 评测基准、[[entities/从月球漫步到赛博都市wbench测出了世界模型的边界|WBench 世界模型边界]] 同属"更贴近真实执行环境"的评测转向。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md]

## 价值

- 提供了超越静态 VQA 的**动态世界评测范式**，可迁移用于 video-understanding / agentic / world-model 类模型的执行能力评估
- 揭示多模态模型在长程规划与隐含前置条件推理上的真实短板

→ [[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026|原文存档]]
