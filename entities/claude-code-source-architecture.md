---
title: "Claude Code 源码拆解：从启动到多 Agent 扩展层"
created: 2026-04-25
updated: 2026-09-07
type: entity
tags: [agent, architecture, claude-code, source-analysis, yc]
sources: [raw/articles/claude-code-source-architecture]
review_value: 8
review_confidence: 7
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---
## 八大模块
1. **入口三段式**：分流→进程初始化→会话准备，进程/交互状态分离 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
2. **REPL 控制台**：不是消息展示器，而是汇总能力面+归并事件流的运行时控制面 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
3. **Query Loop 状态机**：prefetch→budget→compact→stream→tool→write-back 闭环，维护跨迭代状态 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
4. **Tool Runtime**：四段式执行（解析→校验→权限→执行），横切问题（校验/并发/错误/回填）收敛统一 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
5. **Permission System**：四层权限模型（规则→运行时判定→交互→沙箱隔离），可解释的执行链 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
6. **Task Runtime**：统一任务抽象（pendingMessages/isBackgrounded/retain），子Agent先任务后智能体 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
7. **扩展层**：MCP/Skills/Plugins统一翻译为内部对象，外部动态/内部稳定 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
8. **总架构**：三条主干链路（控制链/执行链/任务链）+扩展层 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]

## 核心架构原则
- **复杂度分层**：边界→启动层，连续运行→Query Loop，行动协议→Tool Runtime，风险治理→Permission，长时执行→Task Runtime，平台增长→扩展层
- **三条主干链路**：控制链（定边界+推理）、执行链（Tool→Permission→sandbox→副作用）、任务链（后台/子Agent/远程统一生命周期）
- **外部可以热闹，内部必须收敛**

## 关键设计对比
| 模块 | 常见错误 | Claude Code 做法 | ^[raw/articles/claude-code-source-architecture.md]
|------|---------|----------------| ^[raw/articles/claude-code-source-architecture.md]
| 启动层 | 一个大main拉起全世界 | 三段式分流，复杂度前置 | ^[raw/articles/claude-code-source-architecture.md]
| REPL | 薄输入框+消息列表 | 运行时控制台，能力面汇总 | ^[raw/articles/claude-code-source-architecture.md]
| Query Loop | 单次模型调用 | 状态机，跨迭代状态维护 | ^[raw/articles/claude-code-source-architecture.md]
| Tool Runtime | 简单函数签名 | 带schema/校验/权限/并发/回填的受控协议 | ^[raw/articles/claude-code-source-architecture.md]
| Permission | 弹窗机制 | 四层可解释执行链+沙箱隔离 | ^[raw/articles/claude-code-source-architecture.md]
| 多Agent | prompt分工 | 统一任务抽象，生命周期管理 | ^[raw/articles/claude-code-source-architecture.md]

## 相关维度的深度分析
- **[[entities/claude-code-prompt-context-harness]]**（飞樰）侧重 Prompt 模块化/Harness 安全/多 Agent 体系
- **[[entities/claude-code-agent-engineering]]**（SooKool）侧重 StreamingToolExecutor/主循环/压缩/小模型/Hook
- **[[concepts/openclaw-architecture]]**（800行轻量架构）与 Claude Code 同体系但更精简
- **[[entities/agent-skill-writing]]** Skill 编写规范对应 Fat Skills 理念
- **[[concepts/hermes-agent]]** Hermes 的 Self-Evolving 与 Claude Code 架构的关系

