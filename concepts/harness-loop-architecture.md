---
title: "harness 主循环架构"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, harness, agent-loop, architecture, react, claude-code]
sources: [entities/agent-harness-architecture, entities/claude-code-architecture, entities/wow-harness-v3-governance-protocol]
---

## 定义

harness 主循环是 agent 系统的心脏：read user input → 装配 context → 调 LLM → 解析 tool calls → 执行 → 反馈结果 → 决定继续/停止。架构差异决定 agent 行为：单循环 vs 嵌套循环、同步 vs 异步、是否带 governance gate。

## 核心范式

- **ReAct 单循环**：thought-action-observation 串行，简单但难做长程
- **嵌套循环**：主循环管全局，subagent 子循环管局部任务
- **governance gate**：每轮 LLM 调用前检查 budget / safety / context size
- **停止条件**：max iterations、达成目标、用户中断、错误升级

## 背景与提出

harness 主循环架构的概念来自 agent 系统设计的核心工程问题：agent 如何做出决策、调用工具、评估结果、决定下一步？Agent Loop 是所有 agent 系统的共同心脏——无论 agent 有多复杂，它的运转都依赖于一个不断循环的「感知→决策→执行→评估」模式。这个概念在 2026 年随着 Claude Code、OpenClaw 等框架的开源变得可观测、可分析、可对比。 ^[entities/agent-harness-architecture]

主循环架构的设计差异直接决定了 agent 的行为特征：ReAct 单循环适合短程任务，但难以处理需要多步规划的长程任务；嵌套循环（主循环 + subagent 子循环）通过分层解决了这个问题，但引入了新的复杂性——父子循环之间的状态同步和错误传递变得难以调试。Claude Code 的 7 种 subagent 执行模式正是为了处理这种分层循环的复杂性而设计的。 ^[entities/claude-code-core-internals]

## 范式细节

ReAct 单循环（Thought-Action-Observation 串行）是 agent loop 的基础模式。模型输出 thought（推理），再输出 action（工具调用），然后观察结果（observation），再进入下一轮推理。优点是简单、可追踪；缺点是每轮都必须在 context 中携带完整历史，在长程任务中 context 消耗极快。ReAct 的本质是「线性推理」，不适合需要并行探索的复杂任务。 ^[entities/agent-harness-architecture]

嵌套循环是 Claude Code 的主架构：主循环（query.ts）负责整体任务协调，subagent 子循环（AgentTool）负责局部任务执行。7 种执行模式的本质是对「父子状态共享程度」的不同选择：同步前台模式（状态完全共享）、异步后台模式（状态通过 TaskOutput 传递）、Worktree 隔离模式（文件系统级别隔离）、Fork 模式（字节级 system prompt 副本）、Teammate 模式（独立 tmux session + 双向通信）。模式选择决定了子任务的隔离程度和通信开销。 ^[entities/claude-code-core-internals]

governance gate 是每次 LLM 调用前的安全检查点。Claude Code 的 governance 包括四维预算检查：token 预算（`output_config.task_budget`）、成本预算（`maxBudgetUsd`）、工具结果预算（每个工具的 `maxResultSizeChars`）、轮次预算（`maxTurns`）。任何一维超限，agent 循环立即停止并上报。这个设计的精妙之处在于：四维独立，不相互干扰——token 超了不代表 cost 超了，可以选择继续执行而不触发停止。 ^[entities/claude-code-core-internals]

停止条件的设计反映了对 agent 行为的控制粒度：max iterations（防止无限循环）、达成目标（通过 verifier 判断）、用户中断（主动信号）、错误升级（连续失败 N 次后触发人工介入）。Claude Code 的权限拒绝渐进升级机制（连续 3 次或累计 20 次后自动升级到询问用户）是停止条件的一种精细化实现——防止 agent 在权限问题上死循环，同时不直接中断任务。 ^[entities/claude-code-core-internals]

## 局限与反对声音

