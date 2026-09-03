---
title: "agentic engineering 工程范式"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, agentic-engineering, karpathy, harness, verifier, paradigm]
sources: [entities/karpathy-vibe-coding-agentic-engineering, entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering, entities/karpathy-vibe-coding-to-agentic-engineering]
---

## 定义

agentic engineering 是 Karpathy 2026 年提出的范式升级：把 vibe coding 的「让 AI 帮你写出来」工程化为「让 AI 在一套可验证、可托管、可问责的系统里把东西交付出去」。核心对象从「代码生成」迁移到「harness + context + verifier」三件套。

## 核心范式

- **harness 是一等公民**：不再问「模型多强」，而问「围绕模型的系统多稳」
- **context 工程化**：把 working set、long-term memory、tool catalog 都视为可优化对象
- **verifier 必须存在**：所有 agent 产出必须有自动校验通道，否则不算交付
- **问责性**：每次行为可追溯，错了能 rollback、能定位根因

## 背景与提出

2026 年 Karpathy 在采访中修正了自己一年前的 vibe coding 主张，提出 agentic engineering 才是 AGI-native 开发的真正范式。 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering] 这个修正绝非话术翻新——它源于一个工程实践的事实：模型能力快速提升（Claude Opus 4 → 4.5 → 4.6 → 4.7），但围绕模型构建的系统稳定性没有自动提升，反而因为模型更强导致的输出更多样化而更不可控。

核心洞察来自一次具体的技术对比：vibe coding 的「接受/拒绝」交互模式无法scale——当任务从「写一个函数」扩展到「部署一套微服务」时，AI 每轮输出的选项数量和复杂度呈指数增长，人类判断者根本不可能在合理时间内做出有质量的「接受/拒绝」决策。解决这个问题的路径不是训练更强的模型，而是建立一套「AI 生产流水线」——让系统本身来承担质量控制和交付责任。

这个思路的工程化起点是 Karpathy 在 2026 年初提出的一个具体数字：context window 是新的代码行数。在 Software 3.0 时代，程序员操控的不再是「写进文件的代码」，而是「装入 context 的信息量」——这个视角转移直接催生了「harness + context + verifier」三件套的系统化框架。 ^[entities/karpathy-vibe-coding-agentic-engineering]

## 范式细节

agentic engineering 的三件套是 harness、context 和 verifier。Harness 是 agent 的运行环境——LLM 调用、工具执行、状态持久化、错误恢复的框架。Context 是 agent 每 turn 能看到的全部信息——system prompt、working set、tool descriptions、conversation history。Verifier 是每步产出的质量门——测试、lint、类型检查、安全扫描、人工批准。三者形成一个「生产流水线」：context 决定模型能正确「知道多少」，harness 决定模型能「做」什么，verifier 决定「能不能交付」。

具体数字来看：Claude Code 的五层递进式 Context 压缩机制（microCompact/contextCollapse/autoCompact）能在 200K token 窗口内处理超过 50 个文件的并发操作，每层压缩策略针对不同触发条件——时间触发的 microCompact 利用 `cache_edits` 在服务端屏蔽旧工具结果而不修改本地消息，保持 prompt cache 命中；autoCompact 通过 fork 子 Agent 生成摘要，牺牲约 15% 的上下文精度换取 60% 的 token 节省。 ^[entities/claude-code-core-internals]

harness 作为一等公民的核心工程体现是「治理协议」——每次 LLM 调用前，系统检查 token 预算、成本上限、context 大小、工具权限，四维独立控制而非单一预算。以 Claude Code 为例：Token 预算通过 `output_config.task_budget` 设置；成本预算通过 `maxBudgetUsd` 设置单次会话美元上限；工具结果预算是每个工具声明的 `maxResultSizeChars`；轮次预算通过 `maxTurns` 限制最大迭代次数。这四维中的任何一维超限，agent 循环都会立即停止。 ^[entities/claude-code-core-internals]

