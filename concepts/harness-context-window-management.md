---
title: "harness 上下文窗口管理"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, harness, context-management, working-set, agent, context-engineering]
sources: [entities/agent-harness-context-management-working-set, entities/harness-engineering, entities/agent-harness-architecture]
---

## 定义

harness 上下文窗口管理是 agentic engineering 的核心子问题：在固定 token 上限下，决定 working set 装什么、按什么优先级裁剪、什么时候 swap 进 long-term memory。这是 harness 工程化最重的部分——上下文管不好，再强的模型也跑不出长程任务。

## 核心范式

- **working set 概念**：当前 turn 必须看到的最小信息集，类似 OS 工作集理论
- **优先级裁剪**：system prompt > 任务指令 > 最近 turns > 历史摘要，按优先级丢弃
- **swap 到 long-term memory**：把丢弃的信息存到外部存储，按需召回
- **评估指标**：context utilization rate / recall accuracy / token budget overshoot

## 背景与提出

harness 上下文窗口管理的概念在 2026 年由 Anthropic 官方话语权正式命名为「Context Engineering（CE）」——比 prompt engineering 范围更大的工程学科。这个命名标志着社区长期实践的「上下文管理」正式成为独立子学科。 ^[entities/agent-harness-context-management-working-set]

Context Engineering 提出的背景是：随着 Claude Code 等工具中 context window 越来越大（200K token 级别），如何管理这个巨大的上下文空间成了一个独立的工程问题。传统 PE（Prompt Engineering）关注的是「这一轮 prompt 写什么」，而 CE 关注的是「过去 + 未来的整体上下文状态」——包括工具输出预算、状态分舱、隔离、摘要、预算等多个维度。CE = PE 的超集，未来讨论 LLM 工程时，CE 可能会取代 PE 成为主流框架。 ^[entities/agent-harness-context-management-working-set]

[[entities/claude-code-context-engineering-anthropic-thariq|Claude Code 的 Context Engineering 实践]] 由 Anthropic 官方博客背书，将社区分散的上下文管理经验系统化为五大实战模式。[[entities/agentic-ai-infrastructure-practice-series-nine-context-engineering|九大上下文工程基础设施实践]] 则从企业级基础设施角度补充了另一视角：如何在多 Agent 并发场景下做全局 context 预算控制，而不只是单 Agent 内部的 context 优化。

## 范式细节

working set 概念来自 OS 内存管理理论——进程运行时活跃访问的页面集合构成工作集。将这个概念映射到 Agent 上下文管理：模型注意力 ≈ CPU 寄存器访问，工具输出缓存 ≈ L1/L2 缓存，分页工作集 ≈ 压缩后上下文，磁盘/Swap ≈ overflow 文件和 memory repo。关键区别在于：OS 内存管理只能根据访问频率做统计推断，而 Agent Harness 知道任务语义——哪些是用户目标、哪些是探索过程、哪些是中间错误，这让压缩决策远比 OS 页面替换更精准。 ^[entities/agent-harness-context-management-working-set]

Claude Code 的五层递进式 Context 压缩机制是 CE 实践的具体化：1）工具结果预算（applyToolResultBudget）——工具返回超限时写入磁盘，替换为路径引用；2）历史片段截断（snipCompact）——不调用 LLM，通过规则对历史消息打分；3）微压缩（microCompact）——利用 `cache_edits` 在服务端屏蔽旧工具结果，本地消息不动，cache 命中；4）上下文折叠（contextCollapse）——折叠旧对话但保留近期原始精度；5）完整摘要压缩（autoCompact）——fork 子 Agent 生成完整摘要。五层策略对应五种不同触发条件，互相兜底而非互相竞争。 ^[entities/claude-code-core-internals]

[[entities/claude-code-7-layer-memory-architecture|Claude Code 七层记忆架构]] 揭示了另一层设计逻辑：工具调用结果、知识库检索结果、用户个人偏好、项目规范等不同类型的上下文信息，在进入主 context window 之前就已经被分类处理了。这种「入 context 前就分流」的设计比「先塞进去再压缩」要高效得多——因为压缩本身是有信息损耗的，能不压就不压才是最优策略。

pre-query optimization 是 Claude Code 的设计亮点：在每次 API 调用前都处理工具输出，而非等窗口快满再救火。这相当于 OS 的 proactive memory management——提前整理工作集，而不是等 page fault 了才换入换出。如果只在 compaction 时才处理工具输出，会导致：窗口被工具输出持续撑大；compaction 触发时需处理内容更多，质量下降；模型在工具输出处理上浪费注意力。 ^[entities/agent-harness-context-management-working-set]

[[entities/context-engineering-three-memory-paradigms|三种记忆范式的对比分析]] 从更底层梳理了 working set 和 long-term memory 的关系：工作记忆（关注点窗口内）、情景记忆（当前 session 的历史）、语义记忆（跨 session 积累的概念知识）。Context Engineering 的核心任务就是管理这三种记忆之间的流转——哪些应该驻留在工作记忆，哪些应该落盘为情景记忆，哪些应该被蒸馏为语义记忆。

## 局限与反对声音

第一个局限是「compaction 本身可能撑爆窗口」：compaction 逻辑本身也是工具调用，消耗上下文。Claude Code 的兜底方案是：先钳工具返回到小阈值再重试；仍不行对 transcript 中间截断（留头留尾、丢中间）；或按 API-round 成组扔掉最旧的组。这是 200K+ token 级别对话中必然遇到的实际问题，不是理论担忧。 ^[entities/agent-harness-context-management-working-set]

