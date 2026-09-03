---
title: "单 Agent vs 多 Agent 系统选择"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, single-agent, multi-agent, tradeoff, architecture, selection]
sources: [entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏, entities/claude-code-core-internals, entities/agent-harness-architecture]
---

## 对照表

| 维度 | 单 Agent | 多 Agent |
|------|---|---|
| context 开销 | 1×（独享 context） | N+1×（每位独立 context） |
| 任务并行度 | 串行（subagent 可部分并行） | 天然并行（N worker 同时跑） |
| 编排复杂度 | 低（单一 loop） | 高（orchestrator + 合并） |
| 模型幻觉 | 仅单层 | 可级联放大 |
| 调试/可观测 | 低（单一 trace） | 高（需分布式追踪） |
| 适用场景 | 任务单一 + 长链推理 + 单 domain | 多 domain + 高并行 + 角色差异化 |
| real-world 比例 | 80% 场景 | 20% 但增长快 |

## 判断

默认从单 Agent 起步——多 Agent 的复杂度税太高，不到必要不付。判断标准：若任务能被单一 prompt 描述、且推理链 < 20 步、且不需要并行 → 单 Agent；若需要多种专业能力 + 并行 + 隔离试错 → 多 Agent。Anti-pattern：为了「看起来高级」过早多 Agent 化。

## 对比方来源

- [[concepts/multi-agent-team-coordination|多智能体团队协作]]
- coding agent comparison
- [[concepts/multi-agent-systems|multi-agent systems]]
- [[concepts/orchestrator-worker-architecture|orchestrator-worker]]

## 进一步阅读

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/claude-code-core-internals]]
- [[entities/agent-harness-architecture]]
