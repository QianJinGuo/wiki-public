---
title: "dig.bench — 文本游戏科学发现智能体基准"
created: 2026-08-19
updated: 2026-09-07
type: entity
tags: [agent-benchmark, scientific-discovery, text-games, agent-evaluation, experimentation, rule-discovery, harness]
sources: [raw/articles/digbench-scientific-discovery-text-games-agent-benchmark]
confidence: 0.78
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# dig.bench — 文本游戏科学发现智能体基准

## 概述

dig.bench 是一个以**科学发现**为目标的智能体基准：70 个交互式文本游戏，**每个游戏衡量智能体能否通过实验发现游戏自身的未知规则**（Discovering unknown rules in text-based games）。配备 21 个公开 API 文档，并提供 Paper / GitHub / Leaderboard。^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]

## 设计要点

- **科学发现的实验能力**：不只测「答对问题」，而测智能体能否主动设计实验、从反馈中归纳未知规则——把 LLM 的科学假设-检验循环压缩进文本游戏环境^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]
- **人类可基准**：每个游戏都至少被一位人类测试者在首次尝试中击败，难度分层——所有游戏都是人类首试可达的^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]
- **得分机制**：胜率 = 每个游戏 runs 均值再跨 tier 的 10 个游戏平均；Tier 划分覆盖从 Basic Harness 到 Agentic Harness（基础模型 vs 带 Agent 框架的完整编排）^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]

## 意义

dig.bench 把「智能体实验/发现能力」从通用 QA 基准中独立出来，与 [[entities/autoresearch-ai-scientific-discovery-l0-l4-challengehub|AutoResearch]] 同属 AI 科学发现评估前沿。其「基础模型 vs Agentic Harness」双层测法为 [[entities/agent-evaluation-systematic-guide-metrics-to-closed-loop|Agent 评测]] 提供了实验驱动的基准范式。

→ [[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark|原文存档]]