verifier 必须存在这条原则的具体化是 Claude Code 的 Plan Mode：写操作在权限检查阶段就被拦截，不依赖模型是否「听话」。模型完成规划后调用 ExitPlanMode，系统将规划内容写入 `.claude/plans/` 下的 Markdown 文件，用户必须在 UI 中显式点击批准，才能将权限模式从 `'plan'` 恢复为原始模式。这个设计把「人类审查」从可选项变成了系统强制执行的质量门。 ^[entities/claude-code-core-internals]

## 局限与反对声音

反对者认为 agentic engineering 的本质是「把已有的软件工程实践搬到 AI 上面」——测试、CI/CD、code review 这些东西在传统软件工程里都存在，换个名字没新意。Google 的 Addy Osmani 在 2026 年初发表《Agentic Engineering》时明确区分了两者：传统软件开发是「人写代码，机器执行」，而 agentic engineering 是「AI 写代码，AI 自己的运行时执行」——多了一个概率层，导致「写」「执行」「验收」三者的信任关系完全不同。

更深层的质疑是：当 AI 产出超过人类理解能力时，人类 gate 就是形式主义。例如 AI 生成了一段复杂的分布式一致性算法，人类 reviewer 能否真的判断其正确性？如果不能，这个 gate 的意义就只剩下「合规」而非「质量保证」。Karpathy 本人也在访谈中承认：「我有时候也不知道 AI 写的代码对不对，我只是相信系统设计。」 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]

另一类质疑更根本：verifier 本身由谁验证？当 AI 产出超过人类理解能力时，人类 gate 就是形式主义。更具体地说，LLM-as-judge 存在已知的 self-enhancement bias——评审模型也用同一模型家族，会对同家族的输出有系统性偏爱。 ^[entities/karpathy-vibe-coding-to-agentic-engineering]

## 现实案例

Hermes Agent 的 cronjob + skill 系统是 agentic engineering 的完整实现：harness = Hermes Agent runtime（cron, terminal, delegate_task），context = Memory + Skills + Wiki 三层注入，verifier = lint pre-commit gate + 用户 final review。三件套全部在同一个框架里走通，而非分散到不同工具。 ^[entities/hermes-agent]

Claude Code 的权限系统四模式（default/auto/plan/bypassPermissions）是 agentic engineering 在产品层面的具体化：default 模式每次工具调用弹出确认对话框；auto 模式由 AI 分类器判断是否放行；plan 模式只读操作自动放行、写操作拦截；bypassPermissions 跳过所有检查。其中 auto 模式的 AI 分类器是真正有趣的部分——它是一个独立的小模型，专门判断操作是否安全，而非依赖规则匹配。 ^[entities/claude-code-core-internals]

OpenClaw 的 subagent 隔离策略也体现了 agentic engineering 原则：子 Agent 默认获得 fresh isolated session，携带过滤后的 AGENTS.md/TOOLS.md/SOUL.md，但不带父 transcript——探索性任务的中间结果不污染主窗口。父 Agent 只拿到最终结果和必要的上下文继承，主窗口的 token 预算因此得到保护。 ^[entities/agent-harness-context-management-working-set]

## 三件套的相互依赖

harness / context / verifier 三件套不是独立的——它们之间有强依赖。harness 决定 context 能装什么（不同 harness 提供不同的 tool 输出格式和 working set API）。context 决定 verifier 能检查什么（context 里有 spec 才能 verify）。verifier 决定 harness 能多激进（没有 verifier 时 harness 必须保守，有 verifier 时可以快速迭代）。三件套一起设计、一起演化、一起重构是 agentic engineering 的工程纪律，缺一件整套范式就失效。

## 在 wiki 中的关联

- [[entities/karpathy-vibe-coding-agentic-engineering|Karpathy 原文方法论]]
- [[concepts/vibe-coding-paradigm|vibe coding 范式（前身）]]
- [[concepts/verifier-driven-development|verifier-driven development]]
- [[concepts/harness-engineering-paradigm-shift|harness engineering 范式迁移]]
- [[concepts/harness-loop-architecture|harness 主循环架构]]

## 进一步阅读

- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/karpathy-vibe-coding-to-agentic-engineering]]

## 所属 MOC

- [[moc/layer-0-foundation|Layer 0 Foundation]]
- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
