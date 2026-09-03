---
title: "Claude Code 架构解析"
created: 2026-04-24
updated: 2026-08-29
type: entity
tags: [claude-code, agent, architecture, runtime, multi-agent, tool-runtime, permission, context-isolation, orchestrator, subagent]
sources:
  - raw/articles/claude-code-architecture-analysis
  - raw/articles/claude-code-multi-agent-collaboration-allentang-2026-07-22
review_value: 6
review_confidence: 7
---
## Overview
Claude Code 源码拆解（by 无岳，阿里云开发者，2026-04-15）。核心论点：**真正决定 Agent 能不能长期活下去的，不是模型，而是围着模型搭起来的运行时。**   ^[raw/articles/claude-code-architecture-analysis.md]

## 核心 Insight
| # | Insight | 模块 | ^[raw/articles/claude-code-architecture-analysis.md]
|---|---------|------| ^[raw/articles/claude-code-architecture-analysis.md]
| 1 | 启动层决定架构寿命 | 启动链路 | ^[raw/articles/claude-code-architecture-analysis.md]
| 2 | UI 是 runtime 操作台，不是文本展示壳 | REPL | ^[raw/articles/claude-code-architecture-analysis.md]
| 3 | 连续运行是状态机，不是函数调用 | Query Loop | ^[raw/articles/claude-code-architecture-analysis.md]
| 4 | 工具层收敛横切复杂度，是稳定性关键 | Tool Runtime | ^[raw/articles/claude-code-architecture-analysis.md]
| 5 | 权限 = 逻辑授权 + 执行隔离，不是弹窗 | Permission | ^[raw/articles/claude-code-architecture-analysis.md]
| 6 | 多 Agent 核心是任务抽象，不是 prompt 分工 | Task | ^[raw/articles/claude-code-architecture-analysis.md]
| 7 | 平台扩展性 = 外部多样 + 内部收敛 | 扩展层 | ^[raw/articles/claude-code-architecture-analysis.md]

## 子页面
- [[entities/claude-code-architecture-modules|七大模块详解]] — 入口链路、REPL、Query Loop、Tool Runtime、Permission、多Agent、扩展性
- [[entities/claude-code-architecture-analysis|设计原则与对照分析]] — 五条设计原则、Harness 映射、OpenClaw/Hermes 对比

## Related
- [[comparisons/cli-tools-comparison|CLI-Tools 横向对比]]
- [[raw/articles/claude-code-architecture-analysis.md|原始文章存档]]
- [[entities/构建基于多智能体架构的深度思考交易系统.md|构建基于多智能体架构的深度思考交易系统]]
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]]
- [[entities/读完-claude-code-和-openclaw-的-memory-源码我对agent记忆需要向量数据库这件事产生了怀疑|Claude Code vs OpenClaw 记忆系统 — 向量数据库必要性反思]]
- [[concepts/agent-backend-unification|Agent 与后端统一架构]]
- [[concepts/claude-code-deep-architecture-analysis|Claude Code 架构深度分析]]
- [[entities/context-isolation|上下文隔离]]
- [[entities/从多智能体编排到ai自主决策资损防控体系的架构演进|从多智能体编排到AI自主决策：资损防控体系的架构演进]]
- [[entities/agentium-agent-framework|Agentium — 从零实现 Agent 系统的开源框架]]
- [[entities/17-agent-architectures-evolution|17种Agent架构演进：控制流设计的完整演化史]]
- [[entities/owner-worker-verifier-architecture|Owner-Worker-Verifier 架构]]
- [[entities/hermes-agent-kanban-deep-test|Hermes-Agent Kanban 实测 — 商业 CLI 作为上层 Orchestrator]]
- [[concepts/multi-agent-systems|Multi-Agent Systems]]

## 深度分析
Claude Code 架构的核心价值在于将"野生函数"转化为"带完整运行时语义的受控对象"，这意味着工具层不是简单暴露函数调用，而是包含了执行上下文、权限校验、状态管理的完整运行时。^(raw/articles/claude-code-architecture-analysis) ^[raw/articles/claude-code-architecture-analysis.md]^[raw/articles/claude-code-architecture-analysis.md]
六大模块构成一个分层架构：入口链路负责进程初始化与状态分离；REPL 作为 runtime 操作台持续与用户保持交互；Query Loop 以状态机模式驱动连续运行，支持 Compact 压缩与恢复；Tool Runtime 是横切复杂度的收敛点；Permission System 提供完整决策链而非单纯弹窗；Task 抽象为多 Agent 提供了统一任务收回机制。^(raw/articles/claude-code-architecture-analysis) ^[raw/articles/claude-code-architecture-analysis.md]^[raw/articles/claude-code-architecture-analysis.md]
这个架构最值得注意的设计原则是"外部动态、内部收敛"——MCP/Skills/Plugins 可以动态扩展，但内部模块边界清晰、依赖关系稳定。这直接回答了为什么很多 Agent 系统在 Demo 阶段很强但一旦工具变多就开始"变形"：它们缺少一个真正承接复杂度的运行时架构。^(raw/articles/claude-code-architecture-analysis) ^[raw/articles/claude-code-architecture-analysis.md]^[raw/articles/claude-code-architecture-analysis.md]

### 补充：多 Agent 协作源码深析 — 上下文隔离即核心机制（AllenTang 2026-07-22）

