---
title: "cursor ios mobile app"
created: 2026-08-01
updated: 2026-09-07
type: entity
tags: ['raw', 'article']
sources: [raw/articles/cursor-ios-mobile-app]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/cursor-ios-mobile-app.md|原文存档]]

sha256: 9d7485d970079eba039c75cd4b2ab25db70050ad853c2b91e9236ed0183efeb4 ^[raw/articles/cursor-ios-mobile-app.md]

## 摘要

Cursor 官方博客（2026-06-29）宣布 Cursor for iOS 原生应用进入公测：开发者可以从手机启动云端 always-on agent，或用 Remote Control 远程操控运行在自己电脑上的 agent，灵感来时随时开工，工作完成后收到通知，在手机上直接 review 和 merge PR。手机端支持选择任意 frontier 模型、语音输入描述想法、slash commands 引导；针对本机 agent 可开启保持电脑唤醒的设置。官方团队和早期测试者已用它形成新工作流：on-call 时被 pager 叫醒就先让 agent 调查并准备 PR、离席时处理客户时效性 bug、在 X 上看到用户反馈就截图标注发给 agent 作为 UI 改动的视觉上下文 ^[raw/articles/cursor-ios-mobile-app.md]

Cloud agents 跑在隔离虚拟机里，带完整开发环境做测试、验证和 demo，可异步长时间运行迭代出 merge-ready PR；支持 local plan 发给 cloud agent、把活跃 agent 移到云端继续跑、再把云端会话移回本地测试后才 merge。通知侧依赖锁屏 Live Activities 和推送通知；cloud agent 产出 demo、截图和日志便于验证工作。路线图上，Cursor 希望云端跑 agent 的体验最终与本机无异，并计划支持 repo-less chats（团队已用 MCP 查询 Datadog 日志、汇总 Slack 频道动态）。iOS 版对所有付费计划开放，2026-07-05 前 Composer 2.5 移动端运行 75% 折扣 ^[raw/articles/cursor-ios-mobile-app.md]

## 关键要点

- 双模式：云端 always-on agent（隔离 VM、完整开发环境、异步长时间运行）+ Remote Control 远程操控本机 agent。
- 通知机制：锁屏 Live Activities + 推送通知（agent 完成、需要输入、待 review 时触发）；cloud agent 产出 demo/截图/日志供验证。
- Local↔Cloud 交接：本地计划发给云端 agent、活跃 agent 可迁云、云会话可移回本地测试后再 merge。
- 早期工作流案例：on-call 事故处理、客户时效性 bug 修复、社交平台反馈截图作为视觉上下文启动 UI 改动。
- 可用性与优惠：iOS 公测面向所有付费计划；2026-07-05 前移动端 Composer 2.5 运行 75% 折扣；后续将支持 repo-less chats。

## 来源

- 原文: [[raw/articles/cursor-ios-mobile-app.md|cursor ios mobile app]]
