---
title: "subagent 派生模式"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, subagent, spawn, multi-agent, claude-code, openclaw]
sources: [entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏, entities/claude-code-core-internals]
---

## 定义

subagent 派生模式：主 agent 在需要时启动一个上下文隔离的 child agent 处理子任务，child 完成后返回结果总结给主 agent。Claude Code 的 Task tool、Hermes 的 delegate_task、OpenClaw 的 lobster 都是这一模式的实现。

## 核心范式

- **显式调度**：主 agent 决定何时 spawn、传什么 context 给 child
- **结果摘要返回**：child 不返回完整 tool trace，只返回结构化摘要，节省 token
- **深度限制**：通常禁止 child 再 spawn grandchild，防止资源失控
- **适用场景**：长链分析、并行调研、隔离风险操作（试错不污染主 context）

### 背景与提出

subagent 派生模式的技术起点是 2023 年 Claude Code 引入的 Task tool——这是第一个被大规模使用的「在 AI agent 内部再启动一个 AI agent」的实现。在 Task tool 之前，agent 系统的常见模式还是「一个 LLM + 一堆 tool」，能力边界受限于单个 context window 的容量。Task tool 的关键洞察是：可以把一个复杂任务「暂停」当前 agent，启动一个全新的 agent 在独立 context 里处理子任务，完成后把结果摘要返回主 agent，主 agent 继续原有任务。

这个模式后来被其他框架广泛借鉴：OpenClaw 的 lobster、AutoGen 的 AssistantAgent + UserProxyAgent 组合、Hermes 的 delegate_task。名字不同，但核心机制一致：主 agent 控制生命周期，child agent 独立运行，结果结构化返回。^[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]

这个模式的出现也对应了 LLM 应用的一个根本矛盾：单个 context window 无法同时容纳「完整的任务上下文」和「完整的工具执行日志」。当 tool 调用次数增加（比如一次爬虫任务产生 500 条日志），日志会把有用上下文挤出 window。subagent 隔离让 tool log 只存在于 child 的 context，主 agent 只看到 child 最终返回的「结论」，解决了日志膨胀的问题。

### 范式细节

**显式调度**意味着 subagent 的启动是主 agent 的主动决策，而非系统自动行为。主 agent 在决定 spawn child 之前，需要判断：子任务是否足够独立（不依赖主 context 里的其他中间结果）、子任务是否足够复杂（值得消耗一次完整的 LLM 调用 overhead）、子任务的失败是否可接受（child 失败不应该导致主任务崩溃）。这三个判断本身就是 prompt engineering 的一部分——很多新手实现 subagent 时，发现主 agent 根本不愿意 spawn child，就是因为判断逻辑没有写清楚。

**结果摘要返回**是 token 成本控制的核心机制。假设一个爬虫子任务产生了 200 次 tool calls，raw output 可能超过 50K token，但主 agent 真正需要的信息可能只有 200 字：「爬取完成，共 150 个页面，50 个有有效内容，3 个域名解析失败」。从 50K → 200 字的压缩比达到 250:1，大幅节省了主 context 的空间。实现上，通常要求 child agent 在完成所有 tool calls 后，再调用一次 LLM 生成结构化摘要，而非直接把 raw log 返回。

**深度限制**是防止资源失控的安全机制。如果允许 child agent 继续 spawn grandchild，理论上可以产生无限深度的递归，每个层级都消耗独立的 LLM 调用成本。实践中通常限制最大深度为 2（主 → child），少数系统允许到 3。深度限制的实现方式有两种：一是限制 tool 可用性（child 的 tool list 里没有「spawn」相关的 tool），二是运行时检查（每次 spawn 前检查当前深度计数器）。

**适用场景**的判断经验：适合用 subagent 的任务特征是「高风险/高变化/高并行」，不适合的是「短小精悍/确定性流程/错误需要即时反馈」。一个具体的判断标准是：子任务是否需要消耗超过 50 次 tool calls？是否会产生大量中间日志？结果是否需要等所有步骤完成才能判断成功/失败？如果三个都是 yes，subagent 是好选择。如果三个都是 no，用 function calling 即可，不需要启动独立 agent。

### 局限与反对声音

subagent 模式最大的工程问题是「状态丢失」。child agent 运行在完全独立的 context 里，它中途产生的任何中间状态（变量、文件、全局对象）主 agent 无法访问，只能通过 child 最终的返回值获取。如果 child 因为某种原因提前退出（比如 context window 满了），主 agent 只能知道「它失败了」，但不知道失败发生在哪一步、已经完成了多少工作。这个 observability 黑洞是 subagent 模式的主要运维挑战。

第二个问题是「错误传播的复杂性」。child 执行过程中产生的错误，是返回给主 agent 处理，还是 child 自己处理？如果是后者，child 需要有完整的 error handling 能力——这意味着 child 的 prompt 里要包含「如果 X 错误，则执行 Y」这样的规则，而这正是 v1 版本 agent 的做法，容易导致 prompt 膨胀。实践中多数系统选择「child 只管执行，错误统一由 orchestrator 处理」，但这意味着 orchestrator 需要有足够的错误信息来判断如何重试。

第三个批评是「overhead 不值得」。启动一个 subagent 的固定成本包括：一次新的 LLM system prompt + context initialization（通常 2-5K token）+ 结果摘要（至少一次 LLM 调用）。对于一个只需要调用 3-5 次 tool 的简单任务，这个 overhead 可能占总成本的 30-50%。经验建议是，只有当子任务需要 20+ 次 tool calls 或需要跨越多个独立 domain（比如先爬虫再分析再写报告）时，subagent 的 overhead 才合理。

### 现实案例

[[entities/claude-code-core-internals|Claude Code]] 的 Task tool 是 subagent 模式最成熟的工业实现。每次 Task spawn 时，Claude Code 会创建一个新的上下文环境，包含主 session 的「快照」（用户提供的 @file 内容、当前工作目录状态、必要的系统信息），child 在这个快照里运行，完成后返回结构化结果。主 session 的 conversation history 不包含 child 的中间过程——这是一个关键设计，因为如果主 context 包含了所有 subagent 的完整日志，context 会在几次 subagent 调用后迅速膨胀。

[[concepts/multi-agent-context-isolation|多智能体上下文隔离]] 的另一个典型案例是 OpenClaw 的 lobster 工具：它允许主 agent 在遇到需要长时间运行的命令（比如 `npm install`，可能需要 5 分钟）时，spawn 一个独立的 lobster child 去执行，主 agent 可以并行处理其他任务。lobster child 执行完毕后通过消息队列通知主 agent，主 agent 再决定是否读取结果。这个模式的变种被用在 [[concepts/orchestrator-worker-architecture|orchestrator-worker 架构]] 中，worker 本质上就是长期运行的 subagent。

## 在 wiki 中的关联

- [[entities/claude-code-core-internals|Claude Code 核心机制]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw subagent 实战]]
- [[concepts/multi-agent-team-coordination|多智能体团队协作]]
- [[concepts/multi-agent-context-isolation|多智能体上下文隔离]]
- [[concepts/orchestrator-worker-architecture|orchestrator-worker 架构]]

## 进一步阅读

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/claude-code-core-internals]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
