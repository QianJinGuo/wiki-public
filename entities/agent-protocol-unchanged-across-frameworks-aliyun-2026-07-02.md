---
title: "Agent Protocol 不变层：跨框架的 6 个稳定 Runtime 对象"
created: 2026-07-02
updated: 2026-11-09
type: entity
tags: [agent, protocol, runtime, framework, architecture, langgraph, a2a, ag-ui]
sources: [raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Agent Protocol 不变层：跨框架的 6 个稳定 Runtime 对象

> Agent 框架层出不穷——LangGraph、OpenAI Assistants、A2A、AG-UI、Deep Agents——名字越来越多，API 越来越像一套套独立世界观。但框架名词在变，底层问题始终围绕任务、上下文、步骤、事件、状态和产物展开。将这些名词往下拆，会发现它们都在回答同一个底层问题：**Agent Runtime 对外暴露的一组稳定对象、生命周期操作和状态迁移是什么？** ^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

## 核心框架：Agent Runtime Protocol 6 对象

**Agent Protocol ≠ 某一个具体标准**（不等于 A2A、AG-UI、LangChain Agent Protocol 或任意单一规范）。它指的是 Agent Runtime 对外暴露的一组稳定对象、生命周期操作和状态迁移。具体协议标准和框架 API 是证据，不是主线。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

跨框架反复出现的 6 个稳定对象：

| 对象 | 人话解释 | 回答的问题 |
|------|---------|-----------|
| **Thread/Session** | 一段长期上下文 | 这是谁的哪段任务？ |
| **Run/Task** | 一次具体执行 | 这次具体跑了什么？ |
| **Step** | 执行中的一个可观测步骤 | 哪一步调用了模型、工具或子 Agent？ |
| **Event** | 执行过程中的进展变化 | 现在发生了什么？ |
| **Artifact** | Agent 产出的正式结果 | 结果在哪里，由哪次执行产生？ |
| **Checkpoint** | 可以恢复的执行快照 | 失败或中断后从哪里继续？ |

这 6 个对象是理解 Agent Runtime Protocol 的入口。围绕它们，生产级 Protocol 还需表达 `stream/interrupt/resume/cancel/retry` 等生命周期操作。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

## 三层概念：标准、对象、Runtime 能力

讨论 Agent Protocol 时最容易混淆的三层：^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

1. **具体协议标准**（A2A、AG-UI、LangChain Agent Protocol、ACP）——不同系统如何通信
2. **通用协议对象**（Thread、Run、Step、Event、Artifact、Checkpoint）——外部世界如何稳定理解一次 Agent 任务
3. **Runtime 实现能力**（状态持久化、中断恢复、可恢复流、权限控制、可观测性）——Runtime 内部如何兑现这些对象和状态机

文章重点讨论第二层：通用协议对象。具体协议标准和框架实现只作为证据。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

## Runtime 与 Protocol 的关系

**Protocol 是 Runtime 的外部边界，Runtime 是 Protocol 的内部实现。** ^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

Agent Runtime 暴露给外部世界的契约回答的是：^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

- **如何启动一次任务**：创建 Thread、Task、Run，或发送一条 Message
- **如何携带上下文**：历史消息、文件、结构化数据、参与者、能力声明
- **如何观察进展**：状态变更、流式事件、Artifact 增量、Trace
- **如何中断和恢复**：需要输入、需要授权、取消、重试、继续执行
- **如何拿到结果**：最终消息、Artifact、结构化输出、错误信息

## 跨框架映射

| 框架/标准 | Thread/Session | Run/Task | Step | Event | Artifact | Checkpoint |
|-----------|:---:|:---:|:---:|:---:|:---:|:---:|
| LangGraph | — | — | — | — | — | ✅ Checkpoint |
| OpenAI/Run | Thread | Run | Step | — | — | — |
| A2A | — | Task | — | — | Artifact | — |
| AG-UI | Session | — | — | Event | Artifact | — |
| Deep Agents | — | Todo | Subagent | — | VFS | — |

这组映射说明：**没有哪个框架完整实现全部 6 个对象。** 每个框架选择聚焦的对象子集，决定了它的适用场景和局限性。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

## 核心洞见

1. **Agent Runtime 的核心不是模型调用，而是任务生命周期管理**——Thread/Run/Step/Event/Artifact/Checkpoint 这 6 个对象围绕的都是任务如何被创建、执行、观测、中断、恢复和完成，而非模型如何推理。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

2. **Thread/Run/Step/Event/Artifact/Checkpoint 会成为跨框架的稳定对象**——框架会更迭，API 会改名，但这些对象对应的底层问题不会消失。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

3. **执行模型不会统一**——Runtime Loop 承载方式和编排协议会长期分层演进。不同框架在"一次性运行 vs 长任务恢复"、"同步 vs 流式"、"单Agent vs Multi-Agent"等维度上的选择是设计偏好，不是正确性差异。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

4. **真正区分玩具 Agent 和生产 Agent 的，是状态持久化、中断恢复、可观测性和可评测性**——这是协议对象之外的 Runtime 实现能力层。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

5. **值得看的不是某个框架 API，而是协议边界和 Runtime 抽象**——框架会更迭，但协议边界稳定的时间更长。当你理解这 6 个对象后，看新框架时就能快速判断：它只是换了一套 API 名字，还是解决了一个真实的 Runtime 问题？^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md]

