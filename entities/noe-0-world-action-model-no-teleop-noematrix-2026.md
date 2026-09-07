---
title: "Noe-0 世界动作模型（无本体数据训练）"
created: 2026-08-27
updated: 2026-09-07
type: entity
tags: [embodied-ai, world-model, wam, robot, no-teleoperation, cross-embodiment, data-infra]
confidence: 0.68
provenance_state: extracted
sources: [raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Noe-0 世界动作模型（无本体数据训练）

穹彻智能（Noematrix）2026-08-26 发布具身智能预训练模型 **Noe-0**——训练全链路采用**无本体数据**的世界动作模型（World Action Model, WAM），证明了完全不依赖传统遥操作数据也能完成具身世界模型的预训练、后训练并在真实机器人上形成执行任务的能力。^[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布.md]

## 核心范式：全链路无本体数据

传统遥操数据在集中场地标准化工位采集（相似桌面、有限物体、预设任务流程），本质是为数据采集人为构造的环境。Noe-0 的所有自采训练数据都来自人在真实环境中直接完成操作的记录——超 50 个城市、数十万小时、数十万种任务类型。^[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布.md]

关键设计是**预训练与后训练始终沿用同一种无本体数据路线**。若预训练用无本体数据、后训练切回遥操作数据，两阶段会出现数据分布和动作模式的断层——模型在预训练学到的自然操作规律，需要在后训练重新适配一套不匹配的动作表达。Noe-0 从预训练到后训练保持数据范式连续，避免后训练成为链路能力瓶颈。^[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布.md]

## WAM 架构与跨本体迁移

Noe-0 采用 World Action Model 架构（对应「VLAs are dead, long live World Action Models」路线，见 [[entities/4d-wam-world-action-model-3d-trajectory-alignment-2026|WAM]]），以视频预测作为核心高层学习目标：接收连续历史帧与状态信息，预测未来视觉状态，同时把状态变化理解转化为可执行动作。动作侧模型规模相对较小，更换末端构型时无需重训整套模型。^[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布.md]

穹彻团队发现：即使采集数据与推理部署之间存在本体差异，**Pixel Prediction 依然能显著提升跨本体迁移**。原因在于像素预测提供隐式反事实推理——模型必须区分哪些视觉变化与任务完成相关、哪些只是本体外观或背景差异，从而过滤掉本体绑定的表面信息、抓取驱动任务进展的物理本质。^[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布.md]

## AI-Native 数据基础设施

穹彻强调「分布规模化」而非「数据量规模化」：`Distributional Scaling = More Data × More Tasks × More Objects × More Spaces × More Real-world Variation`。为了支撑全链路无本体数据，团队打造了 RoboPocket + Data Agent 的数据管线。^[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布.md]

- **RoboPocket**：端侧采集入口，覆盖 50+ 城市、18 省级行政区，三组传感器（左右手+头部）采集视觉与动作轨迹；累计数十万小时，峰值 1000+ 采集员同时参与。任务由采集者在真实场景自主产生（煮面、接线装配、填充缝隙等），产生「数十万小时对应数十万种任务类型、平均每种约 1 小时」的广度远大于深度的数据特征。^[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布.md]
- **Data Agent**：连接数据管理者与分散采集员的智能体，覆盖任务生成、数据查询统计、质量问题处理、通知推送、权限管控五项功能，把 AI 嵌入任务产生→采集→质量改进的完整管理闭环。^[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布.md]

## 定位与关联

Noe-0 与已有具身世界模型实体同属 WAM/无本体/跨本体前沿：[[entities/全球首个具身原生世界动作模型来了|LingBot-VA 2.0]]（蚂蚁灵波具身原生 WAM）、[[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0]]（跨本体多模态物理世界模型）、[[entities/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026|VBot 跨本体基因]]。其「无本体数据 + WAM 完美匹配」的结论与数据基础设施视角，可与 [[entities/embodied-ai-data-market-landscape-97-players-44-billion-2026|具身数据市场]]、[[entities/zhuji-dynamics-pre-ipo-embodied-ai-2026|具身智能创业]] 对照。

→ [[raw/articles/突破遥操瓶颈全新无本体数据世界动作模型noe-0发布|原文存档]]
