---
title: "GraphPlanner — 图记忆网络驱动多智能体 LLM 工作流路由"
created: 2026-07-12
updated: 2026-09-10
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

### 为什么图记忆优于扁平检索

一次调用的价值不由单字段决定，而由「query 类型 × 角色 × 模型」的组合关系决定；扁平检索（历史 traces 塞 context 或向量库）只能靠相似度猜测，异构图却把 accuracy-cost 建成边、角色建成共享 hub，等于把「哪个模型适合当 Planner」编码成可沿边传播的结构信号。role hub 也是跨任务对齐锚点：query 字面相似度可能极低，却共享「拆解 → 并行执行 → 汇总」的同一角色模式。

### Router 何时能泛化到未见过的 workflow

两条证据（out-of-domain 78%、unseen LLMs 稳定）的机制在于策略学到的是「角色 × 能力 × 成本」协作模式，而非模型名或任务模板。边界：新任务推理结构落在已知角色组合覆盖内（多跳拆解、可并行子问题、需一致化输出的聚合）时迁移最有效；需要三角色之外的能力（长程工具使用、环境交互、外部检索）时角色空间即成上限。

### 失败模式与边界

- **角色集合写死**：仅 Planner / Executor / Summarizer，缺 Verifier / Critic / Tool-user，「必须验证或调用外部世界」的任务无法被正确表达。
- **成本可观测性依赖**：部署时的延迟、限流、缓存命中难以在训练中刻画，α 调好的策略在生产可能偏离最优。
- **记忆规模与静态评估**：冷启动弱、分布漂移污染能力画像；traces 累积使检索成本增长，需淘汰／衰减机制；且 14 任务 6 领域仍是基准数据集，缺带副作用的端到端验证。

## 实践启示

1. **动作空间从「选模型」升级为「选角色 × 选模型」**：多模型系统的收益常来自分工结构，不只是换更强 backbone。
2. **显式记录 accuracy 与 cost 并让二者进入决策**：每次调用的效果与费用都应成为可查询的边。
3. **用「角色」作跨任务对齐键**：中间层抽象应是角色/能力而非模型名或 prompt 模板；且能一步做完就别拆。
4. **先定角色集合，再定推理模式**：优先补齐 Verifier / Tool-user，否则能力上限在设计阶段就被锁死；求轻量用 Inductive，求上限用 Transductive 并配套记忆淘汰。

## 相关实体

- 相关概念：[[concepts/harness-engineering-framework|Harness Engineering]]、[[concepts/agent-role-specialization|Agent Role Specialization]]、[[concepts/agent-orchestration-patterns|Agent Orchestration Patterns]]、[[concepts/agent-memory-architecture|Agent Memory Architecture]]
- 训练与路由：[[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法]]、[[entities/layerrecall-memory-router-zju-hku-arxiv-2026|LayerRecall Memory Router]]、[[entities/cursor-router-production-model-routing-2026|Cursor Router 模型路由]]；相关：Agent 架构

→ [[raw/articles/给多智能体llm装上图记忆工作流路由器搞定调用协作规划|原文存档]]