## 深度分析

### 为什么会沉淀出一个不变层

框架层注定不稳定，因为它把两类变化速度完全不同的东西绑在一起：模型与产品形态的月级迭代，和"任务如何被启动、观测、中断、恢复"这类长期不变的结构问题。文章的信号是：十来个标准与框架——LangChain Agent Protocol、A2A、AITP、ACP、AG-UI、OpenAI Assistants、LangGraph Server API、Deep Agents、OpenTelemetry GenAI——没有一个完全收敛，却共同把中心从单次 chat completion 移向"长生命周期、可观测、可评测、可恢复、可协作的任务对象"。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:99-164] 这是标准竞争的典型路径：先稳定的不是某个标准，而是谁也绕不开的对象词汇表。这 6 个对象不变，因为它们是任务生命周期的逻辑必经点，不是某家的 API 设计偏好。

### 分层经济学：哪些层先标准化，哪些层永不统一

文章实际给出一张跨层标准化的"难度光谱"。工具层最先标准化：JSON Schema 已是事实标准，MCP 把工具发现、定义、调用、资源读取从框架内部抽出来——因为它边界清晰、输入输出结构化、与 loop 承载方式解耦。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:606-678] 事件流层次之：SSE 因 HTTP 原生、自带重连被判定为持久选择，可恢复流是跨网络部署的刚需。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:705-733] 而执行模型、编排协议、多 Agent 协作被明确判定不会统一——问题空间太大。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:993-996] 内在逻辑：**离模型越近的层越不稳定，离"任务记账"越近的层越稳定**，协议化因此自外向内推进。

### 不变层为什么必须"低约束"：生态位与互操作

不变层能成立，前提是只约束语义、不约束实现："最好的协议是低约束的，最好的 Runtime 是高内聚的"——协议只定义外部可依赖的边界，Runtime 在边界内保持实现自由。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:933-953] 低约束正是互操作的杠杆：所有 Runtime 按统一语义表达 `Thread -> Run -> Step -> Artifact` 后，控制台、评测、审计系统不再需要理解各框架内部状态结构。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:398] 生态已为碎片化付出代价：Trace 格式各自为战、框架绑定过深、混用多框架即失去统一视图，文章据此判断 Agent Tracing 的 OpenTelemetry 式标准化是迟早的事。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:853-857]

### 偏离不变层的代价：级联缺口与向后兼容负担

