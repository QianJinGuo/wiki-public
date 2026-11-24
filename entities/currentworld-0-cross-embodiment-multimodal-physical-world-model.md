---
title: "CurrentWorld-0 — 跨本体多视角多模态物理世界模型"
created: 2026-08-19
updated: 2026-11-21
type: entity
tags: [world-model, robotics, physical-world-model, multimodal, embodiment, tactile, current-robotics, simulation, robot-learning]
sources: [raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model]
confidence: 0.78
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# CurrentWorld-0 — 跨本体多视角多模态物理世界模型

## 概述

Current Robotics 发布 **CurrentWorld-0**，一种**交互式世界仿真器（Interactive World Simulator）**，是其在发布人形全身精细操作 Curr-0 后的第二份工作。与依赖人工编写物理公式的传统物理引擎不同，它从真实世界数据中学习规律，形成数据驱动的仿真方式。^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]

## 核心创新：跨本体 + 多视角 + 力触觉统一

这是**首次将跨本体、多视角和力触预测整合进同一个世界仿真器**，同时用于模拟、学习以及与人类和机器人模型交互。此前类似方向如 1X World Model、Physical Intelligence 的 Ctrl-World 与 SC3-Eval、DeepMind WorldGym、英伟达 Cosmos Predict，但都未做到三者统一。^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]

- **跨本体**：移动机器人、配备灵巧手的双足人形机器人都可被建模与控制^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]
- **多视角 + 力触觉**：不只刚体，柔性物体与流体（叠袜子、拾枕头、削黄瓜）都能保持合理物理状态^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]
- 关键价值不在画面逼真度，而在于机器人动作变化后世界如何正确演化^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md]

## 意义

CurrentWorld-0 与 [[entities/feifei-li-masked-visual-actions-world-model-2026|李飞飞世界模型]]、[[entities/nvidia-gamma-world-multi-agent-world-model|NVIDIA Gamma World]] 等同属世界模型前沿，但其「跨本体 + 力触觉」的数据驱动仿真路线在具身智能领域具有差异化方法论价值。

## 深度分析

### 为什么评测是第一场景：机器人需要可以无限复制的考场

文章把世界仿真器的首要用途落在评测上：训练大模型可以并行跑成千上万个 benchmark case，而机器人的测试规模被真机数量、场地、人工看护和任务复位硬性限制，真实世界不可替代却无法无限复制^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:40-42]。传统物理仿真（如 Isaac Sim）虽能并行大量实验，但前提是世界可以被人工建模：刚体、摩擦、关节可以定义，叠袜子、抹面包、倒啤酒背后的柔性物体、流体与复杂接触却很难逐条写进物理引擎^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:44-46]。CurrentWorld-0 的答案是把真实世界本身"学"进一个可无限运行的环境：直接从真实交互中学习动作之后世界如何演化，若预测足够可靠，大量策略可以先在生成的世界里运行、失败和比较，再把最关键的问题带回真机^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:48-50]。这与 wiki 已有的评测方法论积累（[[entities/world-model-evaluation-position-paper-nju-2026|世界模型评测立场论文]]、[[entities/triworldbench-three-view-embodied-world-model-benchmark-2026|TriWorldBench 三视角具身基准]]）形成呼应。

### 前提是动作必须算数：防止世界模型替机器人纠错

视频模型自带强视觉与运动先验，这在评测场景里可能是负资产：机械臂已经抓偏，模型却可能沿更熟悉的轨迹继续生成，让物体"自动"跟着夹爪移动；任务实际失败，画面却补出一个成功结局^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:66]。一旦世界模型开始替机器人纠错，评测就失去了意义^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:68]。Current Robotics 前作 dWorldEval（ICML 2026 Spotlight）专门研究动作可控性：机器人执行了什么动作，环境就必须给出对应结果；动作没做对，世界必须允许失败真实发生^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:70]。CurrentWorld 在此基础上再解决三个问题，三者共同指向同一句话：无论谁在观察、谁在行动，世界都应该是同一个世界^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:54-58]。

### 三项能力的技术拆解：Action Subspace、联合多视角与力触预测

跨本体：固定双臂、移动双臂与人形机器人的自由度、运动学结构和控制方式各不相同，CurrentWorld 不强行让它们共享动作空间，而是为不同机器人保留各自的 Action Subspace，让不同本体以各自方式行动、却共享对同一个世界的理解^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:74-80]。这与 [[entities/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026|VBot 跨本体基因继承]] 从数据继承角度处理跨本体的路线互补。多视角：头部、腕部和第三视角相机看到的是同一场交互的不同切面，一个视角中物体移动，其他视角的状态必须同步变化，短暂被遮挡后再次出现也不能"换了一个世界"；CurrentWorld 对多个同步视角联合建模，共同锚定同一套物理状态^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:82-88]。力触觉：仅靠视觉无法判断接触是否真正成功——同样是夹住物体，画面几乎一样，但一次已稳定夹持、另一次正在滑落；削皮、开瓶、插拔的结果也由接触和受力决定。因此模型不只预测未来画面，还同时预测力觉与触觉：视觉告诉机器人发生了什么，力触则进一步告诉它接触是否真的发生、正在如何变化^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:90-96]。

### 失败即数据：Hi-WM 接管、回滚与全链路闭环

评测只报告失败还不够，更重要的是能否从失败的位置继续出发、产生新的训练数据^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:104-106]。在 CurrentWorld 中，策略先自主执行，进入潜在失败状态时人类可直接遥操作接管纠正，同一中间状态还可以被保存、回滚、再尝试不同的恢复方式；这延续了 Hi-WM（Human-in-the-World-Model，RSS 2026 Robot World Models Workshop）把一次性失败变成可反复探索的数据节点的思路，且人类介入产生的不只是动作轨迹，还包括同步的视觉、力觉和触觉反馈^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:108-112]。放进 Current Robotics 的技术路线看：HumanEx 提供真实人类行为数据，Curr-0 从中学习全身移动与灵巧操作，策略进入 CurrentWorld 评测、暴露失败，再通过 Hi-WM 在失败附近引入人类接管产生新纠错数据，形成"真实经验→模型学习→世界模型评测→暴露失败→人类纠正→新数据→再训练"的完整循环^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:122-128]。其定位是把两次真实执行之间的空间拉长：让策略更快被验证，让失败可以被重新访问，让新经验从失败中继续生长^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:130-134]。

## 实践启示

1. 评测机器人策略时，先在学到的世界仿真器里运行、失败和比较，只把最关键的问题带回真机，用两次真实执行之间的空间换取迭代速度^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:48-50]。
2. 把"动作必须算数"当作世界模型评测的合格线：警惕视觉先验替机器人补出成功画面，动作可控性（如 dWorldEval 所做）是评测可信的前置条件^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:64-70]。
3. 跨本体建模不必强行统一动作空间：为每个本体保留各自的 Action Subspace，统一的是物理规律而非控制接口^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:74-80]。
4. 多视角数据要联合建模并锚定同一套物理状态：视角可以变化，世界状态不能随视角改变，被遮挡后复现也要一致^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:82-88]。
5. 接触类任务（夹持、削皮、开瓶、插拔）的成败判据在力与触觉，纯视觉世界模型在这类任务上天然不完整^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:90-96]。
6. 把失败设计成数据节点：人类遥操作接管加中间状态保存回滚，让评测从训练的终点变成下一轮训练数据的入口^[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model.md:104-116]。

→ [[raw/articles/currentworld-0-cross-embodiment-multimodal-physical-world-model|原文存档]]
