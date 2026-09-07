---
title: "Meet Noz, your AI teammate inside SigNoz"
created: 2026-08-01
updated: 2026-09-07
type: entity
tags: ['raw', 'article']
sources: [raw/articles/meet-noz-your-ai-teammate-inside-signoz]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/meet-noz-your-ai-teammate-inside-signoz.md|原文存档]]

sha256: 6231167270f562d4f70985676559008e14ad5012606e0cc3e4aa339bef1d9ac3 ^[raw/articles/meet-noz-your-ai-teammate-inside-signoz.md]

## 摘要

SigNoz 发布了内置于产品的 AI 队友 Noz（beta 阶段免费），它以侧边栏（sidepane）形式存在、自动感知当前页面上下文，可以基于用户自己的遥测数据和文档回答问题，也能根据自然语言描述直接生成告警规则和仪表盘，例如"当 checkout p99 延迟超过 2 秒时告警"即可自动构建对应告警规则。这个产品的出发点并非"加一个 AI 聊天机器人"，而是观察到用户长期把产品内置的人工客服聊天当成 AI 使用——期望即时回答 dashboard 构建和数据查询问题——说明产品内助手的需求早已存在。^[raw/articles/meet-noz-your-ai-teammate-inside-signoz.md]

文章还区分了 Noz 与上月发布的托管 MCP 服务器的定位：在 SigNoz 界面内工作、追求零配置时用 Noz（已随会话认证、可深链回 explorer）；当调查需要超出 SigNoz 范围（如关联最近 commit、GitHub issue、Slack 讨论）时，则通过 MCP 服务器让 Claude Code、Cursor、Codex 等外部 Agent 拉取 SigNoz 数据。后续路线图包括接入团队已有工具以引入更多调查上下文、承担更多重复性事故响应工作以降低 MTTR，以及构建告警触发式调查（告警触发时 Noz 自动开始收集上下文）。^[raw/articles/meet-noz-your-ai-teammate-inside-signoz.md]

## 关键要点

- Noz 目前处于 beta 阶段、免费使用，内置于 SigNoz Cloud 的侧边栏和独立全屏页面，无需安装。
- 用户过去一年已把产品内人工客服聊天当 AI 用，甚至发现对面是真人后道歉，这证明产品内即时问答需求真实存在。
- Noz 与 MCP 服务器共用同一开源基础：Noz 服务于 SigNoz 界面内的零配置场景，MCP 服务器服务于跨工具的外部 Agent 调查场景。
- 下一步方向：引入更多调查上下文、处理更多事故响应任务以降低 MTTR、支持告警触发后自动开始收集上下文。

## 来源

- 原文: [[raw/articles/meet-noz-your-ai-teammate-inside-signoz.md|Meet Noz, your AI teammate inside SigNoz]]
