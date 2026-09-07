---
title: "Harness 即后端 当Agent基础设施消解于统一原语 unknown"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-01-Harness-即后端-当Agent基础设施消解于统一原语-unknown]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/2026-05-01-Harness-即后端-当Agent基础设施消解于统一原语-unknown.md|原文存档]]

sha256: 4ec1e4f25ca79edacf7a37232018b6e1669c0021c203c1289388b12f13e3a540 ^[raw/articles/2026-05-01-Harness-即后端-当Agent基础设施消解于统一原语-unknown.md]

## 摘要

文章指出 AI 基础设施的核心问题已从模型选型转向 Agent Harness（编排基础设施）的构建，而当前 Harness 与传统 Backend 被当作两个独立系统的分层只是过渡形态：Agent 行为的随机性使跨系统调试路径按"Agent 数量的平方乘以后端服务数量"组合级增长，四个 Agent 对五个后端即膨胀为八十条随机路径。作者提出后端可还原为三种基本原语——Worker（连接引擎注册能力的工作进程）、Trigger（声明式触发器）、Function（具稳定标识符的工作单元），Agent 可直接作为 Worker 接入引擎，其工具即函数、记忆即状态、编排即 Trigger 组合，由此 Harness 不再是后端之上的附加层，"Harness 即后端"。统一原语还带来品类折叠（一切答案都是"添加一个 Worker"）与实时发现、实时扩展、实时可观测三大内生产物，并支持沙箱即 Worker 的递归能力。^[raw/articles/2026-05-01-Harness-即后端-当Agent基础设施消解于统一原语-unknown.md]

## 关键要点

- Harness 设计谱系从精简编排（决策权交模型）到重度结构化编排（指令栈、显式交接、确定性流控），本质是模型信任度与逻辑编码强度的取舍
- Agent 调试难度公式：Agent 数量的平方 × 后端服务数量；Agent 随机性是设计本意而非缺陷
- 后端三原语：Worker / Trigger / Function，Function 以稳定标识符（如 orders::validate）寻址，Trigger 把 HTTP、Cron、队列、状态变更等信号声明式绑定到函数
- 品类折叠：队列、沙箱、Cron、可观测性、Agent 全部归结为注册 Function 和 Trigger 的 Worker，语义存在于 Function 而非基础设施
- 统一架构涌现三大实时特性：实时发现（全量函数目录）、实时扩展（运行时注册无需重启）、实时可观测（Trace ID 跨 Worker/语言/队列自动传播），并支持 Worker 创建 Worker 的递归沙箱

## 来源

- 原文: [[raw/articles/2026-05-01-Harness-即后端-当Agent基础设施消解于统一原语-unknown.md|Harness 即后端 当Agent基础设施消解于统一原语 unknown]]
