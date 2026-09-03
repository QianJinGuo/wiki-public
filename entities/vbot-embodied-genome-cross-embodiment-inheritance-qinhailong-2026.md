---
title: "Vbot 具身基因组：跨本体智能继承（维他动力 秦海龙）"
created: 2026-08-22
updated: 2026-08-29
type: entity
tags: [embodied-ai, robot, cross-embodiment, world-model, multi-embodiment, vla, reinforcement-learning]
sources: [raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026]
confidence: 0.7
---

# Vbot 具身基因组：跨本体智能继承（维他动力 秦海龙）

## 概述

维他动力（Vbot）在 WRC 上首次公开人形机器人 Vbot ATOM 并开启预订，同时提出「Vbot Embodied Genome｜具身基因组」技术体系，核心命题是：机器人换了一副「身体」，能否「继承」已经学会的智能。该框架把「智能跨任务继承、跨本体进化」作为通用机器人的关键，区别于行业常见的「一个模型覆盖更多任务」路线，转而追问「一套已形成的智能系统能否顺畅进入截然不同的物理身体」。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

ATOM 并非从零起步，它背后是已量产交付、走进真实用户生活的四足机器人「大头」及其积累的真实数据与迭代闭环。秦海龙（哈工大、新加坡国立、浦项科大背景，前蔚来自动驾驶高级总监、千里智驾首席科学家）于 2026 年 3 月加入维他动力任研发副总裁，负责世界模型、空间智能、Agent OS 及人形机器人研发。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

## 三大核心模型

维他动力公布三大核心模型，对应机器人智能链条的三个关键环节（理解人、预测未来、适应不同本体），连接成「前台持续交互 → 世界持续推理 → 本体持续进化」的完整闭环：^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

- **Vbot-OmniDuplex** — 多模态流式全双工交互与异步推理，负责「理解人」
- **Vbot-WorldModel** — 动作条件未来预测与策略评估，负责「预测未来」
- **Vbot-EvoMorph** — 共享策略骨干 + 多本体 Real-Sim-Real 进化，负责「适应不同本体」

## 跨本体智能继承的分层

针对「智能继承不能一概而论」，秦海龙提出按模块分层的继承能力：^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

- **交互层** — 可 100% 完全继承。机器人与人交互的理解、成功与失败案例与物理本体形态无关，四足积累的交互数据可直接迁移到人形
- **策略生成与空间理解层** — 高度复用。移动导航任务的空间动静态理解、避障、通行判断在四足与人形间差异不大；操作任务采用无本体（Egocentric / 第一人称视角）基模路线，复用大量公开与合成数据，真正的瓶颈在真机后训练数据
- **底层运控与强化学习层** — 「隐空间共通，末端分叉」。上层隐空间理解、环境感知、控制逻辑相通，下层因关节自由度、拓扑结构、运动学不同而输出分叉，需依赖特定本体的适配层与强化学习（RL）完成控制

## Real-Sim-Real 与数据稀缺

体系依托多本体 Real-Sim-Real 进化：策略共享骨干在仿真与真实世界间交替迭代，让一套智能逐步适配多种本体。文中强调具身智能领域真正稀缺的数据是**真机后训练数据**（real-machine post-training data），而非公开/合成基模数据——这决定了跨本体能力能否落地到具体物理身体。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

## 相关实体

- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- 机器人具身 AI
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld 跨本体世界模型]]
- [[entities/embodied-native-llm-embodied-intelligence-new-stage|具身原生 LLM]]
- [[entities/amap-abot-earth-0.5-3d-native-world-model|Amap Abot 3D 世界模型]]
- [[entities/baai-orca-next-state-prediction-world-model|BAAI Orca 世界模型]]

→ [[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026|原文存档]]
