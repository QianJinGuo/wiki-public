---
title: "原力灵机 DM0.5：4B 具身基础模型，Zero-Shot 提升 31%"
created: 2026-07-09
updated: 2026-09-07
type: entity
tags: [embodied-ai, lingbot, ant-group, foundation-model, zero-shot, data-flywheel, robot, vla]
sources: [raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出, raw/articles/zero-shot长记忆抗干扰dm05把vla带进真实世界]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 原力灵机 DM0.5：4B 具身基础模型，Zero-Shot 提升 31%

> **DM0.5** 是原力灵机（LingBot / 蚂蚁灵波）于 2026 年 7 月发布的具身基础模型（Embodied Foundation Model），定位为面向开放世界的通用具身基础模型。与上一代 DM0 相比，参数量翻倍至 4B，数据量增加 400%，Zero-Shot 性能提升 31%。^[raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出.md]

## 模型架构

DM0.5 参数量为 4B，相比 DM0 翻倍。其核心设计围绕"更大、更强、更实用"展开，目标是支撑数据飞轮在真实场景中高效运转。^[raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出.md]

## 训练数据组成

DM0.5 的数据策略采用三类高质量数据的混合架构，总计约 15 万小时：^[raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出.md]

- **真机数据**（5 万小时）：高精度操作数据，覆盖 100 多种动作，支持秒级精细指令动作对齐
- **Ego 数据**（10 万小时）：第一视角数据，支持毫米级高精度 3D Landmark 生成
- **场景重建数据**（100 万平方米空间数据）：复杂室内环境建模，降低 Sim2Real Gap

## 数据飞轮策略

DM0.5 的设计核心是将被动"采集型数据"转变为真实业务中持续产生的"场景型数据"。原力灵机与物流机器人公司 Atomix 完成合并后，补上了真实场景侧的关键拼图，使数据飞轮从循环论证走向工程落地。^[raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出.md]

## 相关实体

- [[entities/lingbot-video-moe-embodied-video-2026|LingBot-Video 具身专属 MoE 视频模型]] — 蚂蚁灵波同期发布的具身视频基础模型
- [[entities/lingbot-vla-2-60000h-open-source-vla|LingBot-VLA 2.0]] — 蚂蚁灵波 60,000 小时开源 VLA 模型
- [[entities/embodied-intelligence-sim-to-real-active-inference-behavior-tree-intrinsic-motivation-chenzhiyan-2026-06-17|具身智能 Sim2Real]] — 具身智能领域的 Sim2Real 与行为树方法

→ [[raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出|原文存档]]

## 第 2 来源 — 机器之心（2026-07-09）

> 本来源提供了 DM0.5 在**长记忆、抗干扰、Zero-Shot 泛化**方面的深度分析，重点讨论了 VLA 模型走出实验室环境后面临的真实世界挑战。^[raw/articles/zero-shot长记忆抗干扰dm05把vla带进真实世界.md]

### VLA 模型的真实世界挑战

DM0.5 将 VLA（视觉-语言-动作）模型从精心搭建的"剧本环境"推向真实场景。真实世界中光照变幻、视角漂移以及人类随意干扰，使得纯实验室环境下训练的模型泛化性不足。^[raw/articles/zero-shot长记忆抗干扰dm05把vla带进真实世界.md]

### 关键技术特性

- Zero-Shot 泛化：无需特定场景训练即可应对新环境
- 长记忆机制：跨 session 保持对环境和任务的理解
- 抗干扰能力：在人类随意干扰下保持稳定的操作性能^[raw/articles/zero-shot长记忆抗干扰dm05把vla带进真实世界.md]

→ [[raw/articles/zero-shot长记忆抗干扰dm05把vla带进真实世界|第 2 来源原文]]
