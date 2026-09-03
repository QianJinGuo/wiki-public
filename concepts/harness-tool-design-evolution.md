---
title: "harness 工具设计演化"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, harness, tool-use, claude-code, tool-design, evolution]
sources: [entities/claude-code-core-internals, entities/claude-code-harness-deep-understanding, entities/claude-code-20000-char-source-analysis]
---

## 定义

harness 工具设计演化指的是 agent 系统中 tool catalog 从粗粒度 shell exec 演化到细粒度专用工具的过程。Claude Code 等成熟 harness 的实证：工具数从 5 个膨胀到 30+ 后，必须用「工具分组、按需加载、subagent 工具隔离」三招控制 context 污染。

## 核心范式

- **粗 → 细粒度**：从 shell exec 一统天下，到 read_file/search_files/patch 等专用工具
- **专用工具的胜利**：领域专用工具比通用工具产出质量高、context 开销小
- **工具分组**：把工具按 toolset 分组（terminal / browser / file），按需开启
- **subagent 隔离**：长程任务把噪音工具放到 subagent，主 context 不被污染

## 背景与提出

harness 工具设计演化的背景是 2025-2026 年间 AI coding agent 的工具集从少数几个爆炸式扩张到数十个。Claude Code 初版只有约 10 个工具（read_file/write_file/search_files/terminal 等），到 2026 年中已有 45+ 工具分布在 40+ 子目录。这种扩张带来了一个核心工程问题：当工具数从 10 增加到 45 时，工具描述的 token 消耗会超过 context window 的 50%，导致模型无法有效使用工具——这个问题被称作「工具污染」。 ^[entities/claude-code-core-internals]

工具设计演化不是单纯的功能增加，而是对「工具粒度」「工具分组」「按需加载」的系统化设计。Claude Code 团队在实践中发现，粗粒度的 shell exec（一个工具执行所有命令）根本无法满足需求——安全审计、错误处理、权限控制在单一工具内无法实现；必须拆成 read_file/search_files/patch/terminal 等细粒度专用工具，才能对每个工具独立设置权限、预算和并发策略。 ^[entities/claude-code-core-internals]

## 范式细节

粗 → 细粒度的工具演化是经过验证的方向。Claude Code 的工具集包括：read_file（带行数限制和 offset 分页）、Grep（带上下文行数和正则支持）、Glob（带模式匹配）、WebSearch（带结果数量限制）、patch（原子性写入）、Bash（受限 shell）。每个工具都有独立的 `prompt()` 方法生成描述文本、`isConcurrencySafe()` 声明并发安全性、`maxResultSizeChars` 限制结果大小、`shouldDefer` 标记延迟加载。细粒度拆分让调度层能精确控制每个工具的行为，而不是把所有工具当作同类处理。 ^[entities/claude-code-core-internals]

专用工具的胜利体现在具体数字：Claude Code 评测显示，用专用 Grep 工具替代 shell grep，模型在代码搜索任务上的准确率从 67% 提升到 89%，同时每次调用的平均 token 消耗降低了 40%。原因是专用工具提供结构化的结果格式（而非 shell 输出的非结构化文本），让模型更容易解析和利用。 ^[entities/claude-code-core-internals]

工具分组和按需加载是控制工具污染的核心策略。Claude Code 通过 `shouldDefer` + `ToolSearch` 实现：设置了 `shouldDefer: true` 的工具（MCP 工具等）在初始请求中完全不携带 schema，只有空壳标记；模型通过 ToolSearch 搜索关键词后才把完整 schema 加入后续请求。ToolSearch 的匹配逻辑：`searchHint` 命中得 4 分，工具名命中得 2 分，完整 prompt 描述命中得 1 分。评分机制确保工具发现是精准的而非模糊匹配。 ^[entities/claude-code-core-internals]

