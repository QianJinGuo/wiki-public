---
title: "Claude Code vs Kimi vs MiniMax Agent Tea 科技充电站"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-21-Claude-Code-vs-Kimi-vs-MiniMax-Agent-Tea-科技充电站]
provenance_state: extracted
---

> -> [[raw/articles/2026-05-21-Claude-Code-vs-Kimi-vs-MiniMax-Agent-Tea-科技充电站.md|原文存档]]

sha256: 1fbf52be40c316d259b5778950cdc8d41e0aa1138af6685f6fbcf7bf8bfc75d1 ^[raw/articles/2026-05-21-Claude-Code-vs-Kimi-vs-MiniMax-Agent-Tea-科技充电站.md]

## 摘要

科技充电站（作者行小招）对比三家 Agent Teams 实现的文章，核心论点是 "Agent = Model + Harness"——真正有价值的 Agent Teams 拼的是外面那层 harness（工具权限、上下文隔离、任务队列、状态机、审批门禁、日志观测、成本控制），没有 harness 的多 agent 只是"一群很贵的复读机"，容易共识塌陷、互相附和一起错。文章提出合格的 Agent Teams 需要 6 层控制：调度流（状态机、任务队列、Ralph Loop）、工具层（MCP、沙盒、权限分级）、记忆层（文件即内存、摘要落盘）、门控层（Plan-Execute-Verify 分阶段）、安全层（approval gate、凭据隔离）、观测层（telemetry、成本统计）^[raw/articles/2026-05-21-Claude-Code-vs-Kimi-vs-MiniMax-Agent-Tea-科技充电站.md]

三家路线各异：Claude Code Agent Teams 最像工程团队——teammate 是长期在线的同事而非一次性 subagent，共享 task list 落盘本地文件（有状态、依赖、claim 机制和 file locking 防竞态）、mailbox 点对点通信防 context 污染、配 tmux/iTerm2 分屏可观测，但仍是实验能力且 token 成本随 teammate 数量上涨。Kimi K2.5 Agent Swarm 最像搜索集团军——自组织最多 100 个 sub-agents、1500 次并行工具调用，端到端耗时最高降到 1/4.5，用 PARL（并行 Agent 强化学习）让 orchestrator 学会并发拓扑，强项是大规模研究的广度。MiniMax Mavis 最像产品化工作流——状态机接管流程、前台响应与后台执行解耦、Owner/Worker/Verifier 三角色分工且 Worker 与 Verifier 上下文隔离做对抗验证。结论：未来差距不只在模型，更在 runtime 和 harness；管不住 harness，10 个 agent 只是 10 倍混乱 ^[raw/articles/2026-05-21-Claude-Code-vs-Kimi-vs-MiniMax-Agent-Tea-科技充电站.md]

## 关键要点

- Agent Teams 第一性问题：谁来决定任务怎么拆、什么时候停、谁来隔离上下文、谁来验收——多 agent 的价值在于把混乱变成流程，不在于人多。
- Claude Code 两大利器：共享 task list（任务落盘本地文件 + claim 机制 + file locking 防竞态）、mailbox 点对点消息（安全 reviewer 不需要知道 UI 细节就不污染它的 context）。
- Kimi K2.5 官方数据：Agent Swarm 最多 100 个 sub-agents、1500 次并行工具调用、端到端耗时最高降至原来的 1/4.5；PARL 让模型自己学会并发编排。
- MiniMax Mavis 痛点诊断：单 agent 长任务会突然停、context 变长质量下降、复杂任务阻塞用户交互、prompt 角色扮演做不到真分工。
- 选型 5 问：任务状态是否落盘、agent 之间是否隔离、验证者是否独立、高风险动作有没有 approval gate、成本有没有熔断（retry 上限）。

## 来源

- 原文: [[raw/articles/2026-05-21-Claude-Code-vs-Kimi-vs-MiniMax-Agent-Tea-科技充电站.md|Claude Code vs Kimi vs MiniMax Agent Tea 科技充电站]]
