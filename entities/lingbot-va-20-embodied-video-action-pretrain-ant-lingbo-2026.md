---
title: "LingBot-VA 2.0 — 蚂蚁灵波具身原生预训练视频-动作基座模型"
created: 2026-07-11
updated: 2026-09-07
type: entity
tags: [embodied-ai, video-action-model, pretraining, robot, ant-group, lingbo]
confidence: 0.65
provenance_state: extracted
sources: [raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LingBot-VA 2.0 — 蚂蚁灵波具身原生预训练视频-动作基座模型

蚂蚁灵波（Ant Lingbo）于 2026 年 7 月发布 LingBot-VA 2.0，全球首个**具身原生**（Embodiment-Native）预训练 VA（Video-Action）基座模型。与传统"先视觉预训练、后接入机器人"的级联方案不同，LingBot-VA 2.0 从架构、数据到训练目标均为机器人量身定制，采用因果 DiT（Causal Diffusion Transformer）+ 稀疏 MoE 主干，在单 GPU 上实现 150Hz 推理频率，双臂任务成功率达 93.6%。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:140-148]

## 技术亮点

- **因果 DiT + 稀疏 MoE 主干**：模型核心架构，结合因果注意力与扩散 Transformer，通过稀疏 MoE 降低推理成本
- **单 GPU 150Hz 推理**：高效推理性能，适合实时机器人控制场景
- **双臂任务成功率 93.6%**：在真实机器人实验中的任务完成率
- **预判式控制**：模型基于对物理动态的预测（而非纯反应式）生成动作序列，在快速变化场景（如冰球对战）中表现显著优于反应式基线

## 应用场景

LingBot-VA 2.0 在多项真实机器人任务中得到验证，包括整理桌面、冰球对战等动态操控场景。研究团队通过端到端 VA 预训练，使机器人具备"预判未来"的能力，从"看到哪打到哪"的反应式操作升级为基于物理预测的主动控制。

## 深度分析

### 1. "具身原生"设计：从级联范式到原生预训练的范式转变

传统的机器人 VLA（Vision-Language-Action）范式采用级联方案：先用互联网数据预训练视觉模型，再将视觉特征接入动作策略网络。LingBot-VA 2.0 的突破在于彻底抛弃了这一路线，从第一天起就采用因果架构从零训练整套模型。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:606-614] 原因在于：改造式路线（将双向注意力模型手术式改造成因果模型）存在天然的预训练知识磨损风险 — 机器人数据本就稀缺，改造过程容易抹掉预训练阶段学到的大规模先验知识。而是用原生因果架构，让模型天然按照"只能看过去、不能看未来"的时间线学习，恰好匹配闭环控制中"当下不能预知未来"的物理现实。

### 2. 语义视觉-动作分词器：超越像素重建的表示学习

LingBot-VA 2.0 的 VAE 分词器是整套系统的基础创新。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:596-604] 传统的视频 VAE 只追求像素重建质量（"压缩得像"），但 LingBot-VA 2.0 的语义视觉-动作分词器在像素重建的同时，额外向冻结的视觉基础模型对齐特征，把语义信息也编码进 latent 中。更重要的是，团队单独训练了一个隐动作模块 — 通过一对逆动力学模型和正向动力学模型，从连续两帧 latent 中反推出中间发生的动作类别。这意味着即使是完全未标注的网络视频，也能被模型学出与动作相关的监督信号。这种设计大幅扩展了可训练数据来源，使模型能从海量互联网视频中习得丰富的物理交互先验。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:604-604]

### 3. 稀疏 MoE + Foresight Reasoning：实时闭环控制的两大工程突破

实时机器人控制的最大挑战是模型计算延迟直接转化为动作延迟。LingBot-VA 2.0 通过两个正交的工程创新突破这一瓶颈。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:618-644]

首先，**稀疏 MoE 架构**让模型总参数做大（视频主干约 13B，总训练参数量约 15.3B）而推理时仅激活其中一部分（每 token 约 2.5B）。经过一致性蒸馏、低精度编译执行、长程注意力优化和运行时开销削减后，推理时间从 965ms/chunk 降至 142ms/chunk，异步控制频率从 33Hz 提升至 225Hz。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:628-630]

