---
title: Claude Code 工具设计演化
created: 2026-05-07
updated: 2026-08-01
type: concept
tags: [claude-code, tool-design, anthropic, agent-harness]
related:
  - [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]]
  - [[entities/anthropic-prompt-caching-claude-code|Prompt Caching 工程实践 — Anthropic Claude Code 经验总结]]
  - [[entities/ai-agent-tool-count-trap|AI Agent工具数量陷阱——5个边界清楚的工具胜过20个模糊工具]]
sources: ['raw/articles/claude-code-tool-design-evolution-anthropic']
confidence: high
---
# Claude Code Tool Design Evolution
> Anthropic's official engineering retrospective on three tool design iterations in Claude Code — revealing that tools must evolve with model capabilities. Core principles: elicitation design, Todo→Task architecture shift, and the "let Claude find its own context" philosophy.
---
## Three Tool Evolution Case Studies
### 1. AskUserQuestion: Three Iterations to Elicitation Design
**Problem**: Claude asking questions in plain text had high user friction — answering felt slow.
**Iterative design process**:
| Iteration | Approach | Why It Failed |
|-----------|----------|--------------|
| #1 | ExitPlanTool + `questions` parameter | Confused Claude — simultaneously planning and being questioned; logic conflicts |
| #2 | Modified output format (bullet + brackets) | Unstable: extra sentences, dropped options, format abandonment |
| #3 | Independent AskUserQuestion tool ✅ | Blocked agent loop until user answered; Claude liked calling it |
**Key principle**: "Even the best designed tool doesn't work if Claude doesn't understand how to call it."
**Final design**: Structured output → modal display → blocked agent loop until user responds.
---
### 2. TodoWrite → Task: From Linear Checklist to Collaboration Graph
**Driving force**: Model capability improvement created new coordination problems.
**Evolution**:
```
TodoWrite (v1)
  - Single agent linear checklist
  - 5-turn system reminders to prevent forgetting
  - Problem: reminders made Claude rigid; subagents couldn't share
Task (v2, replacement)
  - Multi-agent collaboration task graph
  - Supports dependencies
  - Cross-subagent state sharing
  - Dynamic mutation/deletion
```
**Design philosophy shift**:
- Todo = "keep model on track"
- Task = "help agents communicate with each other"
**Implication**: As models improve at using subagents, coordination infrastructure must evolve.
---
### 3. Context Construction: RAG → Grep
**RAG era problems**:
- Pre-indexed vector database; context "pushed" to Claude before each response
- Required preprocessing; fragile across environments
- **Root issue**: Context was given to Claude, not found by Claude
**Grep solution**: Give Claude a search tool and let it build its own context.
**Foundational principle**:
> "The most consequential tools we've built are the ones that let Claude find its own context."
**Evolutionary chain**: Claude can search the web → why not search the codebase? → Grep tool → Claude self-builds context → 7-layer memory architecture (from other sources)
## Meta-Principles for Tool Design
### P1: Tools Must Track Model Capability
> "As model capabilities increase, the tools that your models once needed might now be constraining them."
Periodic review of "is this tool still necessary?" is required. Tool design is not one-time.
### P2: Small, Capability-Aligned Model Sets
Supporting many models with divergent capability profiles forces tool fragmentation. A small, aligned set enables focused tool design.
### P3: Claude Adoption Is a Design Constraint
The best tool architecture fails if the model doesn't understand how to invoke it. "Claude likes calling this tool" is a success metric.
### P4: Context is Better When Self-Built
Push-based context (RAG) works; pull-based context (Grep) scales with model intelligence. The most impactful tools let the model find context, not receive it.
## 工具设计的反模式：Anthropic 踩过的坑
Anthropic 在官方回顾中坦诚分享了三个失败案例，理解这些反模式对所有 Agent 框架设计者都有重要参考价值：

### 反模式一：多参数工具的认知过载
**失败案例**：ExitPlanTool + `questions` 参数
**问题本质**：将两个不同意图（退出计划模式 + 向用户提问）合并到同一个工具中，导致 Claude 在同一轮对话中同时处理"规划执行"和"准备提问"两种状态，逻辑冲突。
**设计原则**：**一个工具一个核心意图**。当工具描述中出现"同时做 X 和 Y"时，应该拆成两个工具。Claude Code 的 AskUserQuestion 最终成为独立工具而非 ExitPlanTool 的参数，这是关键设计决策。

