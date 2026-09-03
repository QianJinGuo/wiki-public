---
title: "评估 Harness 设计"
created: 2026-06-11
updated: 2026-08-18
type: concept
tags: [agent, evaluation, harness, architecture, memory]
sources: [raw/articles/harnesseval-evaluation-harness-mirros-jiqizhixin-2026-08-18]
description: "评估 Harness 设计：可复现评测、自动评分、结果验证、基准管线"
---
# 评估 Harness 设计

评估 Harness 设计：可复现评测、自动评分、结果验证、基准管线。本概念页汇聚 wiki 中 15 个相关实体的核心洞察，形成系统化的知识框架。

## 核心定义

评估 Harness（Evaluation Harness）：当被评测的 AI 从单一模型变成完成含上下文管理/工具调用/记忆/任务拆解/执行环境/权限/结果验证的复杂系统时，评测不再是一把固定 rubric 的"尺子"，而是为评测本身建立 harness——从 Metric 到 Harness，从静态 benchmark 到可执行的评测系统。核心：评测也是一个需要理解、规划、调查和验证的过程，将原本隐含在人类专家判断中的过程显式化。^[raw/articles/harnesseval-evaluation-harness-mirros-jiqizhixin-2026-08-18.md]

四阶段工作流（HarnessEval 范式）：**Plan**（先理解案例再决定测什么）→ **Route**（从 Skill Library 选适用技能，记录为何启用/跳过某项检查）→ **Decompose**（把抽象判断拆成目标定位/状态追踪/时序变化/因果顺序/结构保持等可验证子问题，交专门 sub-agent/工具）→ **Verify**（验证各分支证据质量与逻辑关系，输出完整 evidence tree 而非单个标量分数——记录测了什么/为什么测/调什么工具/找到什么证据/如何支持结论）。^[raw/articles/harnesseval-evaluation-harness-mirros-jiqizhixin-2026-08-18.md]

benchmark 核心资产从数据与指标扩展到技能路由/工具调用/证据验证/持续生长的 Skill Library；模型可 test-time scaling，评测器也应投入更多计算深入搜索验证。世界模型是 HarnessEval-w 的第一块试验场（观测质量/状态转移正确性/世界持续性三维），也是迈向 Physical RSI 的关键一步——评测关注什么模型就朝什么优化，可靠 Eval 是 RSI 闭环的关键反馈。^[raw/articles/harnesseval-evaluation-harness-mirros-jiqizhixin-2026-08-18.md]

## 关键维度

<!-- TODO: 补充关键维度分析 -->

## 相关实体

- [[entities/aliyun-cio-ai-rd-efficiency|阿里云CIO：AI产研效能规模化提升实践（抛弃生码率、重构Half-Stack）]]
- [[entities/business-agent-augmentation-layer-practitioner-methodology-20260606|业务 Agent 增强层架构：复用通用 Agent 基座，把业务能力做成可验证增强层]]
- [[entities/harness-engineering-comprehensive-guide-conardli|Harness Engineering 综合性指南]]
- [[entities/impeccable-frontend-design-skill-harness-vibecoder|Impeccable：把 AI 前端设计变成可检查的工作流]]
- [[entities/wiki-evolver|Wiki Evolver]]
- [[entities/agent-development-crawl-walk-run-crewai-iterative|Agent 开发应小步快跑：第一个 Agent 只需做一件事（哪怕很烂）]]
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计与实现：生产级 Agent 系统落地指南]]
- [[entities/agent-harness-observability-production|Agent Harness 可观测性：生产级 AI 项目必须补上的一课]]
- [[entities/agent-memory-architecture-past-influence-future-ruofei|agent memory architecture past influence future ruofei]]
- [[entities/ai-agent-harness-construction-akshay-baoyu|深度拆解：AI 智能体 Harness 的构造【译】]]
- [[entities/ai-friendly-architecture-design-taobao|面向 LLM 的架构设计：什么是真正的 AI Friendly 架构？]]
- [[entities/ai-native-rd-org-design|AI Native 时代研发组织何去何从]]

## 相关概念

<!-- TODO: 补充相关概念 wikilinks -->

## 实践启示

<!-- TODO: 补充实践启示 -->

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
