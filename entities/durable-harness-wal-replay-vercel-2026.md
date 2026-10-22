---
title: "durable-harness: 把任意 Agent Harness 变成可崩溃恢复的持久执行体"
created: 2026-10-21
updated: 2026-10-22
type: entity
tags: [agent, harness, durability, vercel, workflow, sandbox, coding-agent]
sources: [raw/articles/durable-harness-wal-replay-vercel-2026]
confidence: 0.75
provenance_state: extracted
---

# durable-harness: 把任意 Agent Harness 变成可崩溃恢复的持久执行体

Vercel 2026-10 发布的 durable-harness 针对的是一个很具体、也很被低估的失败面：**coding harness 正在从本地进程搬到云端，而云端默认不记录"哪一步已经完成"**。本地跑 harness 时，工作区、会话历史与进程在同一块盘上，中断后一句 prompt 就能续上；搬到云端后，用 VM + 持久存储可以复刻这套，但 VM 不会自动记录哪些 model call / tool call 已经完成，一次 turn 失败或网络抖动就是实打实的返工。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

## 机制：intent/outcome 双记录的 turn 日志 + 队列兜底

实现是一个薄适配层：它在每一步 turn 写入 durable log——**在 model call 或 tool call 执行前记录 intent，结束后记录 outcome**；turn 本身再由一条队列消息兜底，若承载它的函数死掉，队列把该 turn 交给新函数，新函数从日志读出"哪些已完成、哪些正在跑、哪些从未开始"。已完成调用不再执行（结果直接读日志），因此不会为已完成的 model call 二次计费；模型也从日志里记得自己此前调用过什么，而不是靠重放整段上下文猜。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

工具侧的语义是这套设计的关键区分：被标记为 `replay: 'safe'` 的中断工具会重跑；其他工具则回到模型面前以"可能已执行"的状态呈现，由模型决定是否核实。这是对"至少一次"执行语义的显式承认——**幂等性无法自动判定，只能由工具作者声明，剩下的用不确定性交还给模型**。与之配套，Claude Code 的 transcript、Codex 的 rollout、OpenCode 的 event store 这类 harness 自有会话文件也会被镜像进日志，使新函数能接续而不丢失会话上下文。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

## World 抽象：日志、兜底、流三合一

作者把持久执行需要的三件东西归纳为**一条日志、一种在进程死亡后恢复 turn 的方式、一条给旁观者的流**，并统一在一个 `World` 接口后面；durable-harness 只面向该接口写入，因此同一份 agent 代码可以在 Vercel World 与 Postgres World 之间迁移——后者意味着可以自托管、用自家 Postgres 做持久后端。这个抽象层次的选择值得抄：**把"持久化后端"从 harness 里剥出来，harness 本身只负责产生 step 语义**。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

另一个被明确的边界是 sandbox 与 function 的分工：并非每个 agent 任务都需要沙箱，数据库查询与 API 调用在函数里就能安全执行，且**凭证因此不必进入沙箱**；需要文件系统时再升级到 `sandbox()`——一个能活得比任何单个 sandbox 与函数都久的持久工作区。中间形态还有一个 bash 兼容的受限 shell（不触达函数文件系统与网络），适合短任务与小工作区。这实际上是给 harness 提供了一条"按任务风险逐级提升执行环境"的阶梯，而非一律上沙箱。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

## 深度分析

### 崩溃窗口：两条记录把状态空间从二值变三值

"先做事、再记账"的系统都有致命窗口：副作用已落在外 world，账还没落。单条 done 记录分不清"从未开始"与"跑了但回执丢了"——前者可安全重跑，后者重跑就是重复副作用。intent/outcome 把每个 step 的状态扩成三值：无 intent = 未开始，有 intent 无 outcome = 在飞（窗口本身），有 outcome = 已完成可复用。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

但 intent 只把窗口变窄：写 outcome 本身也会失败。

### 幂等性无法自动推断：判定权给作者，核实权给模型

跨进程 exactly-once 在外部系统不配合时不可实现，这是老结论。durable-harness 把它变成显式声明：`replay: 'safe'` 的工具崩溃后自动重跑，其余（退款、发评论）以"可能已执行"回到模型面前，由模型决定是否核实——即对 at-least-once 的公开承认。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

三个后果一起接：声明即承诺，误标是只在崩溃路径上兑现的 bug 类别，得进 review 与 CI；不可解的情形要命名——外部系统不吃幂等键又无可靠查询时只能靠模型核验，核验还可能撞上最终一致性，这类工具该加人工确认；不确定性必须写进上下文，否则模型会把"工具失败"当结论直接重试。

### World 抽象：依赖倒置，但抽象有漏

持久执行的需求被收成三件：一条日志、一种进程死后接管 turn 的方式、一条给旁观者的流，藏在 `World` 接口后面，harness 只对它写。后端由变量注入，同一份代码于是在 Vercel World 与 Postgres World 间迁移。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

这层抽象比多数方案靠上：不管 harness 怎么写循环，只管每步留下什么痕迹、死后谁接管。挂 EFS/S3 存 session 目录（[[entities/amazon-bedrock-agentcore-data-persistence-file-system-session-storage-efs-s3|AgentCore 会话持久化]]）持久化的是**数据**，World 持久化的是**决策依据**：数据可重建，决策记录丢一次就是返工。

