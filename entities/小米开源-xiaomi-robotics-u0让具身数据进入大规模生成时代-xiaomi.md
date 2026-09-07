---
title: "小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代"
created: 2026-07-22
updated: 2026-08-01
type: entity
tags: [robotics, embodied-ai, open-source, xiaomi, world-model, multimodal, video-generation]
source: [[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi]]
provenance_state: extracted
confidence: 0.65
sources:
  - raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代

> 来源：小米技术 | 发布日期：2026-07-15

## 摘要

小米于 2026 年 7 月正式发布并开源 **Xiaomi-Robotics-U0**，一个拥有 380 亿参数的多模态自回归具身生成基础模型，是具身领域首个"通吃"四类任务的统一生成模型——具身场景生成、具身迁移、机器人交互视频生成、通用文生图与图像编辑。该模型在 WorldArena 评测基准上以 126 个模型参评获得总分第一名，真机评测中在未知光照、陌生背景等 OOD 场景下任务完成进度平均提升超 26%。通过 FlashAR+ 推理加速方案，生成效率较原始自回归范式提升 82.9 倍，1024×1024 分辨率单样本吞吐从 450.77 秒/图降至 5.44 秒/图。代码与模型权重已全量开源。 ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]

## 核心要点

1. **统一生成框架**：380亿参数多模态自回归模型，统一覆盖具身场景生成、具身迁移、视频生成、文生图四类任务 ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]
2. **WorldArena 全球第一**：在清华大学、北京大学等联合打造的 WorldArena 榜单上以 126 个模型参评取得总分第一名，指令遵循、交互质量、视角一致性均为第一 ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]
3. **真机验证显著提升**：OOD 场景下任务完成进度平均提升 26.3%，VLA 在面临反光、彩色灯光等极端视觉干扰时仍能保持稳定 ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]
4. **五维解耦结构化控制**：通过工作台布局、前景操作物体、前景无关杂物、光照条件、背景信息五个维度的解耦控制实现精细化生成 ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]
5. **FlashAR+ 极致加速**：对角并行解码 + vLLM 分页 KV 缓存批量调度，生成效率较原始 AR 范式提升 82.9 倍 ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]

## 技术架构

### 统一生成框架

Xiaomi-Robotics-U0 基于多模态 AutoRegressive（AR）架构，采用 IBQ 作为图像 tokenizer，将多种模态表示在同一空间，使用标准多模态 Next Token Prediction 方式进行持续训练。其核心突破在于用统一框架覆盖四类任务，打破了此前"一个任务一个模型"的割裂局面： ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]

1. **具身场景生成（Scene Generation）**：根据文本描述为指定机器人本体生成多视角初始场景，覆盖桌面、厨房、仓库到开放世界环境
2. **具身迁移（Embodied Transfer）**：将已有机器人轨迹迁移到新环境（改变光照、背景、材质、物体等），同时保留原始轨迹中的机械臂位姿和场景布局
3. **机器人交互视频生成（Video Generation）**：基于初始观测和操作指令生成连贯的抓取、放置、形变交互长时序视频，支持虚拟相机运动仿真
4. **通用文生图和图像编辑（Text2Image & Anything2Image）**：保留通用图像生成与编辑能力，使互联网视觉知识迁移到具身智能任务中

### 五维解耦结构化控制

与通用图像生成不同，具身生成需要精确控制每个细节——多视角一致性、场景布局、物体位置必须严格对齐。为此，Xiaomi-Robotics-U0 设计了五维解耦结构化控制范式： ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]

- **工作台布局**：工作区颜色、结构、空间摆放
- **前景操作物体**：任务目标物的种类与姿态
- **前景无关杂物**：工作区上非目标物体的种类与姿态
- **光照条件**：强光、窗外洒落的阳光、彩色灯光
- **背景信息**：室内室外背景物体的布局和样式

这五个维度均可通过自然语言独立调控，实现了跨机器人机型（方舟无限、智元 G1/G2、松灵 PiPER 四种本体）的多视图几何一致性生成。^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]


### FlashAR+ 推理加速

原始自回归模型传统的逐 token 生成方式在面对高分辨率图片生成任务时效率大幅降低。Xiaomi-Robotics-U0 提出的 FlashAR+ 方案在 FlashAR 文生图适配基础上进行扩展，适配了图像编辑、具身迁移等任务，并结合 vLLM 进一步提升效率： ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]

- 对角并行解码 + vLLM 分页 KV 缓存批量调度
- 1024×1024 分辨率单样本吞吐：从 450.77 秒/图提升至 5.44 秒/图
- 生成效率较原始 AR 范式提升 82.9 倍

## 实战验证

### 真机验证

