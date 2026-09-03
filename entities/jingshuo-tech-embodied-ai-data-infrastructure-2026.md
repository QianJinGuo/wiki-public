---
title: "景烁科技 — 具身智能数据基础设施"
created: 2026-07-15
updated: 2026-07-27
type: entity
tags: [embodied-ai, data-infrastructure, world-model, simulation, autonomous-driving]
sources: [raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15]
confidence: 0.65
---

# 景烁科技 — 具身智能数据基础设施

景烁科技（Jingshuo Tech）是一家具身智能数据基础设施公司，由文远知行（WeRide）内部孵化，CEO 霍达（文远 001 号员工）和 President & COO 韩明联合创立，2026 年正式独立运营。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

## 核心定位

不做整机，不做单纯的大脑，押注物理 AI 的基础设施——数据和世界模型技术体系。在行业追逐"大脑"和"通用模型"时，专注底层的"经验体系"和"世界认知"系统搭建。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

## 技术路线

### 从自动驾驶到具身智能

团队在文远知行期间完整经历了自动驾驶的瓶颈迁移史：

1. **拼算法** → 2. **拼算力** → 3. **拼数据**（端到端时代）

贯穿三条暗线是数据基础设施持续建设。最终认识到：再好的算法、再大的算力，没有高质量数据喂给模型，迭代就停在原地。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

### 仿真生成长尾场景

面对实测数据天然劣势（车队数量远不如车企），景烁团队率先用仿真器生成大量长尾场景。不是模仿类视频生成，而是真正理解物理法则、因果关系的"世界模拟器"，后演进为文远知行的 GENESIS 世界模型。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

### 数据设计理念

核心理念：**不要在茫茫数据里捞"钻石"，而是直接人工生成"钻石"**。

这种选择在具身智能时代被放大——自动驾驶至少有一个早期模型可上路，但具身智能没有任何一个具备最低泛化性的工程化模型，更没有 ChatGPT 那样现成的互联网数据。制约核心因素就是缺少高质量数据基础设施形成闭环。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

### EGO 路径验证

EGO（第一人称视角采集）路径被验证可行后，团队敏锐察觉物理 AI 的 Scaling Law 触发第一次有了可能性，于是决定正式独立。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

## 深度分析

### 具身智能数据基础设施的稀缺性

景烁科技选择了一个被行业忽视的赛道——不做整机、不做「大脑」，而是做数据和世界模型基础设施。这个选择背后的判断是：具身智能的瓶颈不是算法（Transformer 已被证明在物理任务中有效），也不是硬件（中国制造业把本体性能做到极致），而是缺少高质量数据基础设施形成闭环。没有数据，模型迭代不动，除了 Demo 几乎无法落地。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

### 从自动驾驶到具身智能：十年数据信仰的延续

团队完整经历了自动驾驶的「瓶颈迁移史」：拼算法 → 拼算力 → 拼数据。在文远知行期间积累的工程化方法论——用仿真器生成长尾场景、以世界模型驱动数据闭环——被无缝迁移到具身智能赛道。这种「经历过完整淘汰赛」的 knowhow 是景烁最核心的壁垒：知道模型缺什么、什么数据真正有用、怎么设计数据能让模型能力跃升。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

### 三层产品架构的飞轮效应

景烁的产品体系分为三层：WorldEngine（标准化数据模型底座，覆盖采集→治理→标注→合成→测评→部署的完整闭环）、GENESIS-Robotics（世界模型核心引擎，采用 Transfusion 路线在同一个 Transformer 中融合语言、策略、图像、视频，共享参数）、SkillForge（物理 AI 资产引擎，200+ 标准化技能包覆盖家庭、制造、零售、教育四大领域）。三层环环相扣，形成飞轮：模型越强→合成数据越好→下游模型更强→采集策略更精准→真实数据质量更高→模型更强。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

### 跨本体泛化的工程实践

SkillForge 的核心产品理念是「数据基础设施即服务」——客户拿到的不是「一堆视频」，而是「一个经过验证可以直接用于训练的技能」。通过 WorldEngine 全流程验证后附 L1/L2/L3 三层评测结果，客户只需要将人的动作映射到自己的末端执行器上，适配到不同自由度和关节。这种跨本体的设计思路，为具身智能的数据工业化提供了可行路径。^[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15.md]

## 实践启示

1. **数据基础设施优先于模型竞争**：景烁的案例表明，在物理 AI 领域，缺少高质量数据闭环是比算法落后更致命的瓶颈。创业公司在追逐「大脑」之前，应先评估数据基础设施是否到位。
2. **仿真器不是替代真实数据，而是弥补长尾**：用仿真生成「钻石」而非在茫茫数据里捞「钻石」——仿真的价值在于定向生成有意义的难例，而非替代真实世界数据。
3. **世界模型是数据飞轮的核心引擎**：只有拥有自己的世界模型，才能直接定义「什么数据有用」。GENESIS-Robotics 的 Transfusion 架构让语言、策略、视频共享参数，三种能力互相增强。
4. **跨本体泛化需从数据设计阶段开始**：SkillForge 的设计表明，跨本体的泛化不是模型训练后「涌现」出来的，而是从数据采集阶段就设计好的——特定的状态分布 + 动作分布 + 评测标准决定了最终的迁移能力。
5. **硬件采集同样重要：全链路同源同标**：EGOK 采集设备的 8ms 延迟、280g 重量、5 小时续航本身不是竞争壁垒，但「采集发生时手—物—场景—动作已经对齐」的设计——全链路模组同源同标，不需要事后人工拼接——才是工程化落地的关键细节。

→ [[raw/articles/jingshuo-tech-embodied-ai-data-infrastructure-2026-07-15|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

