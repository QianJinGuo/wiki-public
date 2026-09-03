---
title: "Hermes Cron vs GitHub Actions：自动化对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, hermes, cron, github-actions, automation, scheduling]
sources: [entities/hermes-agent, entities/hermes-agent-self-evolving]
---

## 对照表

| 维度 | Hermes Cron | GitHub Actions |
|------|---|---|
| 触发单位 | Hermes Agent 执行 cron prompt | YAML workflow on push/schedule/pr |
| 有状态 | ✅ 跨 tick，Memory 持久 | ❌ 每次全新环境 |
| 技能支持 | ✅ 加载 SKILL.md 作 context | ❌ 纯 YAML steps，无 LLM 理解 |
| LLM 驱动 | ✅ prompt 可含推理/筛选 | ❌ 确定性 shell 命令 |
| 交付方式 | Telegram/Discord/Slack 等 | Action 日志 + PR/Issue 模板 |
| 环境一致性 | 本机 [本地运行时路径已隐藏] 持久 | ephemeral runner，每次重装 |
| 最佳用途 | 知识库 cron / RSS 监控 / 日报 | CI/CD / 测试 / 容器部署 |

## 判断

两者互补不替代：GitHub Actions 是「确定性 CI/CD 流水线」，Hermes Cron 是「LLM 驱动的智能调度」。前者跑测试/部署，后者跑「每天看 RSS 筛选 + 总结 + 发送」这种需要语义理解的任务。可组合：Hermes Cron webhook 触发 GitHub Actions。

## 对比方来源

- [[entities/hermes-agent|Hermes Agent]]
- [[entities/hermes-agent-self-evolving|Hermes Agent 自进化]]
- workflow automation AI
- [[concepts/long-running-agent-architecture|long-running agent 架构]]

## 进一步阅读

- [[entities/hermes-agent]]
- [[entities/hermes-agent-self-evolving]]