### 反模式二：依赖输出格式约定的脆弱接口
**失败案例**：用bullet + 括号约定来结构化 Claude 的提问内容
**问题本质**：纯文本格式约定没有强制力，Claude 可以轻易添加解释性句子、遗漏选项、或完全放弃格式。这些"软约定"在模型输出不稳定时迅速崩溃。
**设计原则**：**结构化输出必须配合工具 schema 验证**，而非依赖约定。如果需要特定格式，应该用 Tool 的 input_schema 强制约束，而非在描述中约定。

### 反模式三：推送式上下文的脆弱性
**失败案例**：RAG 时代的预索引向量数据库
**问题本质**：预索引是环境敏感的——代码库结构变化时索引失效；预计算 embedding 是计算密集的；最根本的问题是"上下文由系统推送给模型"而非"模型主动获取"，这削弱了模型的主体性。
**设计原则**：**优先设计拉取式工具而非推送式工具**。给模型搜索能力，让模型自己决定需要什么上下文。随着模型能力提升，拉取式设计的边际收益越来越大。
## 工具演进对 Agent 框架设计者的启示
Anthropic 的三个案例揭示了一个核心规律：**工具设计是动态的，必须随模型能力进化**。这意味着：

### 启示一：定期工具审计是必须的
每隔一个模型版本周期，应该重新评估现有工具是否仍然必要、是否成为约束、是否需要合并或拆分。Claude Code 从 TodoWrite 迁移到 Task 就是因为模型使用子 Agent 的能力提升了，原有的线性 checklist 无法支持新的协作模式。

### 启示二：工具的可发现性比功能性更重要
AskUserQuestion 最终成功的关键不是功能强大，而是"Claude 喜欢调用它"。这说明工具设计必须考虑**模型的认知模型**，而非仅考虑功能正确性。工具的名称、描述、调用条件都会影响模型的调用意愿。

### 启示三：工具粒度与模型推理能力匹配
当模型推理能力弱时，工具应该粗粒度、覆盖常见场景（如 RAG 的推送式上下文）；当模型推理能力增强后，工具应该细粒度、给予模型更多选择权（如 Grep 的拉取式搜索）。

### 启示四：协作工具的设计滞后于单 Agent 工具
TodoWrite→Task 的演进说明，**多 Agent 协作工具的设计是独立的工程挑战**，不能简单复用单 Agent 工具的设计经验。Task 工具的依赖图、跨 Agent 状态共享、动态变更能力，都不是原始 TodoWrite 设计时考虑的场景。
## MCP 协议与工具设计的协同演化
Anthropic 在 Claude Code 中对工具设计的思考，与 **Model Context Protocol (MCP)** 的出现形成了有意义的呼应。MCP 作为一种标准化的工具发现与调用协议，其设计哲学与 Claude Code 工具演化的核心原则高度一致。

### MCP 的核心价值：工具的"可发现性"标准化
MCP 解决了工具设计中的一个关键问题：**工具接口的标准化描述**。在 MCP 出现之前，每个 Agent 框架都需要自己定义"工具如何被调用"，导致工具在框架间的迁移成本极高。Claude Code 的 AskUserQuestion 成功的前提是 Claude Code 内部完全理解这个工具的 schema，但如果把这个工具放到另一个 Agent 框架中，调用契约就会失效。

MCP 通过标准化的工具描述格式（input_schema、description、annotations）让工具具备**跨框架可发现性**。这与 Claude Code 工具演化 P3 原则"Claude adoption is a design constraint"形成互补——P3 强调的是模型层面的理解，MCP 强调的是框架层面的互操作性。

### MCP 对 Claude Code 工具设计的验证
Claude Code 的三次迭代（AskUserQuestion）和两次架构迁移（TodoWrite→Task、RAG→Grep）都在验证一个规律：**工具设计必须考虑调用者的认知模型**。MCP 的 annotations 机制（如 `readOnlyHint`、`destructiveHint`）正是这一原则的协议层实现——通过明确的语义标注，帮助模型理解工具的行为边界，而不是让模型从工具描述中推断。

