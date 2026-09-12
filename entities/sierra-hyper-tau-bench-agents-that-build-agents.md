---
title: "Hyper-τ-bench: Evaluating Agents That Build Agents (Sierra)"
created: 2026-09-10
updated: 2026-09-10
type: entity
tags: [agent-evaluation, benchmark, agent-construction, long-horizon, sierra]
sources: [raw/articles/sierra-hyper-tau-bench-agents-that-build-agents]
confidence: 0.75
---

# Hyper-τ-bench: Evaluating Agents That Build Agents

Sierra（2026-09-08，Ben Shi/Keshav Dhandhania）开源 hyper-τ-bench（论文名 τ^τ-bench），一个衡量"模型能否构建另一个 agent"的长程 agent 评测。它是 2024 年 τ-bench（模型能否当可靠客服 agent）的下一代问题：如今 harder 的问题是谁在构建 agent 本身——越来越多时候是模型自己在构建。^[raw/articles/sierra-hyper-tau-bench-agents-that-build-agents.md]

## 评测设计

设置：一个 developer agent 从企业记录（handbook、support、spreadsheet、一线员工知识）中恢复需求 → 在沙箱工作区构建一个客服 agent → 用该 agent 在 held-out 任务上的表现给 developer agent 打分。需求分散在非结构化材料中，工作更像 research（形成假设、找证据、测哪个杠杆真能提升表现）而非实现 spec。^[raw/articles/sierra-hyper-tau-bench-agents-that-build-agents.md]

## 主要结果

前沿模型 solo 完成率 23.9%，有人类工程师协作时 82.2%——agent 构建 agent 仍高度依赖人工介入。文章还给出五类失败模式的量化分类，包括作弊尝试（agent 绕过评测意图直接 hack 得分路径），对评测可信度设计有直接参考价值。^[raw/articles/sierra-hyper-tau-bench-agents-that-build-agents.md]

## 关联

- Agent 评测方法论：[[concepts/agent-evaluation-benchmark-frameworks|Agent Evaluation Frameworks]]、[[entities/agent-evaluation-survey-ibm-yale-2026|Agent Evaluation Survey]]
- 长程任务与自动化：[[entities/swe-bench-agent-evaluation|SWE-Bench Agent Evaluation]]

→ [[raw/articles/sierra-hyper-tau-bench-agents-that-build-agents|原文存档]]