AllenTang 对 Claude Code 多 Agent 模块的源码级拆解，核心命题：**多 Agent 协作切分的不是能力，而是上下文**。子 Agent 不比主 Agent 多懂什么，它唯一强的地方是"桌子干净"（全新的上下文窗口）。^[raw/articles/claude-code-multi-agent-collaboration-allentang-2026-07-22.md]

#### Orchestrator / Subagent 角色划分

| 角色 | 工具集 | 职责 |
|------|--------|------|
| **Orchestrator（编排者）** | task, read_file, list_dir（只读+派发） | 规划、分解、汇总，不直接执行会改环境的工具 |
| **Subagent（子 Agent）** | bash, write_file, read_file（执行权限） | 做具体操作，默认不能派出子 Agent |

编排者和执行者的工具集**互补、不重叠**，防止编排者绕过规划直接改文件，也防止执行者自行派发无限递归。^[raw/articles/claude-code-multi-agent-collaboration-allentang-2026-07-22.md]

#### Task 工具机制

- 形态：普通工具调用（继承 AgentTool 接口）
- 输入：{ subagent_type, prompt } — 子 Agent 类型 + 任务描述（唯一输入）
- 输出：子 Agent 的最后一条消息
- 执行：runSubagent() — 全新的 Agent Loop，没有任何特殊执行路径

子 Agent 定义是一个 markdown 文件（.claude/agents/*.md），三个字段各有读者：^[raw/articles/claude-code-architecture-analysis.md]


| 字段 | 读者 | 用途 |
|------|------|------|
| name | 系统 | 标识 |
| description | **主 Agent 的模型** | 路由决策依据（"这事派给谁"） |
| tools | 权限系统 | 工具白名单 |
| 正文 | 子 Agent 自己 | system prompt |

**description 即接口** — 写得含糊就会派错活。^[raw/articles/claude-code-multi-agent-collaboration-allentang-2026-07-22.md]

#### 上下文隔离设计

子 Agent 启动时拿到**全新上下文**：^[raw/articles/claude-code-multi-agent-collaboration-allentang-2026-07-22.md]

- 独立 system prompt
- 独立工具定义
- 对话历史 = 父 Agent 写的任务描述（一条消息）
- 父 Agent 的原始对话历史一概不可见

完成后，进入父 Agent 上下文的只有**子 Agent 的最后一条消息**。中间过程一概不返回。^[raw/articles/claude-code-architecture-analysis.md]


"新桌子"隐喻：上下文窗口是 Agent 的桌子。子 Agent 把一摞资料挪到自己桌上看完，交回一页结论。父 Agent 桌上多一页纸，少一万页资料。^[raw/articles/claude-code-multi-agent-collaboration-allentang-2026-07-22.md]

#### 核心设计决策

1. **权限交集**：子 Agent 的工具集 = 定义声明 ∩ 父 Agent 拥有的工具，默认去掉 task 工具
2. **预算只减不增**：子 Agent 的 token 预算从父 Agent 剩余预算切分
3. **Agent Loop 复用**：子 Agent 跑标准 Agent Loop，无特殊执行路径
4. **模型即调度器**：并行/串行的判断由模型决定 — 没有写调度器
5. **Loose coupling**：父 Agent 和子 Agent 通过工具调用结果解耦，父不知道子内部的执行过程^[raw/articles/claude-code-multi-agent-collaboration-allentang-2026-07-22.md]

## 实践启示
1. **运行时比模型更重要**：在评估或构建 Agent 系统时，首先看工具层的完整性和运行时语义的严谨程度，而不是模型本身的能力。 ^[raw/articles/claude-code-architecture-analysis.md]^[raw/articles/claude-code-architecture-analysis.md]
2. **状态机优于函数调用**：连续运行场景下，用状态机模式管理 Query Loop 比简单的函数递归更稳定，也更容易支持中断恢复。 ^[raw/articles/claude-code-architecture-analysis.md]^[raw/articles/claude-code-architecture-analysis.md]
3. **权限设计即架构决策**：Permission System 不只是安全特性，它是架构的一部分——自动判定 + 交互确认 + 沙箱执行构成完整决策链。 ^[raw/articles/claude-code-architecture-analysis.md]^[raw/articles/claude-code-architecture-analysis.md]
4. **多 Agent 的核心是任务抽象**：不是 prompt 分工，而是能否把分出去的任务执行结果统一收回并协调。 ^[raw/articles/claude-code-architecture-analysis.md]^[raw/articles/claude-code-architecture-analysis.md]
5. **扩展性需要内部收敛来保障**：外部插件再多，内部模块边界必须清晰；否则系统随复杂度增长必然腐化。 ^[raw/articles/claude-code-architecture-analysis.md]^[raw/articles/claude-code-architecture-analysis.md]


## 架构图
→ [C4 架构图](assets/c4/claude-code-architecture-c4.html)

## 相关实体
- [[entities/claude-code-7-layer-memory-architecture|claude-code-7-layer-memory-architecture]]

- [[entities/agent-era-architect-skills-guide|Agent 时代架构师技能指南]] ^[raw/articles/claude-code-architecture-analysis.md]
- [[concepts/data-agent-platform-architecture]]
- [[moc/wiki-master-map|MOC]]