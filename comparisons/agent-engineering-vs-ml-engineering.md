---
title: "Agent Engineering vs ML Engineering：能力域对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, agent-engineering, ml-engineering, capability, career, skill]
sources: [concepts/agent-engineering-capability-map, entities/agent-engineering-principles-architecture-practice]
---

## 对照表

| 维度 | Agent Engineer | ML Engineer |
|------|---|---|
| 核心产物 | Agent 系统（prompt/harness/memory/tools） | 模型（训练/微调/部署） |
| 关键技能 | 上下文工程 / harness / tool / verifier / observability | 模型架构 / 训练 pipeline / data curation / infra |
| 调优对象 | prompt + working set + memory schema | 模型权重 + 超参 + 数据集 |
| AI 作为工具 | ✅ LLM 是核心运行时 | ✅ LLM 是研究工具 |
| 失败模式 | agent 幻觉 / context 溢出 / tool 错 | underfitting / 数据泄露 / 部署宕机 |
| 迭代速率 | 分钟（改 prompt 立生效） | 小时-天（训练周期） |
| 门槛 | 中（懂 prompt + system design） | 高（需 ML + 数学 + 工程） |
| 当前需求 | 需求爆发中（2026 agent 元年） | 成熟岗位（受 agent 化冲击） |

## 判断

互补不替代：ML engineer 产出可用的模型，agent engineer 产出使用模型的产品系统。趋势：ML engineer 的迭代速度被基础模型供给（开源/API）限制，agent engineer 的迭代速度被设计模式认知限制——后者增长空间更大。2026 后 agent engineering 会成为独立工种。

## 对比方来源

- [[concepts/agent-engineering-capability-map|agent engineering 能力地图]]
- [[entities/agent-engineering-principles-architecture-practice|agent engineering 原理]]
- [[entities/ai-agent-engineer-capability-map|AI agent engineer 能力地图源]]
- [[concepts/production-agent-engineering|production agent engineering]]

## 进一步阅读

- [[concepts/agent-engineering-capability-map]]
- [[entities/agent-engineering-principles-architecture-practice]]