漏处在语义而非 API：会话内顺序、turn 认领互斥、流的可重放偏移，三条在各后端强度不同，迁移成本全压在这层兼容层。

### 成本模型：贵在 p50，便宜在 p99

每 step 两次写加一次流推送，30 次工具调用就是 60 次写；远端服务每次往返，不批量就涨几十毫秒。真实账看分布：p50 略慢，p99 大幅变好，最坏情况从"整 turn 重跑（含已完成 model call 的 token）"变成"从日志续跑"——小 turn 净亏，大 turn 净赚，赚的是尾部。

存储更易低估：会话文件与文件树也镜像进日志，长会话就成了新的膨胀源。队列兜底还改了失败语义：旧函数活着但变慢时重投，同一 turn 会被并发执行，需租约或 fencing token 才挡得住——原文只说"函数死了队列接过去"，没说"函数没死但被判死"。因此短交互会话不该上，长程编码与无人值守任务（[[entities/alibaba-devix-harness-ops-agent-7x24|Devix 的 7×24 实践]]）必须上。

### 与持久执行引擎的分界：不是 workflow-as-code 换皮

Temporal 式引擎要"代码 + 确定性重放"：崩溃后从头重跑函数，靠确定性跳过已完成 activity——它拥有程序，能对结构提要求。durable-harness 不改 harness 循环、也不要求循环确定性，只在外套适配器、把会话文件镜像进日志，再从日志续一个 turn。**确定性重放要求你拥有循环，日志续跑只要求你能看见每一步。**^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

复用的全是老件：WAL、至少一次投递、幂等键、队列兜底。真新的两件：把未改造的第三方 harness 变成可持久对象（四种会话格式抹平到一条日志，属打包贡献）；把恢复决策下放给模型（经典引擎遇 in-flight 副作用会失败工作流或要补偿事务，这里把"可能已执行"交给模型判断）。

与 checkpoint/resume 系（[[entities/hermes-agent-goal-runtime-architecture-state-persistence-judge-closed-loop|goal runtime]]、[[entities/openrath-session-centered-agent-runtime-tsinghua-2026|会话中心 runtime]]）的差别在责任归属：快照语义由 agent 自定义，WAL 语义由运行时定义，因此能自动判定可否重跑。判断保留：厂商博客无故障注入数据；"换后端只改变量"取决于兼容层的语义覆盖。

## 实践启示

1. **先给工具分三类，再谈持久化。** 只读类标 `replay: 'safe'` 允许重跑；外部副作用类（退款、发评论）声明不可重跑；没想清楚的一律落到"不确定"。这张表要进 PR review 清单：误标的代价只在崩溃路径上兑现。

2. **每个副作用带 dedupe key。** 用 `session_id + turn_id + tool_call_id` 作幂等键写进入参与下游请求头；下游不支持就降级为"先查后写"，并在 tool description 里明写"可能已执行，重试前必须核实"。

3. **记账到 step，会话文件降级为镜像。** model call 与 tool call 各自记账：一次 turn 里最贵的是 model call 的 token。真相源交给日志，transcript / rollout / event store 只做镜像，换后端与重建沙箱才不用改 harness。

4. **选 World 按语义验收，不按品牌。** 必测三条：会话内顺序、并发重投下 turn 认领是否互斥、流能否从偏移重放。自托管优先 Postgres World；给不出这三条，别指望只换环境变量。

5. **用故障注入定价，顺手定保留策略。** 复现"turn 中途 kill 函数"与"浏览器断连"，量三个数：重复执行的 step 数、重复副作用次数、恢复墙钟时间。短会话跑完多半是"不值得"；长会话结论相反，则把日志保留期与压缩一并定下来。

## 在既有 harness 脉络中的位置

如果说 [[concepts/harness-engineering|Harness Engineering]] 关心的是"如何给模型搭一套可靠的外骨骼"，那么这篇补的是外骨骼的**故障模型**：[[concepts/harness-long-running-task|长任务 harness]] 讨论任务能跑多久，durable-harness 讨论"跑到一半断电后还剩什么"。它与 [[entities/agent-long-task-intentional-resume-taobao-live-2026-09|长任务的有意续跑]] 属于同一问题的两条路径——后者靠 agent 自己维护状态与意图，前者靠运行时把每步落成日志；两者可以叠加：日志保证不丢步，意图记录保证恢复后仍知道为什么做。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

对 [[concepts/coding-harness-engineering|Coding Harness Engineering]] 的实践含义有三条：(1) 把 turn 视为可重放的事务单元，而不是一次函数调用；(2) 工具必须显式声明重放安全性，默认按"不确定"处理；(3) 把会话文件的真相源交给日志，harness 本地格式只作镜像。它与 [[concepts/harness-loop-architecture|Harness 循环架构]] 的组合是：循环负责推进，日志负责记账，World 负责在崩溃后把两者对上。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

需要保留的判断是：这是厂商博客，架构与机制描述清楚但未给公开 benchmark 或故障注入数据，因此"durable"的量化收益（返工率下降多少、token 浪费减少多少）仍需自行验证；适配器目前绑定 Vercel 的实现路径（Workflow SDK / eve / fx），跨平台迁移的代价主要落在 World 后端的兼容层上。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

→ [[raw/articles/durable-harness-wal-replay-vercel-2026|原文存档]]
