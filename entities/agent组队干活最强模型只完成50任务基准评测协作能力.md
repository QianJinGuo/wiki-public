---
title: "AgentWorld：多智能体协作评测基准"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [multi-agent, benchmark, collaboration]
sources: [raw/articles/agent组队干活最强模型只完成50任务基准评测协作能力]
confidence: 0.75
reviewed: 2026-10-26
review_verdict: refresh
---

# AgentWorld：多智能体协作评测基准

## 深度分析

OpenAgents+哥大/宾大/首尔大/宾州州立的 AgentWorld 基准（arXiv:2609.31590）：把长时程任务、角色不对称、黑盒交互放进同一协作评测环境。10 个 agent 的 RPG 世界任务（伐木工→木工→法师制作魔法杖），失败点在任意一次交接（交给谁/重复劳动/提前收工）。设计：高层工具屏蔽低层控制干扰、程序化规则判定共同目标、CCE（贡献归因）指标追溯哪些行动与消息对结果有贡献。结果：最强模型只完成约 50% 任务，最高成功率 52%，暴露多智能体协作短板。代码数据开源。 ^[raw/articles/agent组队干活最强模型只完成50任务基准评测协作能力.md]

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]
- [[entities/ai-evals-methodology|AI Evals 方法论]]

→ [[raw/articles/agent组队干活最强模型只完成50任务基准评测协作能力|原文存档]]