在精细操作（耳机收纳）、可变形物体（毛巾折叠）、长程任务（物品装箱）三类场景中验证： ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]

- 未知光照、陌生背景等 OOD 场景下，使用 Xiaomi-Robotics-U0 扩增数据训练的策略任务完成进度**平均提升 26.3%**
- 面对反光、彩色灯光等极端视觉干扰，模型仍能保持稳定，不会卡死且能够进行自校正
- 首个能在复杂光照和强背景干扰下保留大部分成功率的 VLA 模型

### 对比主流模型

在具身迁移任务上，Xiaomi-Robotics-U0 的深度一致性、结构保真、语义对齐维度指标大幅超越闭源模型 GPT-Image-2.0。核心差距在于：Xiaomi-Robotics-U0 的多视图几何完全统一，而 GPT-Image-2.0 普遍出现跨视图物体错位、空间畸变问题。 ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]

在人工搭建的具身场景生成（200 Easy + 200 Hard）和具身迁移（150 Easy + 150 Hard）基准中，Xiaomi-Robotics-U0 在人类评测中均取得明显领先。^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]


## 深度分析

### "通吃"四类任务的统一生成模型意味着什么

Xiaomi-Robotics-U0 最核心的设计决策是**用统一的自回归框架覆盖四类任务**，而非为每类任务单独训练模型。这一决策有几个深层含义： ^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]

第一，**数据协同效应**。四类任务的训练数据共享同一个模型容量，场景生成学到的空间理解可以迁移到视频生成，文生图学到的视觉概念可以辅助具身迁移。这种跨任务的知识迁移是"一个任务一个模型"的范式无法实现的。^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]


第二，**推理时灵活组合**。用户可以在一次推理中同时执行场景生成+具身迁移+视频生成，形成完整的"仿真→训练→验证"流水线，无需在多个模型之间切换和协调。^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]


第三，**开源生态的价值倍增**。全量开源意味着社区可以在 U0 基础上进行微调、扩展和组合，将其作为具身生成的基础设施而非一个孤立的模型。这与 [[entities/lingbot-dm05-4b-embodied-foundation-model-zero-shot-2026]] 所代表的具身基础模型趋势一致——从专用模型走向统一基础模型。^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]


### 具身数据大规模生成的核心瓶颈被突破

具身智能长期以来面临一个"数据鸿沟"：真实机器人数据采集成本极高（需要硬件、人工、时间），而仿真数据又存在 sim-to-real 差距。Xiaomi-Robotics-U0 的路径是通过**生成式数据增广**来弥合这一鸿沟——用生成模型从已有数据出发，低成本产生多样化的、几何一致的新数据。^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]


五维解耦结构化控制的关键价值在于：它让数据的生成是**可控的**而不是随机的。研究人员可以针对性地增加某一维度的多样性（如光照变化、背景干扰），测试策略模型在特定 OOD 维度上的鲁棒性。这种"定向增广"能力是传统数据增强方法（随机裁剪、颜色抖动等）无法比拟的。^[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi.md]


## 实践启示

1. **具身生成正在从"辅助工具"升级为"核心基础设施"**：Xiaomi-Robotics-U0 表明，生成式数据增广不再只是锦上添花，而是解决具身数据瓶颈的战略级方案。具身智能团队应将数据生成能力视为与模型训练同等重要的能力。
2. **"统一生成模型 > 专用生成模型集合"**：在数据、算力和工程成本受限的情况下，一个统一的多任务生成模型比多个专用模型更具长期性价比——跨任务的知识迁移和推理时的灵活组合带来了超出算术加成的价值。
3. **开源生态决定影响力天花板**：Xiaomi-Robotics-U0 全量开源（代码+权重+多平台部署支持）使其有望成为具身生成社区的基础设施。对于做具身智能的企业，开源策略需要从一开始就纳入产品规划。
4. **FlashAR+ 的工程意义**：82.9 倍的加速不是学术噱头，而是将"实验室能跑"变成"产线能用"的关键一跳。在机器人领域，模型的工程落地速度（推理延迟、部署门槛）往往比学术指标更重要。

## 相关实体

- [[entities/lingbot-dm05-4b-embodied-foundation-model-zero-shot-2026|原力灵机 DM0.5 具身基础模型]]
- [[entities/全球首个小时级世界模型来了中国造已开源|全球首个 hour-level 世界模型]]
- [[entities/最适合机器人的视频基座模型被中国团队开源了|机器人视频基座模型]]
- [[entities/具身智能空间视觉死穴终于被最新顶会彻底解决|具身智能空间视觉]]

→ [[raw/articles/小米开源-xiaomi-robotics-u0让具身数据进入大规模生成时代-xiaomi|原文存档]]