第二个局限是「压缩质量的不可验证性」：compaction 后的摘要是否仍然包含足够的信息来支撑当前任务，这个判断本身没有自动化的标准。autoCompact 通过 fork 子 Agent 生成摘要，但这个子 Agent 本身也无法验证「我生成的摘要是否丢失了关键信息」——这是一个递归问题。 ^[entities/agent-harness-context-management-working-set]

第三个局限是「subagent 上下文泄露」：如果子 Agent 继承了父对话，父窗口会被探索过程中的中间结果污染。隔离策略必须在系统层面强制执行，不能依赖模型「自觉」。OpenClaw 和 Claude Code 在这一点上都是从架构层面保证的——但这也意味着隔离策略的设计是 harness 工程中最复杂的部分之一。 ^[entities/agent-harness-context-management-working-set]

## 现实案例

Claude Code 是 CE 实践的完整实现：五层压缩策略（工具结果预算/microCompact/contextCollapse/autoCompact）、pre-query optimization（在每次 API 调用前整理工具输出）、subagent 隔离（typed-agent 空白对话 + 工具 allowlist）。这五层压缩的触发阈值与模型 context window 动态绑定：对于 claude-sonnet（200K context），自动压缩约在 167K token 时触发。 ^[entities/claude-code-core-internals]

OpenClaw 的 bootstrap 机制是另一种 CE 思路：bootstrap 单文件 12K 字符，总共 60K；工具输出 16K 或 30% 上下文。这是用「硬上限」而非「动态压缩」来管理 context 的策略——优点是简单可预测，缺点是可能在任务中途强制截断有效信息。两种思路代表不同的工程权衡：Claude Code 选择更精细的动态管理，OpenClaw 选择更简单的静态限制。 ^[entities/agent-harness-context-management-working-set]

Anthropic 官方提出的 Context Engineering 五大实战模式（Anthropic 官方清单）具体化了 CE 的最佳实践：Quarantined subagent（隔离区 subagent）、Auto-summarization（自动摘要）、Tool output budgeting（工具输出预算）、State sharding（状态分舱）、Subagent 做 narrow read。五模式与已有的框架实践一一对应，代表了社区分散实践的官方集大成。 ^[entities/agent-harness-context-management-working-set]

[[entities/agent-memory-architecture-essence|Agent 记忆架构精华]] 从记忆系统的角度印证了 Context Engineering 的必要性：Claude Code 的五层压缩机制实际上是分层的记忆管理系统——microCompact 对应工作记忆的碎片清理，autoCompact 对应情景记忆的摘要生成，而长期记忆的检索则依赖外部 memory repo 的召回机制。三层架构不是 Claude Code 特有的，而是几乎所有长程 Agent 的共同设计模式。

## Context 管理的工程指标

可衡量的 context 管理指标：context 利用率（实际使用 token / context window 容量），理想 60-80%；context 切换成本（每次裁剪/重载的延迟），理想 < 200ms；context 命中率（重要信息是否在 L1），理想 > 95%。这些指标可以用 LangSmith / Helicone / OpenLLMetry 等可观测工具持续监控。context 管理的下一步演进方向是「自动 context 优化」——agent 自己根据任务动态调整 context 内容，类似 JIT 编译器的运行时优化。这需要 LLM 自身有「meta-cognition」能力，是 2026-2027 年的研究热点。

[[concepts/context-engineering|Context Engineering]] 作为独立概念，已经从 prompt engineering 中正式分离出来——这不仅仅是命名问题，而是工程学科成熟度的标志。PE 关注「单次交互的 prompt 质量」，CE 关注「跨时间窗口的信息状态管理」——后者的工程复杂度远高于前者，因为信息在时间维度上的累积效应是非线性的。

**实际工程中最重要的教训**：不要把 working set 管理和 long-term memory 管理混为一谈。Working set 解决的是「当前 turn 该看什么」，long-term memory 解决的是「跨 session 该记住什么」——优化目标不同、淘汰策略不同、存储介质不同。把它们耦合在一起的结果是：working set 膨胀（塞了太多「万一有用」的长时记忆），或者 long-term memory 贫血（太积极地把信息从 working set 换出就删了）。正确做法是两层独立管理、只在「promote/decay」边界处做一次同步。^[entities/agent-harness-context-management-working-set]

**CE 和 PE 的实际分工线**：在日常 agent 开发中，prompt engineering 解决「模型怎么理解任务」，context engineering 解决「模型能看到什么」。一个常见的反模式是把 context 管理问题用 prompt 来解——在 system prompt 里写「请忽略无关信息」——这等于让模型在运行时做本该在编译时做的裁剪，既浪费 token 又不可靠。正确做法是 harness 层面先裁好 context，再让 prompt 聚焦任务本身。^[entities/agent-harness-context-management-working-set]

## 进一步阅读

- [[entities/agent-harness-context-management-working-set|Agent harness working set]]
- [[entities/harness-engineering|Harness Engineering]]
- [[entities/agent-harness-architecture|Agent Harness 架构]]
- [[concepts/working-set-vs-long-term-memory|working set vs long-term memory]]
- [[concepts/context-engineering|context engineering]]
- long context techniques
- [[concepts/harness-loop-architecture|harness 主循环]]

## 所属 MOC

- [[moc/layer-2-interaction|Layer 2 Interaction]]
- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
