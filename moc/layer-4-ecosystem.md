---
title: 第 4 层全库索引：生态工具
created: 2026-06-24
updated: 2026-06-24
type: moc
tags: [learning-path, layer-4, ecosystem, harness, multi-agent]
layer: 4
---

# 第 4 层：生态工具 — 全库索引

> 返回 [学习路径总入口](learning-path.md)

---

## 本层导读

第 3 层让你理解单个 Agent 怎么设计。第 4 层给你「怎么把它做成可靠系统」：Harness 工程、多 Agent 协作、开源生态、评测基准。这层是从「能跑的 Agent」到「能交付的 Agent 系统」。

---

## 学习路径

```
chap-17 Harness 框架（75min）→ chap-18 多 Agent（50min）→ chap-19 开源生态（50min）→ chap-20 评测基准（50min）→ 🚪 关卡
```

---

## 本层 concepts

### Harness 工程
- [[concepts/harness-engineering-framework|Harness 工程框架]]
- [[concepts/harness-loop-architecture|Harness Loop 架构]]
- [[concepts/harness-engineering-7-layers-framework|Harness 七层框架]]
- [[concepts/harness-engineering-paradigm-shift|Harness 范式转移]]
- [[concepts/harness-context-window-management|Harness 上下文管理]]
- [[concepts/harness-long-running-task|Harness 长程任务]]
- [[concepts/harness-tool-design-evolution|Harness 工具设计演进]]
- Harness Gate 评估
- [[concepts/ahe-agentic-harness-engineering|AHE]]
- [[concepts/coding-harness-engineering|Coding Harness]]
- [[concepts/harness-as-product-surface|Harness 作为产品面]]
- [[concepts/managed-agents-architecture|Managed Agents 架构]]

### 多 Agent
- [[concepts/multi-agent-systems|多 Agent 系统]]
- 多 Agent 编排
- [[concepts/multi-agent-collaboration-patterns|多 Agent 协作模式]]
- [[concepts/multi-agent-team-coordination|多 Agent 团队协调]]
- [[concepts/multi-agent-context-isolation|多 Agent 上下文隔离]]
- [[concepts/orchestrator-worker-architecture|Orchestrator-Worker]]
- Subagent 生成模式

### 开源生态
- [[concepts/open-source-ai-ecosystem|开源 AI 生态]]
- 开源 Agent 框架
- 开源 LLM 生态
- 开源模型对比
- Agent 框架对比

### 评测
- Agent 评测基准
- [[concepts/agent-evaluation-benchmark-frameworks|评测框架]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- LLM 基准全景
- 代码生成评测
- AI 测试 QA

### 工程范式
- [[concepts/agentic-engineering-paradigm|agentic engineering 范式]]
- [[concepts/agent-engineering-capability-map|Agent 工程能力图]]
- [[concepts/agent-role-specialization|Agent 角色分工]]
- [[concepts/agent-self-improvement-loops|Agent 自改进循环]]
- [[concepts/specification-driven-agent-development|规格驱动开发]]
- [[concepts/sdd-specification-driven-development-harness|SDD Harness]]
- [[concepts/verifier-driven-development|Verifier 驱动开发]]

---

## 本层 entities（精选）

### Harness 实践
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 生产指南]]
- [[entities/harness-engineering-core-patterns-claude-code|Harness 核心模式]]
- [[entities/harness-engineering-systematic-explainer|Harness 系统解释]]
- [[entities/agentcore-harness|AgentCore Harness]]
- [[entities/agent-harnesses-are-dead-long-live-agent-harnesses|Harness 之死与新生]]

### 多 Agent
- [[entities/agentscope-java-2.0-enterprise-distributed-harness|AgentScope Java]]
- [[entities/agent-room-emergent-collaboration-multi-agent-decision|Agent Room 协作]]
- [[entities/openclaw-multi-agent-team-practice|OpenClaw 多 Agent]]

### 开源框架
- [[entities/agent-framework-owl-principles|OWL 框架]]
- [[entities/agentium-agent-framework|Agentium]]
- [[entities/agentrun-multi-agent-a2a-alibaba-cloud|AgentRun A2A]]

### 评测
- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation|YAML 驱动评测]]
- [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|AgentEvalKit]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr|AgentOps]]
- [[entities/programbench-swe-agent-benchmark|SWE-Bench]]

---

## 本层 raw

- [[raw/articles/harness-engineering-systematic-explainer|Harness 系统解释]]
- [[concepts/harness-engineering-framework|Harness 七层]]
- [[raw/articles/openclaw-multi-agent-team-practice|OpenClaw 多 Agent]]
- [[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation|YAML 评测]]

---

## 🚪 关卡

1. **场景题**：用一句话向技术负责人解释「Harness 解决了什么问题」。
2. **费曼题**：向 12 岁孩子解释「为什么要多个 Agent 合作」。
3. **关联题**：Harness 的「verifier」和第 15 章的 Reflexion 有什么联系？
4. **场景题**：选 Agent 框架，从生态、学习曲线、生产就绪三方面比较 LangGraph 和 CrewAI。
5. **关联题**：回顾第 1 章 vibe coding 崩盘。Harness 怎么解决那个崩盘？

---

## 学完这层你应该能

- [ ] 解释 Harness 是什么、为什么需要它
- [ ] 说出多 Agent 协作的 2-3 种模式
- [ ] 列出 3 个主流开源 Agent 框架及特点
- [ ] 设计一个简单的 Agent 评测方案
- [ ] 理解 verifier 在 Harness 里的角色

---

**下一层**：[第 5 层：生产安全](layer-5-production-security.md)