### 工具生命周期管理与 MCP
Claude Code 文档中提到的"定期工具审计"在 MCP 生态中有了更具体的实践方式：
- **工具废弃**：MCP 的 `deprecated` annotation 可以标记即将废弃的工具
- **能力发现**：MCP 的 `capabilities` 字段让 Agent 能动态发现可用工具集
- **版本演进**：MCP 工具的 schema 版本管理与 Claude Code 的"工具必须随模型能力进化"理念对应

> [!参见]
> - [[entities/anthropic-prompt-caching-claude-code|Prompt Caching 工程实践]] — MCP 协议与 Claude Code 的集成方式
> - [[concepts/hermes-agent]] — Hermes Agent 的工具注册与 MCP 协议实现
## Comparison with Related Concepts
| Concept | This Article | Related |
|---------|-------------|---------|
| Context construction | RAG→Grep, Claude self-builds | [[concepts/hermes-agent]] — memory search FTS5/BMS25 |
| Elicitation design | AskUserQuestion 3 iterations | [[concepts/hermes-agent-onboarding]] — tool adoption as UX |
| Tool evolution | Periodic review required | [[concepts/hermes-agent]] — skill self-generation |
| Tool anti-patterns | Multi-intent tools, format conventions, push-based context | [[concepts/harness-engineering-framework]] — Layer 1-3 failures |
## Related Concepts
- [[concepts/coding-agent-architecture|Coding Agent Architecture]] — Claude Code 是编程 Agent 架构的代表性实现
- [[concepts/claude-code-source-leak-lifecycle]] — AskUserQuestion / Task tool locations in source
- [[concepts/harness-engineering-framework]] — Context management in the Harness
- [[concepts/hermes-agent]] — Hermes's parallel architecture for comparison
- [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]]
- [[entities/anthropic-prompt-caching-claude-code|Prompt Caching 工程实践 — Anthropic Claude Code 经验总结]]
## 相关实体
- [[entities/ai-agent-tool-count-trap|AI Agent工具数量陷阱——5个边界清楚的工具胜过20个模糊工具]]

- [[entities/claude-code-agent-view|claude-code-agent-view]]
- [[entities/anthropic-ai-native-startup-handbook|Anthropic发布「AI原生创业公司」手册：涵盖全流程四大核心阶段，一人公司法典来了]]
- [[entities/claude-code-large-codebase-enterprise-deployment|Claude Code 大型代码库最佳实践 — Anthropic 企业级部署指南]]
- [[entities/anthropic-官方技能最佳实践14-个可复用的-agent-skills-设计模式|Anthropic 官方技能最佳实践：14 个可复用的 Agent Skills 设计模式]]
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]]
- [[entities/claude-发布官方报告承认存在-3-处质量退化问题|Claude 发布官方报告，承认存在 3 处质量退化问题]]
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]]

## 新增关联实体
- [[entities/ai-20260506]]
- [[entities/anthropic-ai-windows-mcp-strategy-geekpark-2026]]
- [[entities/anthropic-building-next-claude]]
- [[entities/anthropic-com-research-making-claude-a-chemist]]
- [[entities/anthropic-founder-playbook-ai-native-startup]]

## 关联实体

**上游依赖**:
- [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2]] — 提供基础理论/方法
- [[entities/anthropic-prompt-caching-claude-code]] — 提供基础理论/方法
- [[entities/ai-agent-tool-count-trap]] — 提供基础理论/方法

**下游应用**:
- [[entities/ai-agent-tool-count-trap]] — 具体应用场景
- [[entities/claude-code-agent-view]] — 具体应用场景
- [[entities/anthropic-ai-native-startup-handbook]] — 具体应用场景

**平行协作**:
- [[entities/claude-发布官方报告承认存在-3-处质量退化问题]] — 替代/补充方案
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台]] — 替代/补充方案
- [[entities/ai-20260506]] — 替代/补充方案

## 所属 MOC

- [[moc/claude-code-complete-guide|Claude Code Complete Guide]]
