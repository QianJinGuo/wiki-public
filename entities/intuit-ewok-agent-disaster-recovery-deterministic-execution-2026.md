---
title: "Intuit EWOK Agent：模型决策/确定性执行分离的灾害恢复 Agent 架构（typed skills + 有界循环）"
created: 2026-09-05
updated: 2026-09-28
type: entity
tags: [agent-architecture, disaster-recovery, decision-execution-separation, typed-skills, mcp, guardrails, bounded-agentic-loop, deterministic-execution, aws, bedrock]
sources: [raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Intuit EWOK Agent：模型决策/确定性执行分离的灾害恢复 Agent 架构（typed skills + 有界循环）

> Intuit 在跨多 AWS Region 数千微服务的规模上，将灾害恢复（DR）决策从「on-call 工程师的部落知识」转移到 AI Agent：构建 EWOK Agent（Amazon Bedrock）叠加在已有的确定性执行系统 EWOK（Ecosystem Wide Orchestrator Kit）之上。核心设计立场是 **"The model decides what to do, and the EWOK Agent deterministically executes how"**——模型决策与确定性执行分离。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

## 问题：EWOK 解决了执行，没解决决策

Intuit 已有一个中央灾害恢复系统 **EWOK**，用 YAML 声明式定义恢复意图，标准化 compute/数据库/网络/缓存/异步负载的故障切换（failover）执行，把恢复时间从数小时降到约 20 分钟。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md] 但 EWOK 只解决**执行**，不解决**决策**：选择哪个恢复工作流适用、确认资产是否就绪，仍依赖 on-call 工程师的部落知识（tribal knowledge）。EWOK Agent 正是为这个决策缺口而构建。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

**关键可迁移洞见（文章原文）**：下文所述模式（typed skills + 薄 Amazon Bedrock 层 + 确定性执行器之上的有界 agentic 循环）并非 EWOK 特有，可应用于任何暴露了"已认证、可审计 API"的系统。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

## 四层架构

EWOK Agent 自顶向下四层，从工程师的自然语言请求到确定性恢复动作：^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

- **Consumer 层（顶）**：Intuit Engineering Portal + IDE 集成，经 [[concepts/coding-agent-architecture|MCP]] 连接，是工程师提交自然语言请求的两个入口。
- **Agent 层**：运行于 Amazon Bedrock，将基础模型选择 + 有界推理循环 与 每次调用都应用的 Amazon Bedrock Guardrails 配对；负责模型选择、guardrails、skill 分发。
- **Skill 层（右）**：持有**类型化、带版本的 skills**（typed, versioned skills），每个定义为 YAML schema + prompt body，编译成模型可选的工具规格。
- **Execution 层（底）**：EWOK API 层，确定性地执行资产解析、恢复工作流查找、就绪检查、策略门（policy gates）、基于 execution-ID 的追踪、变更记录，以及故障切换本身——通过 compute/database/cache/traffic 各自的工作负载专用 agent 完成。

状态沿同一条路径向上流回（execution → agent → consumer）。

## 核心设计原则：决策/执行分离

文章反复强调一条贯穿全部设计选择的原则：**模型决定做什么，EWOK Agent 确定性执行怎么做**（The model decides what to do, the EWOK Agent deterministically executes how）。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

- **决策/执行分离**（decision/execution separation）：模型负责在类型化恢复 skills 中做选择；确定性 EWOK 执行器负责执行选中的工作流。这把模型错误的爆炸半径限定住。
- **类型化、带版本的 skills**：每个 skill = YAML schema + prompt body，编译成工具规格，把部落知识（runbook 查找、哪个工作流适用、资产是否就绪）变成**版本控制、可评审、可测试**的产物。
- **有界推理循环**（bounded reasoning loop）：Agent 在确定性执行器之上运行有界循环，每次调用应用 guardrails，而非不受约束的自主探索。
- **逐任务模型选择**：把基础模型选择与 skill 分发决策配对。

## 工程师视角的故障切换体验

工程师对着 Agent 说"You failover payments-gateway in production"，EWOK Agent 依次：解析资产并发现可用恢复工作流 → 选择合适工作流（或让工程师在多选适用时挑选）→ 验证就绪并检查策略门（如激活中的变更冻结窗口）→ 通过 EWOK 系统触发执行并返回 execution ID + change record → 分阶段监控上报直至完成。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

这些任务过去是 runbook 查找 + 控制台访问的序列，由工程师协调 API 调用；现在改为**监督对话**（supervise conversations）。工程师保留在判断与批准环节，但不再需要当编排器。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

## 深度分析

### 决策/执行分离：控制面与数据面的分野

EWOK 的分层并非新发明——它是分布式系统里**控制面/数据面分离**（control-plane/data-plane split）这一经典架构决策在 Agent 语境下的重现。SDN 把路由决策集中到控制器、把转发留给交换机；Kubernetes 把调度决策交给 control loop、把容器操作交给 kubelet。共同逻辑：**易变、需全局视野的判断收敛到一处，高频、必须可靠的执行下沉到确定性组件**。EWOK Agent 中模型（Agent 层）就是控制面，EWOK 确定性执行器就是数据面。