其次，**Foresight Reasoning 异步推理机制**让模型在机器人执行当前动作片段的同时，已经在并行脑补下一步。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:636-644] 模型先想象当前动作执行完后的画面状态，再基于这个想象结果提前准备下一步动作。为避免"脑补漂移"，在每次真实观测返回时用最新画面重新校准。本质上这是一套"预测-执行-纠偏"闭环，让计算与动作执行流水线并行。

### 4. 三项真实任务验证与消融分析

研究团队在三个真实维度验证了模型能力：^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:160-166]

- **整理桌面**：考验长程状态维持能力。高维 planner 做任务拆解（左臂回收垃圾、右臂复位文具），视频预测分支天然携带时序状态记忆，避免"断片"返工。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:264-270]
- **传送带抓取**：考验动态目标的时间对齐能力。模型不只识别当前位置，而是预测抓取动作完成瞬间物体的所在位置，将动作执行的时间开销提前纳入计算。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:274-380]
- **抓薯片**：考验精细操作的视觉伺服能力。既要精确把握夹爪与薄脆物体的相对位置，又不能捏碎目标。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:482-487]

在 RoboTwin 2.0 仿真基准上，LingBot-VA 2.0 取得 Clean 93.8%、Randomized 93.4%、Avg 93.6% 的成绩，全面超越 π0.5、Motus 等基线模型。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:648-648] 消融实验验证了新分词器的价值：自研分词器在 50 个任务的 Easy/Hard 上取得 86.6%/83.1%，显著高于通用 WAN2.2 VAE（78.0%/76.0%）。MCP（多步预测）辅助目标带来了 2.3× 训练加速。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:650-652]

### 5. "机器人大脑 2.0"全景：从感知到预测的完整链条

LingBot-VA 2.0 不是孤立发布，而是蚂蚁灵波"机器人大脑 2.0"系列的一部分。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:668-672] LingBot-Depth 2.0 解决空间感知，LingBot-VLA 2.0 解决当下动作执行，LingBot-Video 补上视频生成推理效率短板，LingBot-VA 2.0 将所有能力汇聚到预测式控制。串起来看，这是一条从"看清楚世界"→"理解物理世界"→"在真实世界里连续行动"的完整链条。当机器人本体越来越成熟，行业竞争的关键正在从硬件的灵巧度转向"大脑是否从出生起就真正为物理世界而生"。^[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026.md:676-678]

## 实践启示

1. **原生因果架构优于改造式迁移**：从 LingBot-VA 1.0 到 2.0 的演进证明，对于机器人这样的闭环控制任务，从零训练的因果架构优于从双向模型改造的迁移学习路线。如果需要在机器人场景部署视觉基座模型，建议优先考虑原生因果设计而非改造现有双向模型。

2. **视频数据中的隐动作标签远超人工标注**：通过逆动力学模型从连续帧中自动提取动作信号，使得海量互联网无标注视频成为有效训练数据。建议具身智能团队在数据策略上将"自动提取隐动作标签"作为优先级高于人工标注的选项。

3. **推理效率优化需要系统级设计**：LingBot-VA 2.0 的 225Hz 推理频率不是单一优化达成的，而是 MoE 稀疏激活 + 一致性蒸馏 + 低精度编译 + 长程注意力优化 + 异步推理机制的协同结果。对于实时机器人控制场景，建议将推理 pipeline 的延迟预算拆解到每个子模块，系统性地优化而非局部加速。

4. **Foresight Reasoning 可推广至其他实时控制场景**："预测-执行-纠偏"异步机制不仅适用于机器人，也适用于自动驾驶、工业控制等需要低延迟闭环的场景。核心思路是用预测结果替代真实观测填充间歇期，用真实观测周期性校准以避免漂移。

5. **构建"感知→理解→行动"的全栈能力而非单点突破**：蚂蚁灵波从 Depth → VLA → Video → VA 的发布序列表明，机器人智能的核心竞争力在于全栈能力而非单一基座模型。单点模型即使再强，如果缺乏空间感知、动作执行和推理效率的配套支持，也难以在真实场景中落地。建议具身智能团队在模型研发时同步规划感知、推理和控制的全链路优化。

→ [[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