映射表的另一面是缺失：缺口会级联。状态持久化是分水岭：没有持久化就无法崩溃后恢复、无法支持真正的 Human-in-the-Loop、无法分叉重放。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:466] 中断/恢复是它的直接延伸：没有快照只能同步 ask-and-wait，Human-in-the-Loop 的基础设施不是 ask-user API，而是"状态快照 + 中断载荷 + 恢复指令 + 权限上下文"的组合；错误恢复同理，没有 Checkpoint 回滚，10 步任务第 8 步失败就得重跑前 7 步。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:518-526] 时间维度上还有最贵的隐藏成本：Schema 演进——今天的 Checkpoint 三个月后还能否恢复？需要状态版本号、迁移函数、兼容窗口，而"真正难的不是保存，而是恢复"。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:428-439] Checkpoint 链式版本控制因此只被标记为"可能持久"：概念承诺（类 Git）比实现承诺更耐久——与"协议对象比框架 API 更稳定"是同一逻辑。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:962-972]

## 实践启示

1. **用"协议四问"审视新框架，而不是学它的 API**：它只是换 API 名字还是解决真实 Runtime 问题？强化的是执行模型、状态管理、工具协议还是流式事件？长期趋势还是阶段性流行？再查它能否稳定表达 6 对象、有无真正持久化和可恢复事件流。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:42-45]

2. **投入可迁移层，谨慎投入易变层**：协议对象模型、工具协议抽象、状态管理抽象是跨框架通用的设计模式，值得重点投入；特定框架执行模型深入 1-2 个即可，重 why 不重 API；多 Agent 模式碎片化严重，等标准出现再深入不迟。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:1000-1010]

3. **选型先查 6 对象覆盖表**：缺 Checkpoint 就过不了崩溃恢复、Human-in-the-Loop、分叉重放门槛；库形态缺可恢复流尚可接受，服务化后断线恢复是刚需；缺口能否后补，参照持久性判断——状态持久化、Error-as-Data、SSE 均为"持久"。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:62-68]

4. **持久化先分层再选型，Schema 演进是第一天的问题**：Conversation、Run State、Checkpoint、Artifact、Semantic Memory 是五层不同状态，"支持 Memory"的框架往往只支持一层——先说清保存哪层；自建 Checkpoint 就要同时设计版本号与迁移窗口，选托管则接受控制权让渡。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:372-388]

5. **并发语义写进协议而不是锁**：Thread 不是锁、Run 才是执行边界；同一 Thread 是否允许多 Run 并行（串行队列/拒绝/取消覆盖/分叉/乐观并发）必须作为 Run 创建语义显式暴露，否则消息顺序、Artifact 归属、事件去重都会变成不可复现的隐式行为。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:402-426]

6. **把错误当数据，把控制面当安全边界**：Error-as-Data 让 LLM 读错误信息自主决定重试、换工具或上报，仅系统级故障才 raise；工具一旦有真实副作用，权限、Guardrail、审批、预算、取消就从外围变成安全边界——这与 [[concepts/model-context-protocol-mcp|Model Context Protocol (MCP)]]（工具层先标准化）和 [[entities/agent-runtime-7-responsibilities-secondcurve-2026|Agent Runtime 7 大职责]] 划出的边界一致。^[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02.md:650-663]

## 与其他实体的关系

- [[entities/agent-runtime-7-responsibilities-secondcurve-2026|Agent Runtime 7 大职责]] — 从 Runtime 职责视角（状态持久化、工具编排、可观测性等）出发，与本文的 Protocol 视角互补
- [[entities/from-agent-protocol-to-harness-skill|From Agent Protocol to Harness Skill]] — 从 MCP/A2A 协议到 Harness Skill 的演进路径，与本文的 Runtime Protocol 抽象视角不同
- [[entities/agent-executor-googles-distributed-agent-runtime-da1bb4|Google Agent Executor]] — 分布式 Runtime 实现案例
- [[entities/agent-harness-architecture|Agent Harness 架构]] — Harness 作为 Runtime 的上层封装
- [[entities/agent-architecture-harness-new-backend|Agent 架构关键变化：Harness 成为新后端]] — Harness 作为 Runtime 演进方向
- [[entities/agentic-ai-system-architecture-harness-skill-mcp|MCP · Skill · Agent · LLM · Harness]] — 高层架构关系图

→ [[raw/articles/agent-protocol-unchanged-across-frameworks-aliyun-2026-07-02|原文存档]]