subagent 工具隔离解决的是主 context 不被污染的问题。Claude Code 的 subagent 有内置类型（Explore/Plan/general-purpose/claude-code-guide），每种类型有独立的工具集：Explore 型只读工具、Plan 型只读工具 + ExitPlanMode、general-purpose 型全部工具（除 AgentTool 本身）。这种设计让父 Agent 可以委派探索任务给专用子 Agent，而不会把探索过程的噪音带回主窗口。 ^[entities/claude-code-core-internals]

## 局限与反对声音

第一个局限是「工具数量膨胀到某个阈值后，延迟加载本身变成性能瓶颈」：当工具数超过 100 个时，ToolSearch 的匹配精度开始下降——模型可能在多次 ToolSearch 调用中花费过多 token，延迟加载带来的节省被搜索过程本身消耗。更根本的问题是：随着工具数量增长，「工具发现」这个行为的 token 消耗本身成为了不可忽视的成本。

第二个局限是「工具描述的一致性维护成本」：当每个工具都有独立的 `prompt()` 方法时，所有工具描述必须保持风格和详略程度的一致性。在 45+ 工具的规模下，这需要专职的文档工程工作——而大多数团队没有这个资源。

第三个局限是「工具粒度的选择没有客观标准」：什么时候该拆，什么时候该合？Claude Code 团队在实践中发现，细粒度工具在大多数场景下更好，但「大多数场景」不等于「所有场景」——某些高频简单任务（如查看文件列表）反而用粗粒度工具效率更高。粒度选择依赖主观判断，缺乏系统化的决策框架。 ^[entities/claude-code-core-internals]

## 现实案例

Claude Code 的工具系统是 harness 工具设计演化的最佳案例：从 v1 的粗粒度 shell exec 演化到 v3 的 45+ 细粒度专用工具，每个工具独立声明并发安全性、结果大小上限、权限检查逻辑。工具并发调度通过 `isConcurrencySafe()` 实现：只读工具（Read/Grep/Glob/WebSearch 等）并发执行，写操作单独串行，批次之间严格顺序。 ^[entities/claude-code-core-internals]

Claude Code 的 MCP 集成展示了工具扩展的标准方式：第三方工具服务器通过 MCP 协议向 Claude Code 动态注册新工具，工具以 `mcp__serverName__toolName` 格式出现，与内置工具共享完全相同的调用机制——包括并发调度、权限检查、结果大小限制。MCP 工具默认支持延迟加载，可标记 `alwaysLoad = true` 使其始终出现在初始 system prompt 中。这种扩展机制使得 Claude Code 的工具集可以从数十个扩展到数百个而不会撑爆 context。 ^[entities/claude-code-core-internals]

Letta Code 的 overflow 文件机制是另一种工具设计思路：默认 2000 行窗口，超出的写到 overflow 文件。这个设计承认了「工具输出必须被持久化」这个事实——不是在工具层面控制大小，而是在超出后写入外部存储，供后续任务按需读取。相比 Claude Code 的精细化预算控制，Letta Code 的方式更简单但也更粗暴。 ^[entities/agent-harness-context-management-working-set]

- [[concepts/claude-code-tool-design-evolution|Claude Code 工具设计演化]]
- [[concepts/tool-use-patterns-ai-agents|tool use 模式]]
- [[concepts/harness-as-product-surface|harness 作为产品界面]]
- [[concepts/harness-loop-architecture|harness 主循环架构]]
- [[concepts/coding-harness-engineering|coding harness engineering]]

## Tool 设计的可组合性

harness tool 设计的一个进阶原则是可组合性——tool 应该设计为可被其他 tool 组合，而非只能独立调用。例如 file_read + file_search 可以组合为「先搜再读」，database_query + format_output 可以组合为「查询并格式化」。可组合的 tool 设计让 agent 可以灵活应对未预见的任务流，而不可组合的 tool 必须为每个新场景新增 tool。可组合性的代价是 tool 必须有清晰的输入输出 schema（machine-readable），开发成本略高但长期灵活性更好。

## 进一步阅读

- [[entities/claude-code-core-internals]]
- [[entities/claude-code-harness-deep-understanding]]
- [[entities/claude-code-20000-char-source-analysis]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
