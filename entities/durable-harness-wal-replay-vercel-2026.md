---
title: "durable-harness: 把任意 Agent Harness 变成可崩溃恢复的持久执行体"
created: 2026-10-21
updated: 2026-10-21
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

## 在既有 harness 脉络中的位置

如果说 [[concepts/harness-engineering|Harness Engineering]] 关心的是"如何给模型搭一套可靠的外骨骼"，那么这篇补的是外骨骼的**故障模型**：[[concepts/harness-long-running-task|长任务 harness]] 讨论任务能跑多久，durable-harness 讨论"跑到一半断电后还剩什么"。它与 [[entities/agent-long-task-intentional-resume-taobao-live-2026-09|长任务的有意续跑]] 属于同一问题的两条路径——后者靠 agent 自己维护状态与意图，前者靠运行时把每步落成日志；两者可以叠加：日志保证不丢步，意图记录保证恢复后仍知道为什么做。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

对 [[concepts/coding-harness-engineering|Coding Harness Engineering]] 的实践含义有三条：(1) 把 turn 视为可重放的事务单元，而不是一次函数调用；(2) 工具必须显式声明重放安全性，默认按"不确定"处理；(3) 把会话文件的真相源交给日志，harness 本地格式只作镜像。它与 [[concepts/harness-loop-architecture|Harness 循环架构]] 的组合是：循环负责推进，日志负责记账，World 负责在崩溃后把两者对上。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

需要保留的判断是：这是厂商博客，架构与机制描述清楚但未给公开 benchmark 或故障注入数据，因此"durable"的量化收益（返工率下降多少、token 浪费减少多少）仍需自行验证；适配器目前绑定 Vercel 的实现路径（Workflow SDK / eve / fx），跨平台迁移的代价主要落在 World 后端的兼容层上。^[raw/articles/durable-harness-wal-replay-vercel-2026.md]

→ [[raw/articles/durable-harness-wal-replay-vercel-2026|原文存档]]
