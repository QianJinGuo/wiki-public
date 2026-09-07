---
title: "10万小时训出开箱即用机器人基座模型：Xiaomi-Robotics-1 探索具身智能 Scaling 效应"
created: 2026-08-15
updated: 2026-08-15
type: entity
tags: [ai, robotics, embodied-ai, foundation-model, scaling-law, xiaomi, vla]
sources: [raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Xiaomi-Robotics-1：10 万小时数据训练的具身智能基座模型

Xiaomi-Robotics-1 是小米机器人事业部发布的面向真实移动操作任务的具身基座模型，基于 **10 万小时真实世界轨迹** 预训练，再结合跨本体数据后训练，在未见环境真实机器人任务、复杂新任务适配和多个仿真基准上都展现出稳定的规模化收益。它试图回答一个关键问题：机器人策略模型是否也能像大模型一样，随着数据规模、模型容量和训练计算量的提升而持续 Scaling。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

在 [[concepts/embodied-intelligence-frontier|具身智能前沿]] 的探索中，真实机器人数据采集依赖具体硬件、真实环境和人工遥操作，成本高、周期长、规模有限，传统数据又多集中在少量场景和任务上，难以支撑泛化能力。Xiaomi-Robotics-1 正是围绕"机器人策略模型能否像大模型一样 Scaling"这一核心问题展开的系统性尝试。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

## 数据工程：10 万小时 UMI 轨迹与自动化标注

预训练阶段使用了 10 万小时真实世界操作轨迹，通过 Universal Manipulation Interface（UMI）设备采集，覆盖家庭、商业空间、工业场景、办公室、户外等多类环境。UMI 数据不依赖特定机器人本体，可先让模型从大规模真实操作轨迹中学习通用动作生成表征，再迁移到真实机器人上。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

面对 10 万小时数据，团队构建了可规模化自动标注流程：将长轨迹切分为固定长度片段，用视觉语言模型描述片段中夹爪状态和交互物体状态的变化，模型由此学习"在当前视觉观察和语言条件下生成推动场景状态变化的动作"。该流程约 2 周即可完成全量数据的高质量标注。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

## 两阶段训练：从通用动作生成到真实机器人执行

Xiaomi-Robotics-1 采用"预训练 + 后训练"两阶段范式。预训练阶段学习通用动作生成能力：给定当前视觉观察和语言描述，模型预测一段动作序列使场景从当前状态向目标状态变化。后训练阶段解决两个对齐问题：**本体对齐**（将 UMI 数据获得的动作生成能力迁移到真实机器人本体）与 **指令对齐**（将"根据状态变化描述生成动作"转化为"根据自然语言指令执行任务"）。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

后训练数据约 10000 小时跨本体数据，包括 7200+ 小时移动操作机器人和双臂机器人数据、1000+ 小时人工标注 UMI 数据，以及 Bridge V2、RT-1、DROID 等公开机器人数据集。完成后训练的模型可在真实环境中根据自然语言指令直接执行沙发整理、餐具收纳、行李箱打包等移动操作任务，实现"开箱即用"。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

## Scaling 实验与基准表现

数据规模实验使用 2.5K、5K、10K、20K 小时 UMI 数据训练，动作预测损失随数据量增加持续降低；模型规模实验对比 2B、5B、10B 三个版本，结果同样随规模提升而改善。更重要的是，预训练更强的模型在后训练后的真实机器人评测（鞋柜收纳、书包打包、桌面整理、沙发收拾等未见环境任务）中也取得更高成功率，说明大规模预训练获得的通用表征能迁移到真实执行。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

在新任务适配方面，每个任务平均数据时长不足 10 小时条件下，Xiaomi-Robotics-1 在四个复杂灵巧操作任务中均大幅超越 Pi-0.5。仿真基准上，RoboCasa365 平均成功率 57.4%（此前最优 46.6%），RoboDojo 以 20.07 平均分和 13.93% 成功率登顶 Leaderboard（原纪录 13.07 分 / 8.80%），VLABench 平均成功率 59.1%，RoboCasa 达 74.5%，超过 RLDX-1、Cosmos Policy、GR00T N1.6 等方法。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

这一路径表明机器人策略模型有机会从依赖小规模任务定制数据，走向更接近基座模型的训练范式：大规模预训练学通用表征 → 跨本体数据后训练对齐 → 少量数据微调适配新任务。对 [[concepts/scaling-laws|Scaling Law]] 而言，具身智能仍处早期探索阶段，随着数据、模型和任务覆盖范围继续扩大，边界将持续被拓展。^[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应.md]

## 相关

- [[entities/xiaomi-robotics-1-embodied-base-model-scaling-2026|Xiaomi-Robotics-1 具身基座模型]] — 本实体对应的英文条目
- [[concepts/embodied-intelligence-frontier|具身智能前沿]] — 领域背景
- 机器人具身智能 — 相关概念
- [[concepts/scaling-laws|Scaling Law]] — 核心方法论

→ [[raw/articles/10万小时训出开箱即用机器人基座模型xiaomi-robotics-1-探索具身智能-scaling-效应|原文存档]]
