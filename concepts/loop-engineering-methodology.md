---
title: "Loop Engineering 方法论"
created: 2026-07-02
updated: 2026-09-10
type: concept
tags: [loop-engineering, agent, methodology, ai-coding]
provenance_state: inferred
confidence: 0.7
---

# Loop Engineering 方法论

Loop Engineering 是 Agent 工程的核心执行模式：将任务拆解为可迭代的循环（Loop），每个循环包含感知-决策-执行-验证四阶段。它从 Vibe Coding 的"一次性生成"进化为"持续迭代优化"。

## 三种循环形态

1. **单 Agent 循环**：While 循环 + ReAct，适合单任务深度执行
2. **舰队循环（Fleet Loop）**：多 Agent 并行 + 编排者，适合复杂任务分解
3. **开闭环（Open/Closed Loop）**：开环=人类在环中（HITL），闭环=自主运行

## Inner Loop vs Outer Loop

- **Inner Loop**：Agent 内部的感知-决策-执行-验证微循环，秒级
- **Outer Loop**：任务级的目标-规划-执行-反思宏循环，分钟到小时级

## 关联

- [[entities/claude-code-loop-engineering-guide|Claude Code Loop Engineering]]
- [[entities/loop-engineering-claude-code-sustainable-workflow|Loop Engineering 可持续工作流]]
- [[entities/一文看懂-ai-编程智能体工程化新范式loop-engineering|Loop Engineering 新范式]]
- [[entities/别只盯着模型agent-真正的护城河是这四层循环|Agent 的四层循环护城河]]

- 母体实体：[[entities/loop-engineering-concept-analysis-feixue-ali-2026]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/loop-engineering应该赞成还是反对]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/loop-engineering-tsinghua-2026]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/2026-06-18-Loop-Engineering-概念解析-思考与实践-阿里技术]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/dittos-loop-codex-product-pm-zhongshiliu-2026]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/ai-native-development-workflow]]（嵌入近邻锚点，提案卡 #12 批1）
- 母体实体：[[entities/agentic-loop-engineering-handbook-empirical-framework]]（嵌入近邻锚点，提案卡 #12 批1）

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]

## 相关 MOC

- [[moc/loop-engineering|循环工程主题地图]] — 本概念的主题 hub，含簇内实体与学习路径
