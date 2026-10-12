---
title: "ATLAS: Evaluating Agents on Search-Intensive Tasks"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [agent, benchmark, search, evaluation, web-search]
sources: [raw/articles/atlas-agentic-search-benchmark-exa]
confidence: 0.75
---

# ATLAS: Evaluating Agents on Search-Intensive Tasks

## 概述

Exa 于 2026-10-08 发布 ATLAS（Evaluating Agents on Search-Intensive Tasks），一个面向 agentic search 的基准：以真实搜索需求为锚点的 query 配对经过验证的 golden answers，配自动化管线随 Web 与模型演进持续刷新。核心度量是三档 F1（precision/recall 谐和）：Discovery F1（实体）、Row F1（整行正确才算对，headline 指标）、Item F1（单元格级，覆盖 discovery 与 enrichment）。^[raw/articles/atlas-agentic-search-benchmark-exa.md]

## 核心发现

- **穷尽式搜索远未解决**：max-effort search agent 系统性优于低算力版本，但每任务成本低于 $1 的 agent 没有一个 Row F1 超过 0.5。^[raw/articles/atlas-agentic-search-benchmark-exa.md]
- **头部 agent 也漏 1/3**：即使最贵的 search agent 仍 miss 约 1/3 golden results——对完整性敏感的场景（如广度信息收集）存在显著改进空间。^[raw/articles/atlas-agentic-search-benchmark-exa.md]
- **固定 model harness 后搜索后端是变量**：Exa 定义了 cost-performance Pareto 前沿，不同 search backend 间分数差达 16%。^[raw/articles/atlas-agentic-search-benchmark-exa.md]

## 定位

ATLAS 的叙事转折是"agentic search 的瓶颈已从智能转移到搜索基础设施"：早期 agent 卡在长上下文管理与跨文档推理，如今 frontier model 已能处理，瓶颈转向搜索栈本身（backend 选择、成本、完整性）。基准引用系把自身置于 WANDR、BrowseComp、WideSearch、DeepSearchQA 等同类系谱中，差异点是行级完整性 F1 + 持续刷新的 golden answers。^[raw/articles/atlas-agentic-search-benchmark-exa.md]

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]
- [[entities/anysearch-agent-search-engine-product-hunt-2026|AnySearch 搜索 agent]]
- [[entities/arbor-hypothesis-tree-research-agent-2026|Arbor 研究假设树 agent]]
- [[entities/anthropic-dynamic-workflows-ultracode-deep-research-lyuyuebannzi|Deep Research harness 实践]]

→ [[raw/articles/atlas-agentic-search-benchmark-exa|原文存档]]