## 相关实体
- [[entities/claude-code-skills-mcp-rules-source-analysis|Claude Code 源码解析：Skills/MCP/Rules 底层机制对比]]
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 提示词体系源码解析]]
- [[entities/claude-code-open-source-model-enterprise-practice|Claude Code 接入自建开源模型：企业私有化与降本实践 | 亚马逊AWS官方博客]]
- [[entities/claude-code-architecture-analysis|Claude Code 设计原则与对照分析]]
- [[entities/claude-code-architecture|Claude Code 架构解析]]
- [[entities/claude-code-source-deep-dive-warrior|Claude Code 源码深度解析（13 核心机制）]]
- [[entities/boris-cherny-ide-to-agent-console|Boris Cherny — 从 IDE 到 Agent 控制台]]
- [[concepts/agent-backend-unification|Agent 与后端统一架构]]
- [[concepts/claude-code-deep-architecture-analysis|Claude Code 架构深度分析]]
- [[entities/从多智能体编排到ai自主决策资损防控体系的架构演进|从多智能体编排到AI自主决策：资损防控体系的架构演进]]
- [[entities/hermes-agent-kanban-deep-test|Hermes-Agent Kanban 实测 — 商业 CLI 作为上层 Orchestrator]]
- [[entities/读完-claude-code-和-openclaw-的-memory-源码我对agent记忆需要向量数据库这件事产生了怀疑|Claude Code vs OpenClaw 记忆系统 — 向量数据库必要性反思]]
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]]
- [[entities/claude-code-harness-deep-understanding|深入理解 Claude Code 源码中的 Agent Harness 构建之道]]
- [[queries/agent-memory-system-design|Agent Memory System 设计指南]]
- [[entities/agent-harness-architecture|Agent Harness 架构]]
- [[entities/anthropic-官方技能最佳实践14-个可复用的-agent-skills-设计模式|Anthropic 官方技能最佳实践：14 个可复用的 Agent Skills 设计模式]]
- [[entities/构建基于多智能体架构的深度思考交易系统.md|基于多智能体架构的深度思考交易系统]]
- [[entities/imclaw通过微信飞书操控claude-code-coodex-gemini-clipi-agent蜂群|IMClaw：通过微信/飞书操控ClaudeCode/Codex/GeminiCLI/Pi Agent蜂群]]
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]]
- [[entities/claude-code-mcp-server|Claude Code MCP Server]]
- [[queries/sales-team-agent-knowledge-base-architecture|200人销售团队企业级 Agent 知识库问答系统架构设计]]
- [[entities/context-window-management|Agent 上下文窗口管理对比]]
- [[concepts/agent-memory-systematic-framework|Agent Memory 系统性框架]]
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]]
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台-v2|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]]
- [[entities/claude-发布官方报告承认存在-3-处质量退化问题|Claude 发布官方报告，承认存在 3 处质量退化问题]]

- [[entities/claude-code开发负责人-为何放弃rag而选择agentic-search|Claude Code 开发负责人：为何放弃 RAG 而选择 Agentic Search]]
- [[entities/harness-production-agent-engineering-deficit|Harness如何支撑Agent在生产环境稳定运行？]]
- [[entities/agent-architecture-harness-new-backend|Agent架构关键变化：Harness正在成为新后端]]
- [[entities/claude-code-7-layer-memory-architecture|claude-code-7-layer-memory-architecture]]
- [[entities/agent-engineering-principles-architecture-practice|Agent 原理、架构与工程实践]] ^[raw/articles/claude-code-source-architecture.md]

- [[moc/wiki-master-map|MOC]]
## 深度分析
### 为什么复杂度必须安放在正确的位置
Claude Code 最大的架构张力在于：它要同时支持本地交互、headless、SDK、remote session、后台任务、会话恢复。如果这些都堆在一个 main 函数里，或者靠 prompt 里的人肉兜底，系统一复杂必然散架。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
无岳的核心论点是：**复杂度有它该待的位置，不是哪里方便塞哪里**。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
启动层的复杂度（进程状态 vs 交互状态分离）→ 交给三段式启动提前定型 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
连续运行阶段的复杂度（上下文治理/失败恢复/工具回灌）→ 交给 Query Loop 状态机 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
工具副作用的复杂度（校验/并发/错误/回填）→ 交给 Tool Runtime 四段式 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
权限判定复杂度（规则/运行时/交互/沙箱隔离）→ 交给 Permission System 四层模型 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
多 Agent 的复杂度（状态管理/进度观察/结果回流/上下文隔离）→ 交给统一 Task 抽象 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
这本质上是**关注点分离（Separation of Concerns）**在 Agent 系统里的具体实现。每个模块只管自己那一层，不跨层越俎代庖。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]

### 三条链路的分层价值
控制链（启动层 → REPL → Query Loop）解决的是"怎么想、怎么续跑"。它定义了系统的边界、制度、连续运行时的状态维护。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
执行链（Tool Runtime → Permission Decision → sandbox → 副作用）解决的是"怎么动、怎么受约束"。它把"模型想动"变成"在受控边界内真正执行"。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
任务链（Task Runtime）解决的是"怎么并发、怎么持续、怎么回流"。它让后台任务、子 Agent、远程执行都共享同一套生命周期管理，不需要为每种场景重新发明状态管理。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
扩展层（ MCP/Skills/Plugins）不是第四条孤立的链路，而是给三条主链注入能力，但坚持"外部可以热闹，内部必须收敛"——外部来源统一翻译成内部对象，系统内部只说同一种语言。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]

