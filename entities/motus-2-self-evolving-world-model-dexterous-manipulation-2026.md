---
title: "Motus 2：自进化通用世界模型（策略/仿真/评估三接口共享权重）"
created: 2026-09-12
updated: 2026-09-12
type: entity
tags: [world-model, embodied-ai, dexterous-manipulation, self-evolution, mbrl, test-time-planning, video-action-model, agent-planning, model-based-rl, shengshu, arxiv]
sources: [raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026]
confidence: 0.7
---

# Motus 2：自进化通用世界模型

## 一句话定位

生数科技在外滩大会发布的 Motus 2 是面向灵巧操作的「自我演化型通用世界模型」：它把**策略、仿真、评估三种能力压进同一套「视频—动作」共享权重**，用「动作优先」因果掩码保证不剧透未来，再以基于模型的强化学习（MBRL）+ 推理时规划把推演结果变成训练信号，从而让机器人「在自己的脑海里试验」。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

## 为什么需要「第三个接口」：评估器

过去机器人通用灵巧操作常被视作「观测→动作」的端到端映射，行为克隆靠死记硬背，既无物理常识，也无法评判动作优劣，遇到微小扰动不能从自身交互中改进。世界模型的出现补上了「仿真器」这一环——能预测动作之后的画面；但仅会「预言」并不够：预测出的未来画面本身不告诉你结果好不好。Motus 2 的关键推进就是把**价值评估与策略改进接进这条链路**，补上缺失的评估器（价值模型）。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

> 「策略回答『尝试什么动作』，仿真器回答『会发生什么』，评估器回答『这个结果好不好』。」

三个角色共用一套参数，靠不同输入方式切换：普通控制只需生成动作，需要规划或策略优化时再调用预测与评估能力。这与清华朱军团队通用世界模型（GWM）的核心原则同构——动作不是附加的辅助输出，而必须是物理交互的因果接口。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

## 架构：一个模型，三个功能接口

Motus 2 彻底改掉了上一代把「策略模型 + 仿真器」并列的架构，把三种能力融进同一个视频—动作共享权重模型：^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

- **策略（世界-动作模型，WAM）**：看指令、机器人状态和视觉历史，生成动作。
- **仿真器（动作条件世界模型，AC-WM）**：给定当前状态和动作，预测未来画面。
- **评估器（价值模型，VM）**：看动作及其推演结果，判断任务进展。

要让同一个模型既能预测未来又能真实控制机器人，关键是**动作不能提前看到执行之后才会出现的信息**。Motus 2 设计了「动作优先」（Action-first）因果流与掩码机制：块内自回归，中/后训练中动作优先掩码阻止当前未来视频信息进入动作预测，而未来视频 token 可以读取动作，价值查询则可以同时读取两者，把因果链条固化为「生成动作 → 基于动作推演未来 → 评估未来价值」。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

## 闭环自进化：把试错搬进「脑子」

Motus 2 从推理与训练两个阶段实现自进化闭环，避免真实机器人千万次物理试错的成本：^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

- **推理阶段（Best-of-N + 测试时规划）**：先生成 N 个候选动作，在脑内模拟器里全部推演一遍，价值评估器给每种未来打分，选最高分执行；每执行一段再回到真实世界重新规划。机制上类似 AlphaGo 的推演规划。
- **训练阶段（MBRL 反哺策略）**：世界模型推演出的未来与评分反向作为强化学习奖励信号，直接微调动作策略；高分动作被强化、低分动作被抑制。

由此得到一个观念级转变：**失败轨迹从「工业废料」变成无价数据**——它能教会模型「为什么这个动作会让世界变成糟糕的局面」。候选生成、后果预测与价值评估共同提供策略学习信号，规划改变本次选择，MBRL 改变后续候选动作的生成分布。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

## 数据金字塔：13 万小时人类日常

针对机器人遥操作数据稀缺且昂贵的「数据死穴」，Motus 2 走了一条以人类经验补机器人数据的路线，构筑人类数据金字塔：第一阶段用单目 Ego 视频预训练建立基础世界知识；第二阶段加入双目观测与人类动作做视频—动作联合预训练；第三阶段做机器人领域适配。此外，模型通过**记忆机制与轻量级触觉专家**处理单帧视觉难以完整观察的状态。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

## 实测数字

在放置小球、多指操作、贴合橡皮、拧灯泡、放置手机五项主要真机任务中，Motus 2 平均成功率达到 **84%**。在单独评测的放置手机、多指操作两项高难度真机任务中，仅用基础策略平均成功率 **65%**，引入 MBRL 后升至 **72.5%**，再叠加推理时规划达到 **75%**，累计提升 10 个百分点——即在无人类外部干涉的情况下，依靠自身推演与评价实现策略持续进化。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

- 项目页：<https://motus-robotics.github.io/motus2/>
- 论文：<https://arxiv.org/abs/2608.30237>

## 与既有世界模型工作的关系

Motus 2 落在「生成式世界模型 → 可交互世界模型 → 可评估世界模型」这条演化线的第三段：[[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0]] 与 [[entities/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026|LingBot-VA 2.0]] 代表「跨本体统一表征 + 视频—动作预训练」的路线，而 Motus 2 的增量在于**把价值评估显式建模为模型的第三个接口**，使世界模型从「能预测」升级为「能判断好坏并据此改进策略」。这与 [[concepts/world-models|世界模型]] 综述中「世界模型作为 Agent 内部模拟器」的定位一致，也与 [[entities/ropedia-homie-gen2-experience-scaling-law-embodied-ai-2026|Homie Gen2 经验 Scaling Law]] 所强调的「用真实交互经验驱动进化」互补：Motus 2 把经验获取环节前移到了模型内部的推演。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

对 AI Agent 系统设计的可迁移启发有三点：其一，**评估器与执行器共享权重、按输入切换**是一种低成本的「自我评判」实现，可类比 Agent 运行时的 critic 角色；其二，**动作优先掩码**是保证「决策不使用未来信息」的通用因果约束写法，可用于任何带前瞻的 plan-then-execute 架构；其三，**把失败轨迹转化为奖励信号**而非丢弃，是把推理期推演变成训练期收益的关键机制，对应 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 之外的另一条「自监督信号」来源。^[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026.md]

相关工作见 [[concepts/embodied-intelligence-frontier|具身智能前沿]]、[[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM 强化学习算法演化]]。

→ [[raw/articles/motus-2-self-evolving-world-model-dexterous-manipulation-2026|原文存档]]