而 DR 对这条分界线的要求比一般系统苛刻得多：故障切换是低频、高代价、必须可审计的操作——错误执行一次错误的工作流，代价是生产事故叠加事故。因此 DR 不只要分离，还要求控制面的每次决策都留下**版本化产物**（选中的 skill、策略门检查记录、change record），数据面执行可凭 execution-ID 精确追踪与回滚。对照 [[concepts/harness-loop-architecture|Harness 循环架构]]，EWOK 把"模型在环上哪一层介入"压缩到了最小——模型只碰选择，不碰执行。这正呼应 [[entities/martin-fowler-ai-rd-harness-nondeterminism|Martin Fowler 论 AI 研发中的非确定性]]：非确定性组件必须被确定性边界包裹，才可能进入生产关键路径。

### Typed skills：契约层及其漂移失效模式

Typed, versioned skills 是模型与执行器之间的**接口契约**：YAML schema 定义参数类型与约束，prompt body 定义语义，编译产物是模型可选的工具规格。价值在于把部落知识变成可 diff、可评审、可测试的工件——恢复逻辑从此走代码审查流程而非口口相传。

契约层独有的失效模式也由此产生：

- **schema 漂移**：底层 EWOK API 演进（新增必填参数、改枚举值），skill 的 YAML schema 未同步——模型在类型化约束内"正确"地填了过时字段。缓解：schema 与 API 版本绑定，CI 里跑契约测试。
- **语义漂移**：schema 不变但 prompt body 描述的适用条件与真实工作流行为脱节——模型选了"名义上适用"的 skill。缓解：把"哪个工作流适用"写成可执行的就绪检查，而非纯自然语言。
- **版本错配**：多版本并存时模型选中旧版本。缓解：版本选择本身应是显式的、带策略门的决策。

这组失效模式与 [[concepts/skill-engineering-principles|Skill 工程原则]] 的"skill 即契约"立场互为印证：契约的价值不在写出它，而在**演进它时不出错**。

### 有界循环：爆炸半径的最后一道闸

即便决策正确、契约一致，仍须假设模型会偶发出错。有界推理循环 + 每次调用都过 guardrails，是把错误后果限制在可承受范围内的最后一道闸。爆炸半径被三层叠套的边界约束——**决策边界**（只能选类型化 skills）、**循环边界**（步数有限）、**执行边界**（策略门 + execution-ID 追踪）——任何一层失守，下一层兜底。

这比"给模型更多自主权、出问题再回滚"更符合 DR 语境：灾害恢复本身就是在事故中操作，此时最不可承受的是 Agent 自身成为新的故障源。对照 [[entities/agent-vs-workflow-control-continuum-framework|Agent vs Workflow 控制连续谱]]，EWOK Agent 的位置刻意靠近 workflow 一端——自主性只花在"选对工作流"这一个决策点上。

### 与其他确定性执行进路的对照

- [[concepts/harness-engineering-7-layers-framework|Harness 工程七层框架]] 从通用 Agent 出发逐层加确定性结构；EWOK 反其道而行——从既有确定性系统出发，反向叠加一层薄的模型决策。前者"自顶向下加约束"，后者"自底向上加智能"。
- [[concepts/loop-engineering-methodology|Loop 工程方法论]] 关注循环本身的节律与反馈；EWOK 的有界循环是它在"逐次过 guardrails"这一 DR 级约束下的特例。
- 与让 Agent 生成并执行命令的进路相比，EWOK 把"执行"完全外包给经过实战验证的编排系统——模型永远不写恢复逻辑，只选恢复逻辑。这是大规模生产 DR 中更保守也更正确的取舍。

## 实践启示

1. **先有确定性执行器，再叠加模型**。EWOK Agent 的成功前提是 EWOK 已把恢复时间从数小时压到约 20 分钟。把模型叠在你已验证的 API 之上，而不是让它直接操作基础设施。
2. **把领域知识做成类型化契约**。Runbook、适用条件、就绪检查都应沉淀为带版本的 schema + prompt 产物，进入代码审查与 CI 流程。
3. **自主性是稀缺资源，花在最难的单一决策点上**。EWOK 把模型决策压缩到"选哪个工作流"这一个点，其余全部类型化。
4. **为契约漂移设计防护**。契约与底层 API 版本绑定，schema 变更走契约测试，语义适用条件尽量转为可执行检查。
5. **爆炸半径用多层边界套住**。决策、循环、执行三层边界各自独立成立，任一层失效不放大到下一层。
6. **决策必须留下可审计的产物**。每次模型决策都应产生可追溯记录——DR 语境下"为什么这么做"与"做了什么"同等重要。

## 与 wiki 已有知识的关联

- 有界循环 + guardrails 逐次应用 → 对照 [[concepts/harness-loop-architecture|Harness 循环架构]]、[[entities/amazon-bedrock-guardrails-code-generation-six-patterns|Bedrock Guardrails 六模式]]
- typed skills 编译成工具规格 → [[concepts/skill-engineering-principles|Skill 工程原则]]
- 心智"模型决策 + 确定性执行" → [[entities/agent-architecture-harness-new-backend|Harness 成为 Agent 新后端]]

→ [[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026|原文存档]]