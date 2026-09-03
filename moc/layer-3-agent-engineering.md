---
title: 第 3 层全库索引：Agent 工程
created: 2026-06-24
updated: 2026-06-24
type: moc
tags: [learning-path, layer-3, agent, engineering]
layer: 3
---

# 第 3 层：Agent 工程 — 全库索引

> 返回 [学习路径总入口](learning-path.md)

---

## 本层导读

第 2 层让你学会「怎么用 LLM」。第 3 层给你「怎么让 LLM 自主行动」：Agent 是什么、Agent Loop、Agent 记忆、规划与推理、工具使用与 MCP。这层是从「用工具」到「造能自主用工具的系统」。

---

## 学习路径

```
chap-13 从对话到 Agent（50min）→ chap-14 Agent 记忆（75min）→ chap-15 规划与推理（50min）→ chap-16 工具与 MCP（50min）→ 🚪 关卡
```

---

## 本层 concepts

### Agent 基础
- [[concepts/ai-agent-patterns|AI Agent Patterns]]
- Agent 循环设计
- [[concepts/autonomous-agent-systems|自主 Agent 系统]]
- [[concepts/agent-as-software-3-0-substrate|Agent 作为 3.0 基质]]

### Agent 记忆
- [[concepts/agent-memory-architecture|Agent 记忆架构]]
- AI Agent 记忆类型
- [[concepts/agent-memory-system-design|Agent 记忆系统设计]]
- [[concepts/agent-memory-lifecycle-philosophies|记忆生命周期哲学]]
- [[concepts/agent-memory-substrate-three-layer|三层记忆基质]]
- [[concepts/episodic-vs-semantic-memory-agent|情景 vs 语义记忆]]
- [[concepts/memory-consolidation-decay|记忆巩固与衰减]]
- [[concepts/working-set-vs-long-term-memory|工作集 vs 长期记忆]]

### 规划与推理
- Agent 规划与推理
- [[concepts/tool-use-reasoning|工具使用推理]]
- Reasoning Models

### 工具与 MCP
- [[concepts/model-context-protocol-mcp|MCP]]
- [[concepts/tool-use-patterns-ai-agents|Tool Use 模式]]
- MCP 协议生态
- 搜索检索
- Agentic RAG

### 认知架构
- 认知架构
- 技能习得
- Computer Use Agent

---

## 本层 entities（精选）

### Agent 架构
- [[entities/17-agent-architectures-evolution|17 种 Agent 架构演进]]
- [[entities/agent-architecture-harness-new-backend|Harness 成为新后端]]
- [[entities/a-missing-layer-in-agentic-systems|Agentic 缺失的一层]]

### 记忆实践
- [[entities/agent-memory-modular-framework|Agent Memory 模块化框架]]
- [[entities/hermes-agent-memory-system-architecture|Hermes 记忆架构]]
- [[entities/agent-memory-engineering-tax-aws-china-2026|Agent 记忆工程税]]
- [[entities/state-of-memory-in-agent-harness-mem0-2026|Mem0 记忆状态]]

### 工具与 MCP
- [[entities/anthropic-12-mcp-production-patterns|12 个 MCP 生产模式]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr|AgentOps on Bedrock]]
- [[entities/agentscope-builder-enterprise-self-evolving-agent-harness|AgentScope]]

### 前沿
- [[entities/acker-agent-evolution-three-routes-convergence|Agent 演进三路线]]
- [[entities/agent-evolution-four-stages-six-dimensions-aliyun|Agent 演进四阶段]]

---

## 本层 raw

- [[raw/articles/你不知道的-agent原理架构与工程实践|Agent 原理架构]]
- [[raw/articles/17-agent-architectures-evolution|17 种 Agent 架构]]
- [[raw/articles/anthropic-12-mcp-production-patterns|12 个 MCP 模式]]
- [[raw/articles/agent-memory-architecture-past-influence-future-ruofei|记忆架构：过去影响未来]]

---

## 🚪 关卡

1. **场景题**：用一句话向产品经理解释「Agent 和 ChatGPT 的区别」。
2. **费曼题**：向 12 岁孩子解释「Agent 的工作记忆和长期记忆有什么不同」。
3. **关联题**：ReAct Loop 和第 10 章的 CoT 有什么联系和区别？
4. **场景题**：Agent 调用工具经常选错，从工具描述、粒度、反馈三方面优化。
5. **关联题**：回顾第 11 章上下文工程。Agent 记忆和上下文工程的「存」策略是什么关系？

---

## 学完这层你应该能

- [ ] 画出 Agent 基本架构（Loop + 记忆 + 工具 + 规划）
- [ ] 解释 ReAct、Plan-Execute、Reflexion 三种循环
- [ ] 说出工作记忆、情景记忆、语义记忆的区别
- [ ] 解释 MCP 解决了什么问题
- [ ] 设计一个简单的 Agent

---

**下一层**：[第 4 层：生态工具](layer-4-ecosystem.md)
