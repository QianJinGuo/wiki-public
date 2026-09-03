---
title: "Proactive Memory Agent — Meta AI (Wu et al. 2026)"
created: 2026-07-14
updated: 2026-08-01
type: entity
tags: [paper, memory, agent, proactive-memory, long-horizon, meta-ai, arxiv]
sources: [raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026]
confidence: 0.8
provenance_state: extracted
---

# Proactive Memory Agent — Meta AI (Wu et al. 2026)

> **背景**：本篇整理自 Meta AI 的论文 "Remember When It Matters: Proactive Memory Agent for Long-Horizon Agents"（arXiv:2607.08716），提出了一种独立运行的记忆 Agent 架构，通过在长程任务中实施选择性干预来对抗「行为状态衰减」（behavioral state decay）问题。^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]

## 核心发现：Behavioral State Decay

长程 Agent 任务中存在一种隐藏失效模式——随着轨迹（trajectory）的持续增长，任务需求、环境事实、先前的尝试记录、诊断结论以及未完成的子目标等信息会逐渐被埋没在上下文窗口中，最终无法影响决策。论文将此称为 **behavioral state decay**（行为状态衰减）。^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]

这与 [[concepts/memory-consolidation-decay|记忆衰减]] 的概念在认知层面有相似性，但在 Agent 系统中表现为信息在上下文窗口中的「被动沉没」而非主动遗忘。^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]


## 架构设计：双阶段记忆干预

论文提出了一个与 Action Agent 并行运行的独立 Memory Agent，核心架构分为两个阶段：^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]

### Phase 1：记忆管理（Memory Management）
Memory Agent 从 Action Agent 的最近轨迹中提取关键状态信息，更新结构化记忆库（Structured Memory Bank）。此过程与 Action Agent 的动作选择完全解耦。^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]


### Phase 2：干预选择（Intervention Selection）
Memory Agent 判断是否需要注入记忆驱动的提醒（memory-grounded reminder）到 Action Agent 的上下文中。它可以选择：^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]

- **注入提醒**：当检测到行为状态衰减已影响决策质量时
- **保持静默**：当前 Agent 能自行维持有效决策状态

## 选择性干预 > 被动检索

这是一项与现有 [[concepts/working-set-vs-long-term-memory|工作集 vs 长程记忆]] 讨论密切相关的关键发现：^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]

论文通过消融实验比较了多种记忆策略：

| 策略 | 效果 |
|------|------|
| **选择性干预**（论文方案） | **最优** —— 只在必要时注入提醒 |
| 被动记忆暴露（passive bank exposure） | 次优 —— 信息被淹没 |
| 始终注入（always-on injection） | 信息过载，反而降低性能 |
| 顾问式指导（advisor-only guidance） | 不如直接干预有效 |
| 通用检索（general retrieval） | 无针对性，效果最差 |

## 与现有记忆架构的关系

该论文的工作与 wiki 中已有的大量记忆系统研究形成互补：^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]


- [[entities/agent-memory-modular-framework|AGENT MEMORY MODULAR FRAMEWORK]] 和 [[entities/agent-memory-architecture|Agent Memory Architecture]] 主要关注记忆存储的层次化结构，而这篇论文聚焦于记忆的 **主动干预时机**；
- [[entities/hermes-agent-memory-system|Hermes Agent Memory System]] 的三层架构侧重持久化存储，论文的贡献在于何时以及如何 **注入记忆**；
- [[concepts/memory-source-provenance|Memory Source Provenance]] 关注记忆的来源可信度，这篇论文关注的是注入的 **时机选择**；
- [[entities/agent-memory-injection-5-dimensions-4-papers-agent-shouji-2026|Agent Memory Injection]] 从注入维度角度探讨了相似主题。

## 开放权重训练

作为开放权重记忆策略的初步探索，论文在 SETA 基准上使用 SFT 和 GRPO 训练 Qwen3.5-27B，验证了：^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]
- GRPO 训练提升了验证奖励
- 训练后的记忆策略部分迁移到了 Terminal-Bench 任务

## 评价与局限

该论文为长程 Agent 任务中的记忆管理提供了一个清晰的问题定义（behavioral state decay）和实用的解决方案（双阶段记忆干预）。其 Plug-and-Play 设计使其可直接用于现有 Agent Harness。主要局限包括：^[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026.md]

- 仅在两个基准上评估，泛化性待验证
- 开放权重记忆策略的迁移能力有限（"partial transfer"）
- 未与 [[concepts/context-management-agent-systems|Context Management]] 系统做详细对比

## 相关实体

- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL Frameworks Long-Horizon Wolfe 2026]]
- [[entities/roadmapbench-long-horizon-agentic-software-development-benchmark|RoadmapBench Long-Horizon Benchmark]]
- [[entities/memory-agent-systems-cobanov|Memory Agent Systems Cobanov]]
- [[entities/agent-memory-storage-six-schools-wiki-compile-vs-raw-data-debate|Agent Memory Storage Six Schools]]
- [[entities/se-ga-memory-augmented-self-evolution-gui-agents|Self-Evolution GUI Agents Memory]]
- [[entities/state-of-memory-in-agent-harness-mem0-2026|State of Memory in Agent Harness Mem0 2026]]

→ [[raw/articles/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026|原文存档]]
