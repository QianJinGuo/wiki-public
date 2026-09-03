---
title: "OpenRath：以 Session 为核心的多 Agent 运行时状态系统（清华）"
created: 2026-06-28
updated: 2026-07-02
type: entity
tags: [agent-runtime, session, state-management, multi-agent, tsinghua, fork-merge, provenance, evidence-protocol, pytorch-analogy]
sources: [raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026]
confidence: 0.8
provenance_state: extracted
---

# OpenRath：以 Session 为核心的多 Agent 运行时状态系统（清华）

清华开源的多 Agent 运行时系统，核心创新是将分散在聊天记录、工具日志、沙箱、记忆和分支里的运行状态，统一收进一个可分叉、可合并、可回放的 **Session** 对象。^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## 核心问题

多 Agent 系统状态散落：聊天记录在消息列表里，工具调用在日志里，沙箱位置在执行器里，记忆在数据库里，分支关系在控制器代码里。没有共同对象把它们串起来。^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## Session：不只是聊天记录

Session 是程序真正传来传去的运行时值，包含：对话块、沙箱位置、分支血缘、token 用量、工具证据、记忆交互。可以理解为一次 Agent 工作的"运行账本"。 ^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

| 记录类型 | 主要读者 | 主要内容 |
|----------|----------|----------|
| 图检查点 | 调度器 | 执行到哪一步 |
| Trace span | 观察者 | 运行时观察到什么 |
| **Session** | **Agent 程序** | **可传递的工作状态** |

## PyTorch 类比

| PyTorch | OpenRath | 含义 |
|---------|----------|------| ^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]
| Tensor | Session | 流动的运行时值 |
| Device | Sandbox | 工具执行的位置 |
| Parameter | Memory | 持久状态平面 |
| Module | Workflow | 可组合工作流 |

核心对象：Session, Agent, Tool, Sandbox, Memory, Workflow, Selector。每个对象"不拥有"什么很重要——Agent 不拥有完整会话图，血缘属于 Session；Tool 不拥有执行位置。^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## Session 生命周期

创建→放置→变换→分叉→合并→持久化→释放。

fork 复制当前状态并保留父子关系；detach 切断父血缘；merge 检查沙箱兼容性。分支是否能合并，既看内容，也看执行环境。^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## 工具执行边界

模型看到 schema → 副作用通过 Sandbox 落地 → 结果作为 Session 证据返回。从"模型说它做过什么"变成"运行时记录它实际做过什么"。

## Claim-to-evidence 协议

每个结论挂证据等级：supported、partially supported、prerequisite-supported、evidence-gated。evidence packet 包含命令、manifest、源代码、Session JSONL、生成产物和证明说明。 ^[raw/articles/openrath-session-centered-agent-runtime-tsinghua-2026.md]

## 与现有框架的区别

| 框架 | 关注点 |
|------|--------|
| AutoGen | 多 Agent 对话可编程 |
| LangGraph | 图状态解决持久执行 |
| OpenAI Agents SDK | agents、handoff、guardrails 产品化 |
| MCP | 工具接入协议边界 |
| **OpenRath** | **运行时状态边界** |

## 局限

- 没有广泛 benchmark 对比
- Memory 质量未验证
- 安全属性未证明（间接 prompt injection 等）
- 云端后端未验证

## 相关实体

- [[entities/langchain-harrison-chase-sandbox-architecture|LangChain Sandbox Architecture]] — 另一种 Agent 运行时设计
- [[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents]] — 类似的 sandbox 架构
- [[concepts/harness-engineering-framework|Harness Engineering]] — Agent 运行时的更广泛框架
