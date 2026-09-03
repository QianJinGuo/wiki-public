---
title: "Orchestrator-Worker vs DAG 多智能体架构"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, orchestrator, worker, dag, multi-agent, architecture]
sources: [entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏, concepts/orchestrator-worker-architecture]
---

## 对照表

| 维度 | Orchestrator-Worker | DAG-based |
|------|---|---|
| 调度方式 | 集中式：orchestrator 统一派活 | 图拓扑：节点独立消费上游输出 |
| 状态管理 | orchestrator 是唯一 state holder | 每个节点分布式状态 |
| 故障传播 | orchestrator 单点故障 | DAG 中一个节点故障可重试分支 |
| 编排复杂度 | 中（orchestrator 决策逻辑决定） | 高（需要 DAG compiler/scheduler） |
| 适用规模 | 3-10 subagent | 10-100+ 节点 |
| 实现代表 | OpenClaw, AutoGen | LangGraph, CrewAI |
| 调试难度 | 低（所有调度在一处 log） | 高（分布式追踪必须） |

## 判断

Orchestrator-Worker 适合 3-10 subagent 的中等规模任务，调试和容错都简单；DAG-based 适合 10+ 节点的大型工作流，能扩展但运维门槛高。多数项目从 OW 起步，规模到了再迁 DAG。

## 对比方来源

- [[concepts/orchestrator-worker-architecture|Orchestrator-Worker 架构]]
- multi-agent orchestration
- [[concepts/multi-agent-team-coordination|多智能体团队协作]]
- [[concepts/agent-orchestration-patterns|agent orchestration 模式]]

## 进一步阅读

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[concepts/orchestrator-worker-architecture]]
