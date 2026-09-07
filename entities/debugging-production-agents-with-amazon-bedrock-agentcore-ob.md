---
title: "Debugging production agents with Amazon Bedrock AgentCore Observability"
created: 2026-08-01
updated: 2026-09-07
type: entity
tags: ['raw', 'article']
sources: [raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md|原文存档]]

sha256: 4fa3868058e181c2bdc3b17a0ec7f4748ad445e632e19f5bfe4fb593f44a0d15 ^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md]

## 摘要

AWS 机器学习博客介绍如何用 Amazon Bedrock AgentCore Observability 调试生产环境 Agent。生产 Agent 常常"静默失败"——返回看似合理但错误的答案、陷入无限推理循环、选错工具，且不触发任何报错，标准日志和指标无法捕捉决策过程。文章将生产问题分为质量（幻觉、事实错误、在多 Agent 系统中传播）、可靠性（工具调用失败、上下文丢失）、效率（高延迟、token 浪费）三类，并给出三层可观测性工具：CloudWatch 仪表盘（系统级）、OpenTelemetry 分布式 traces（执行级，遵循 OTEL 协议，可导出到 Datadog/Grafana/Elastic）、关键指标（性能 p50/p95/p99 延迟、资源、按类型细分的错误率）^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md]

文章用两个实战场景演示结构化排查流程：无限循环案例中，仪表盘显示消耗 266.9K token 但错误率 0%，单个 trace 含 177 个 span、平均延迟 85,590ms（正常 1–5 秒），根因是系统提示词要求"never give up"却没有终止条件，修复方式是显式终止条件、每会话 token 上限（建议 5,000–10,000）和 10–15 步推理硬限制。工具调用失败案例覆盖五类错误（401/403/400/404/500），用 Logs Insights 查询按工具和状态码聚合定位，并给出 Gateway service role 权限修复等方案。核心结论：结构化可观测性把数小时的猜测变成几分钟的定向排查；系列 Part 2 讲性能优化与内存管理 ^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md]

## 关键要点

- 三层观测工具：GenAI Observability 仪表盘（会话量、延迟、token 用量、错误率）、OpenTelemetry traces（bedrock-agentcore CloudWatch 命名空间）、Logs Insights 查询（每个查询约 2–3 分钟）。
- 无限循环的识别信号：高 token 消耗 + 低错误率；案例中 86 次重复调用 calculate_percentage（输入几乎相同）却永远得不到 25.00% 的精确值。
- 无限循环三大根因：提示词缺少终止条件、框架缺少循环检测（应 3 次相同动作后强制终止）、工具选择错误（描述需含明确用法示例）。
- 工具调用失败五类错误对应不同修复：401 凭证（用 Secrets Manager 自动轮换）、403 IAM 授权（补 Gateway service role 策略）、400 校验（对齐 schema）、404 资源、500 工具自身故障；任一工具错误率超 5% 应立即排查。
- 从被动调试到主动监控：把诊断查询转成 metric filter + CloudWatch 告警与仪表盘，并用 AgentCore Evaluators 对会话做持续自动化质量评分。

## 来源

- 原文: [[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md|Debugging production agents with Amazon Bedrock AgentCore Observability]]
