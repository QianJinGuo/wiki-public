---
title: "GraphPlanner — 图记忆网络驱动多智能体 LLM 工作流路由"
created: 2026-07-12
updated: 2026-11-27
type: entity
tags: [ai, multi-agent, routing, graph, llm, workflow, planning, research]
confidence: 0.8
provenance_state: extracted
sources: [raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# GraphPlanner — 图记忆网络驱动多智能体 LLM 工作流路由

> **Background**：本文基于 UIUC 研究团队提出的 GraphPlanner 框架整理。论文发表于 arXiv 2604.23626，代码开源于 github.com/ulab-uiuc/GraphPlanner。原报道来自新智元。

GraphPlanner 是 UIUC 提出的多智能体 LLM 路由框架，将传统的模型选择路由（LLM Router）升级为动态工作流生成（Agentic Workflow Generation）。其核心创新在于：Router 不仅决定调用哪个模型，还决定每个模型应承担的角色（Planner / Executor / Summarizer），并利用图结构记忆网络（GARNet）记录历史交互以指导决策。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md]

## 摘要

当 LLM 从「单模型回答」走向「多模型协作」，路由随之升维：不只要知道调用哪个 LLM，还要知道它扮演什么角色、在什么步骤调用、如何与其他模型协作。现有 Router 多停留在 query-level 模型选择（Qwen / LLaMA / Gemma / Mixtral 里挑一个）；multi-round router（如 Router-R1 的「思考—路由—聚合」）虽能多次调用，本质仍是连续选择 backbone，不分解任务也不建模角色分工。GraphPlanner 把 routing 重定义为 sequential decision-making：每步同时决定「用哪个 LLM」与「激活哪个角色」，并用异构图记忆组织当前工作流与历史经验，使 Router 从「模型选择器」升级为「多智能体规划器」。

## 核心要点

- **动作空间换维**：从 `LLM Backbone` 扩展为 `Action = Agent Role + LLM Backbone`，同时决定「谁做」与「用哪个模型做」。
- **三类角色**：Planner 拆解 atomic sub-queries、Executor 回答 query 或子问题、Summarizer 聚合中间结果生成回答。
- **GARNet 是主要增益来源**：把本轮 workflow 状态与历史 traces（query、模型、角色、输出、accuracy、cost）连成异构图。
- **MDP + PPO**：奖励含 task utility 与 computational cost，α 控制 accuracy-cost trade-off；学到的是动态策略而非固定模板。
- **效果与效率兼得**：Phase 2 平均准确率较最强 baseline 提升约 9.3%；较 Router-R1 等 RL-based multi-round router 显著降低训练 GPU compute。
- **泛化与推理模式**：out-of-domain（LogicGrid、MGSM、CommonGen）78% 平均准确率，加 unseen LLMs 仍稳定；支持 Inductive（轻量部署）与 Transductive（用历史 memory，性能更高）。

## 核心架构

### GARNet：异构图记忆网络

- **Workflow Memory Graph**：当前 query 在本轮推理中生成的子问题、角色调用和中间回复
- **Historical Memory Graph**：历史任务中的 query、response、LLM-role 交互、accuracy 和 cost

GARNet 把 query node、response node、LLM-role node 与 accuracy-cost edge 组织成异构图，通过共享 role hub nodes 连接当前工作流与历史记忆，使 Router 能利用历史积累的模型能力画像与协作模式。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md]

### 节点 / 边与动作空间

- **节点**：query node（任务与子问题）、response node（回答与中间结果）、LLM-role node（模型在角色下的实例化，如「Qwen-as-Planner」）。
- **边与枢纽**：accuracy-cost edge 承载单次调用的效果与开销；role hub 跨图共享，把当前 workflow 与历史 memory 对齐到同一组角色。
- **动作**：每步选 `(Agent Role, LLM Backbone)`。简单问题一步 Executor 完成；复杂数学／代码／多跳推理先 Planner 拆解、多 Executor 分头解决、最后 Summarizer 汇总。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md]

### Router 网络与训练信号

Router 是轻量级图策略网络，读 GARNet 聚合的图表示后输出下一步 `(Role, Backbone)` 分布。因含离散角色选择与结果汇总、难以端到端求导，故 training signal 来自 PPO；奖励除答对的 task utility 外还惩罚冗余调用。

## 训练与实验

实验覆盖 14 个任务、6 个领域：Math、Code、Commonsense Reasoning、World Knowledge、Popular Benchmark、Out-of-domain Testing，分两阶段。**Phase 1**（固定工作流）仅优化各 agent 的 backbone，GraphPlanner 仍取得最高平均准确率，说明图记忆增强的 routing policy 能更好分配角色；**Phase 2**（自由生成 workflow）优势扩大，相比最强 baseline 提升约 9.3% 平均准确率——query-specific workflow 比固定工作流更适合复杂任务。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md]

**Baselines**：single-round 与 multi-round router（含 Router-R1 等 RL-based 方法）两阶段均被超越；out-of-domain 任务 LogicGrid、MGSM、CommonGen 上取得 78% 平均准确率，优于 GraphRouter、RouterDC、Router-R1。训练 GPU compute 显著低于 Router-R1，加入训练阶段未见过的 LLM 时仍稳定更优。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md]

## 深度分析

### 从调用轨迹到可传播信号：GARNet 的图构造细节

