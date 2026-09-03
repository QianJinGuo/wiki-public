---
title: "Agent Harness 工程范式"
created: 2026-07-02
updated: 2026-08-01
type: concept
tags: [harness-engineering, agent, paradigm, architecture]
provenance_state: inferred
confidence: 0.7
---

# Agent Harness 工程范式

Harness Engineering（驾御工程/运行约束工程）是在 LLM 外部设计一整套机制，让 Agent 的行为能够被约束、被验证、被纠偏的工程方法论。它不是一个框架，而是一组设计原则的集合。

## 核心命题

模型能力不再是瓶颈——harness 才是。同一个模型，不同 harness 下的质量差异可达 4.3×。Harness 决定了模型能力有多少能被有效转化为工程产出。

## 五大支柱

1. **上下文管理**：工作集压缩、分层记忆、主动遗忘、注意力塌缩对抗
2. **工具编排**：MCP 协议、渐进式披露、命令风险分级、生命周期钩子
3. **循环控制**：Inner Loop（执行）/ Outer Loop（规划）、ReAct + Reflexion、Stop Hook 门禁
4. **安全边界**：沙箱隔离、权限分诊（P0-P3）、HITL 爆炸半径控制
5. **人机协作**：Provenance 溯源、Pre-task gating、草稿纸模式

## 从 Demo 到生产的 8 道关卡

记忆分层 → P0-P3 分诊 → Stop Hook 门禁 → 渐进式披露 Skill → 三层路由 → HITL 爆炸半径 → Skill-SubAgent-Workflow-Agent Team 四方图 → Provenance + Pre-task gating

## 关联

- [[entities/harness-paradigm|Harness 范式]]
- [[entities/claude-code-loop-engineering-guide|Claude Code Loop Engineering]]
- [[entities/twelve-agent-design-patterns-yunduojun-datastudio|12 Agent 设计模式]]
- [[entities/agent-loop-engineering-handbook-8-questions-chen-jin-tencent-self-2026|Agent Loop 8 问]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
