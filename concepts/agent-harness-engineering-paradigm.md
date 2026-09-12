---
title: "Agent Harness 工程范式"
created: 2026-07-02
updated: 2026-09-10
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

- 母体实体：[[entities/agent-harness-evolution-from-llm-call-to-harness-tencent-2026]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agent-harness-architecture-deep-dive-aksahy]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-之后-状态边界与失败闭环-ruofei]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agent-harness-architecture-design-production-guide]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-engineering-14-step-roadmap]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-engineered-business-agent-evaluation-aliyun-boyu]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agent-harness-auto-repair-debug-184]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agent-harness-12-components-7-decisions]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/数据研发-multi-agent-harness-工程实践]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agent-harness-architecture]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/taobao-live-anchor-agent-harness-engineering-2026]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agent-harness-production]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-handbook-tencent-behavior-level-manual-2026]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/code-as-agent-harness-survey]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-production-agent-engineering-deficit]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/harness-skill-engineering-alibaba-practice]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/wangyunhe-harness-optimization-agentsoul]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/better-harness-eval-trace-methodology]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agent-harness-skill-system-practical-guide]]（嵌入近邻锚点，提案卡 #12 批1）

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