第一个局限是「循环终止的判定是开放问题」：对于复杂任务，「当前循环是否应该继续」没有客观标准。Max iterations 是最保守的硬限制，但会导致「任务未完成但被迫停止」。Verifier 判断依赖 verifier 本身的可靠性。人类中断依赖人类在场的及时性。没有任何一种停止条件是完美的——设计者必须在「可能无限循环」和「可能过早停止」之间做权衡。

第二个局限是「父子循环的状态同步」：当 subagent 在后台运行时（异步模式），父 Agent 需要主动轮询 `TaskOutput` 来获取子 Agent 的结果。这个轮询行为本身消耗父 Agent 的注意力——如果同时有多个 subagent 在运行，父 Agent 需要在多个 TaskOutput 之间调度，context 管理复杂度急剧上升。 ^[entities/claude-code-core-internals]

第三个局限是「长程任务中 loop 的可观测性」：当 agent 运行超过 50 轮时，人类已经很难追踪「当前在 loop 的哪一步」「为什么在这个步骤上卡住了」「之前尝试过哪些路径」。Claude Code 的 transcript 持久化（JSONL 格式）是对这个问题的缓解，但实际调试时仍然需要大量的日志分析工作。 ^[entities/harness-engineering-long-term-agent-tasks]

## 现实案例

Claude Code 的 query.ts 是 harness 主循环的具体实现：每次用户消息到达时，query.ts 执行 query → compact（压缩上下文）→ buildEffectiveSystemPrompt（组装 system prompt）→ LLM 调用 → 解析 tool calls → 分批调度执行 → 反馈结果 → 判断继续/停止。这个循环在每次 LLM 调用后都会检查四维治理预算，任何一维超限都会触发停止。 ^[entities/claude-code-core-internals]

Anthropic Managed Agents 的设计把 session 放在 harness 外面——session 是持久事件日志，harness 是运行时。这使得 harness 可以失败、重启，而 session log 记录了完整事件流。重启后的 harness 只需要从 session log 恢复状态，不需要模型重新理解历史上下文。这种解耦使得长程任务的故障恢复成为可能。 ^[entities/agent-harness-context-management-working-set]

Harness Engineering 的长程任务框架（long-running-agent-tasks）展示了主循环如何扩展到多会话尺度：通过 `dispatch.js` 并发调度 subagent，通过 `poll.js` 轮询状态并补位，通过 `merge.js` 收集子任务结果。任务状态机（TODO → IN_PROGRESS → DONE/FAILED/SKIPPED）使得任务完成条件可以程序化判断，任务中断可以精确恢复到最后一个 checkpoint。 ^[entities/harness-engineering-long-term-agent-tasks]

- [[entities/agent-harness-architecture|agent harness architecture]]
- [[entities/claude-code-architecture|Claude Code architecture]]
- [[entities/wow-harness-v3-governance-protocol|WOW Harness v3 governance]]
- agent loop design
- [[concepts/harness-context-window-management|harness 上下文管理]]

**一个容易被忽视的设计决策**：主循环是否允许「嵌套 spawn」。Claude Code 的 Task tool 可以在 subagent 内再 spawn subagent（max_spawn_depth=1），形成「agent→subagent→leaf」的三层结构。OpenClaw 的 orchestrator 也可以委派子任务给 worker，但 worker 通常不能再委派。这个选择直接影响系统的「自愈能力」：如果 subagent 遇到超出能力范围的任务，能 spawn 更专的 agent 是一种优雅的退化策略；但嵌套 spawn 也意味着 context 成本指数增长（每层独立窗口），必须在收益和成本间设硬上限。^[entities/claude-code-architecture]

**容错 vs 快速失败**：主循环设计最根本的分歧是「出错后怎么走」。Claude Code 偏快速失败——subagent 出错就返回错误信息让主 agent 决策；Hermes Agent 偏容错——retry with exponential backoff，出错后自动重试。两种选择没有对错：对可恢复错误（网络超时）该重试，对逻辑错误（用错 API）该快败。关键是在 harness 层面做区分，而不是全局一个策略。^[entities/hermes-agent]

## 进一步阅读

- [[entities/agent-harness-architecture]]
- [[entities/claude-code-architecture]]
- [[entities/wow-harness-v3-governance-protocol]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