论文把「一次调用留下什么」列成六字段清单：query 是什么、哪个模型被调用、扮演什么角色、输出了什么、成本多少、最终是否正确。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:85-89] 六个字段各支撑一类判断：query 用于相似任务召回，model×role 组合累积能力画像，cost 与 correctness 把「值不值」变成可计算的量。GARNet 进而把信息分装进两张图——Workflow Memory Graph 只装本轮推理中生成的子问题、角色调用与中间回复，Historical Memory Graph 装历史任务的 query、response、LLM-role 交互与 accuracy-cost——再通过共享 role hub nodes 把两图对齐，当前决策因此能读取历史沉淀的协作模式，而非文本相似的历史条目。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:91-99]

这解释了图记忆优于扁平检索的机制：向量相似度只作用于 query 文本，而「哪个模型适合当 Planner」是挂在 model×role 组合上的关系性知识，不在任何单条 trace 上。role hub 把角色升格为跨任务对齐键——字面毫不相似的任务，只要同走「拆解→执行→汇总」的分工模式，就能经由同一组 hub 复用历史经验——多智能体分工结构由此可复用（参见 [[concepts/multi-agent-collaboration-patterns|Multi-Agent Collaboration Patterns]]）。

### 为什么训练必须绕开端到端求导

动作空间换维是根因：每步决策是二元组 Action = Agent Role + LLM Backbone，对应 Planner 拆解 atomic sub-queries、Executor 作答、Summarizer 聚合三类基础角色。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:60-72] 复杂的数学、代码、多跳推理先拆解、多 Executor 分头、最后汇总，简单问题则一步 Executor 完成，结构由策略而非模板决定。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:74-80] 整个链条含离散角色选择、模型调用、子问题分解与结果汇总，无法直接求导，论文故将其建模为 MDP 并以 PPO 训练。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:105-107] 奖励三项分工明确：答对给 task utility，每步调用计 computational cost，超参 α 调节 accuracy-cost 权衡；学到的产物不是固定工作流模板，而是按 query 自适应决定「是否规划、是否拆解、调用几个模型、何时收口」的动态策略。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:109-117] 效率上，较 Router-R1 等 RL-based 方法训练 GPU compute 显著更低，决策由轻量级图策略网络完成。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:133-135]

对照面在于：multi-round router（如 Router-R1 的「思考—路由—聚合」）虽可多次调用，本质仍是连续选 backbone，不显式建模角色分工与协作结构。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:43-45] 动作空间换维后，优化目标即从「选对模型」变为「组织一次协作」。

### 基准数字要分层审读

14 个任务、6 个领域的两阶段设计本身构成一组消融。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:121-133] Phase 1 固定工作流、只优化各 agent 的 backbone 选择，GraphPlanner 仍取得最高平均准确率——第一层增益来自图记忆改善了「角色-模型」分配；Phase 2 放开自由生成 workflow，优势扩大到约 9.3%——第二层增益来自 query-specific 的结构本身。分层看才能判断可迁移性：若你的场景工作流是写死的，只能拿到第一层增益。泛化证据同样分两类：out-of-domain 任务（LogicGrid、MGSM、CommonGen）取得 78% 平均准确率，优于 GraphRouter、RouterDC、Router-R1；加入训练时未见过的 LLM 仍稳定更优，论文将其归因于 LLM-role graph 与历史交互记忆学到的是可迁移的协作模式，而非「某模型在某任务上」的标签记忆。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:137-143] 需要指出的空档：全文没有给出 token 开销或延迟的绝对数字，「成本更低」仅有训练 GPU compute 的定性对比，生产环境的推理成本收益仍需自行测量。

### 部署模式与失败边界

两种推理模式直接对应部署形态：Inductive 不依赖保留的历史交互、部署轻量；Transductive 复用历史 interaction memory、性能更高，可按需在效率与性能间切换。^[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划.md:145-151] 这组开关同时暴露了四条边界。其一，角色集合只有三个，缺 Verifier / Critic / Tool-user，「必须验证或调用外部世界」的任务在动作空间内无法表达，能力上限在建模阶段即被锁死。其二，α 在训练分布上调优，生产中的延迟、限流、缓存命中难以进入奖励刻画，策略可能偏离成本最优。其三，Transductive 的收益以「有干净历史」为前提——冷启动只能退回 Inductive 档位；而 traces 持续累积会推高检索成本，分布漂移则会污染能力画像，需要淘汰/衰减机制（参见 [[concepts/memory-consolidation-decay|Memory Consolidation & Decay]]）。其四，评估停留在基准数据集，缺带副作用的端到端验证；query-specific workflow 生成的结构决策如何审计与复用，仍是开放问题（参见 [[concepts/agentic-workflow-patterns|Agentic Workflow Patterns]]）。

## 实践启示

1. **动作空间从「选模型」升级为「选角色 × 选模型」**：多模型系统的收益常来自分工结构，不只是换更强 backbone。
2. **显式记录 accuracy 与 cost 并让二者进入决策**：每次调用的效果与费用都应成为可查询的边。
3. **用「角色」作跨任务对齐键**：中间层抽象应是角色/能力而非模型名或 prompt 模板；且能一步做完就别拆。
4. **先定角色集合，再定推理模式**：优先补齐 Verifier / Tool-user，否则能力上限在设计阶段就被锁死；求轻量用 Inductive，求上限用 Transductive 并配套记忆淘汰。

## 相关实体

- 相关概念：[[concepts/harness-engineering-framework|Harness Engineering]]、[[concepts/agent-role-specialization|Agent Role Specialization]]、[[concepts/agent-orchestration-patterns|Agent Orchestration Patterns]]、[[concepts/agent-memory-architecture|Agent Memory Architecture]]
- 训练与路由：[[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法]]、[[entities/layerrecall-memory-router-zju-hku-arxiv-2026|LayerRecall Memory Router]]、[[entities/cursor-router-production-model-routing-2026|Cursor Router 模型路由]]；相关：Agent 架构

→ [[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划|原文存档]]
