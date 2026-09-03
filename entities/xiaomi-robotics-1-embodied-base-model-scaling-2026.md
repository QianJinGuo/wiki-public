---
title: "Xiaomi-Robotics-1: 10万小时训出开箱即用机器人基座模型，探索具身智能 Scaling 效应"
created: 2026-07-23
updated: 2026-08-29
type: entity
tags: [embodied-ai, robotics, xiaomi, scaling, robot-foundation-model, real-world-data, robotics-training]
confidence: 0.75
provenance_state: extracted
sources: [raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026]
---

# Xiaomi-Robotics-1: 10万小时训出开箱即用机器人基座模型，探索具身智能 Scaling 效应

Xiaomi-Robotics-1 是小米于 2026 年 7 月发布的面向真实移动操作任务的具身基座模型。模型基于 10 万小时真实世界操作轨迹数据进行预训练，结合跨本体数据完成后训练，在未见环境真实机器人任务中展现稳定的规模化收益。项目主页：https://robotics.xiaomi.com/xiaomi-robotics-1.html ^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]

## 核心要点

- **10万小时真实世界数据**：使用 Universal Manipulation Interface（UMI）设备采集，覆盖家庭、商业空间、工业、办公室、户外等多类环境 ^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]
- **预训练+后训练两阶段范式**：预训练学习通用动作生成能力，后训练解决本体对齐和指令对齐 ^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]
- **清晰的 Scaling 效应**：预训练数据规模（2.5K→20K 小时）和模型规模（2B→5B→10B 参数）提升均带来动作预测精度持续改善 ^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]
- **真实机器人迁移验证**：预训练更强的模型，在后训练后真实任务执行中成功率更高，说明大规模预训练表征可迁移到真实执行 ^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]
- **少量数据高效适配**：新任务平均 <10 小时微调数据即超越 Pi-0.5，四个测试任务全面领先 ^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]
- **仿真基准领先**：RoboCasa365 (57.4% SR)、RoboDojo (20.07 分登顶)、VLABench (59.1% SR)、RoboCasa (74.5% SR) 四项基准均达 SoTA ^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]

## 技术架构

### 数据采集与标注

使用 Universal Manipulation Interface（UMI）设备采集 10 万小时真实世界操作轨迹。自动标注流程将长轨迹切分为固定长度片段，使用视觉语言模型对片段中的夹爪状态变化和交互物体状态变化进行描述。全量 10 万小时数据标注可在约 2 周内完成。^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]

### 两阶段训练

1. **预训练**：学习通用动作生成能力。给定当前视觉观察和语言描述，预测一段动作序列使场景从当前状态向目标状态变化。
2. **后训练**：约 10000 小时跨本体数据，包括 7200+ 小时移动操作机器人和双臂机器人数据、1000+ 小时人工标注 UMI 数据，以及 Bridge V2、RT-1、DROID 等公开数据集。
   - 本体对齐：UMI 数据能力迁移到真实机器人本体
   - 指令对齐：状态变化描述→自然语言指令执行

## 与同类工作的对比

与 [[entities/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi|Xiaomi-Robotics-U0]]（380B 参数生成式世界模型）不同，Xiaomi-Robotics-1 专注于**具身操作策略**，是机器人的"大脑"而非"生成器"。U0 解决数据生成问题，1 解决任务执行问题，两者在小米具身智能体系中是互补关系。^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]


在仿真基准上，Xiaomi-Robotics-1 大幅超越 [[entities/genesis-ai-gene-25-embodied-foundation-model|GENESIS-AI/Gene-25]]、Cosmos Policy、GR00T N1.6、Pi-0.5 等已有方法。^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]


## 意义与影响

Xiaomi-Robotics-1 验证了一条面向 具身智能 的可规模化训练路径：^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]

1. 大规模预训练学习通用动作表征
2. 跨本体后训练迁移到真实执行
3. 少量下游微调快速适配新任务

这是机器人基座模型领域从"小规模任务定制"走向"大规模基座模型"范式转变的重要实证。与 [[concepts/scaling-laws|Scaling Law]] 在语言模型中的应用类似，Xiaomi-Robotics-1 证明数据规模和模型容量在具身策略模型中同样产生可预测的性能增长。^[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026.md]

→ [[raw/articles/xiaomi-robotics-1-embodied-base-model-scaling-2026|原文存档]]