### REPL 不是视图组件
大多数 Agent 系统的 UI 就是"消息列表 + 输入框"。Claude Code 的 REPL 被设计成**运行时控制台**，负责两件事：汇总当前能力面（本地 tools + MCP tools + plugin commands + 动态 skills + 任务状态 + 权限确认队列 + 远程会话信息）和归并当前事件流（assistant message + tool progress + pending permission + task notification + API error）。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
这个设计让用户能看到系统正在执行什么、为什么停下来、当前有哪些能力、后台有哪些任务，而不只是被动接收消息。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]

### 子 Agent 先是任务对象，才是智能体
Claude Code 的 Task 统一表达了：主会话后台化、本地 subagent、in-process teammate、remote agent、任务通知/状态/输出/恢复。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]
关键洞察是：**子 Agent 先是任务对象，才是智能体**。前后台差别被降为调度与可见性差异，而不是两套世界观。这让多 Agent 没有把主会话撕裂，而是 task system 的自然外延。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]

### 坏路径的显式设计
大多数 Agent 系统只设计happy path，遇到 prompt-too-long、max_output_tokens、fallback model 这些情况就靠 prompt 里加 if-else 兜底。Claude Code 把这些坏路径全部显式设计了 runtime 路径——上下文治理、失败恢复、工具回灌都是 runtime 课题，不是 prompt 技巧。 ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]

→ [[raw/articles/claude-code-source-architecture.md|原文存档]] ^[raw/articles/claude-code-source-architecture.md]^[raw/articles/claude-code-source-architecture.md]

## 实践启示
1. **先定义执行边界，再发起第一轮推理**：启动层不把模式/边界/权限/上下文装配清楚，后面每个宿主都会偷偷长出自己的运行语义。三段式启动（入口分流 → 进程级初始化 → 会话级准备）值得借鉴。^[raw/articles/claude-code-source-architecture.md]
2. **当 Agent 进入连续运行阶段，Query Loop 必须升级成状态机**：不是单次模型调用，而是 prefetch → budget → compact → stream → tool → write-back 的闭环，维护跨迭代状态（messages、toolUseContext、maxOutputTokensOverride、autoCompactTracking 等）。^[raw/articles/claude-code-source-architecture.md]
3. **工具一旦开始碰副作用，工具层就必须制度化**：四段式执行（解析 → 校验 → 权限 → 执行）统一处理参数校验、并发治理、进度上报、错误归一化、结果回填。Tool Runtime 是"把自由发挥变成可交付动作"的那一层。^[raw/articles/claude-code-source-architecture.md]
4. **权限系统的核心不是确认框，而是可解释的执行链**：四层权限模型（规则 → 运行时判定 → 交互 → 沙箱隔离）加上结构化的 PermissionDecision 对象（allow/ask/deny），让权限决策可追溯、可干预。^[raw/articles/claude-code-source-architecture.md]
5. **多 Agent 的前提不是 prompt 分工，而是统一任务抽象**：把后台任务、子任务、远程执行统一到一张任务状态表上，子 Agent 先是任务对象（pendingMessages/isBackgrounded/retain），才是智能体。^[raw/articles/claude-code-source-architecture.md]
6. **外部可以热闹，内部必须收敛**：MCP/Skills/Plugins 统一翻译成内部对象，系统内部只说同一种语言，不让扩展来源导致内部对象模型爆炸。^[raw/articles/claude-code-source-architecture.md]
7. **REPL 是运行时控制台，不是消息展示器**：汇总能力面 + 归并事件流，让用户看到系统正在执行什么、为什么停下来、当前有哪些能力、后台有哪些任务。^[raw/articles/claude-code-source-architecture.md]
8. **坏路径需要显式设计**：上下文治理（snip → microcompact → collapse → autocompact）、失败恢复（reactive compact、max output recovery、fallback model）、工具回灌都要有 runtime 路径，不能只靠 prompt 兜底。^[raw/articles/claude-code-source-architecture.md]