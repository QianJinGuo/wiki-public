---
title: "原力灵机 DM0.5：4B 具身基础模型，Zero-Shot 提升 31%"
created: 2026-07-09
updated: 2026-09-13
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

## 深度分析

### 4B 参数 + 15 万小时：scaling 轴在数据覆盖度而非参数量

DM0.5 只把参数从 DM0 翻倍到 4B，却把数据量提高 400%（约 15 万小时），并让单卡 4090 在 18 小时内完成一个全新下游任务的专家级微调。这说明赌注押在数据覆盖度上：4B 主干（Gemma3 4B 骨干 + 680M Action Expert）已足以承载连续动作生成，继续堆参数只会推高部署成本，却解决不了"模型没见过"这个根本问题。Zero-Shot 提升 31%、Few-shot 提升 45% 的增益，更可能来自见过更多本体、任务与视角组合。^[raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出.md, raw/articles/zero-shot长记忆抗干扰dm05把vla带进真实世界.md]

### 三类数据的分工：真机给精度、Ego 给视角、场景重建给环境先验

5 万小时真机数据覆盖 100 多种动作并做到秒级精细指令对齐，解决"如何在物理世界完成操作"；10 万小时 Ego 第一视角数据支持毫米级高精度 3D Landmark 生成，这类空间原生表征与 [[entities/lingbot-vision-spatial-native-vision-foundation-model-ant|LingBot-Vision 空间原生视觉模型]] 同源；100 万平方米场景重建数据直接对 Sim2Real Gap 开刀，用对真实空间的建模补足纯仿真的分布偏移。三者缺一不可：没有真机数据动作不准，没有 Ego 数据视角一换就废，没有场景重建数据换个房间就崩。^[raw/articles/zero-shot长记忆抗干扰dm05把vla带进真实世界.md]

### 数据飞轮的关键不是采集预算，而是"场景型数据"的组织闭环

采集型数据的成本随时间线性增长；场景型数据是业务运行的副产品，边际成本趋近于零。与物流机器人公司 Atomix 合并，本质是在组织内部解决"谁来提供真实场景"，这与 [[entities/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026|LiOS 端云协同虚实迁移基础设施]] 把场景侧与模型侧咬合起来的思路一致；再叠加世界模型驱动的 DFOL2.0 闭环（真实作业与失败数据持续回流云端），官方结果是真机数据需求下降 60%、训练成本降低 40%。飞轮的转速取决于失败样本的回流速度，而不是采集规模。^[raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出.md]

### 长记忆 + 抗干扰：VLA 走出"剧本环境"的两道门槛

原生 60 秒记忆（行业普遍不超过 10 秒）把任务从"对当前画面做映射"升级为"对任务进程做建模"：杯子被拿走、初始位置已不在画面中时，模型仍能依据压缩后的历史 token 把它放回原位。抗干扰则靠 Sys2 规划、Sys1 高频响应的双系统架构，针对相机位姿骤变与人类动作打断这两类长尾干扰专门强化——真实部署里，干扰是常态而非噪声。这也解释了轨迹对齐层为何把监督从"固定时间点对齐"改成"轨迹进展对齐"：避免模型学到采集者的节奏。^[raw/articles/zero-shot长记忆抗干扰dm05把vla带进真实世界.md]

## 实践启示

对具身 AI 团队而言，DM0.5 可迁移的经验不在榜单数字，而在数据与评测的组织方式。^[raw/articles/zero-shot提升31原力灵机dm05登场15万小时数据喂出.md]

1. **优先建设"场景型数据"来源**：把采集从一次性项目改成业务运行的副产品，先回答"模型进场后由谁持续产生真实任务样本与失败样本"。
2. **数据治理预算不低于训练预算**：异常值与物理不连续片段剔除、静止帧过滤、低价值动作筛除、动作表示统一、跨模态重标注——这比换架构更直接地决定部署时的抓偏与中断率。
3. **把 Ego 视角与 3D Landmark 质量当一等指标**：两者分别提供"以人为师"的动作环境先验与空间表征锚点，采集阶段就该定验收标准。
4. **把 Sim2Real 场景重建数据当资产而非开销**：用真实空间的场景重建对冲仿真分布偏移，是降低"换个环境就失效"性价比最高的投入。
5. **Zero-Shot 与抗干扰分开评测**：前者决定能力上限，后者决定能否在非剧本环境持续运行，二者的训练手段并不相同。
6. **为长时序任务设计记忆层，并保留退化路径**：按数十秒级上下文做历史压缩与采样，同时要求历史缺失时能退化为仅依赖当前观测的策略。

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
