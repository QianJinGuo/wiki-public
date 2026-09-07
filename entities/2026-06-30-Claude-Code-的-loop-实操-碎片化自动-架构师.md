---
title: "Claude Code 的 loop 实操 碎片化自动 架构师"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-30-Claude-Code-的-loop-实操-碎片化自动-架构师]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/2026-06-30-Claude-Code-的-loop-实操-碎片化自动-架构师.md|原文存档]]

sha256: c466246b0273d17d6c36c828b072295d801a42fce4e71200daddeb733f6bcc06 ^[raw/articles/2026-06-30-Claude-Code-的-loop-实操-碎片化自动-架构师.md]

## 摘要

文章把 Claude Code 的 /loop 定位为"会话内观察员"——接管工程师日常的碎片化等待（等 CI、等部署、看 PR 评论、看队列深度、看第三方接口恢复），而非"全自动编程"。它有三种写法：固定间隔（/loop 5m <prompt>）、自适应间隔（不写间隔，由 Claude 根据观察节奏决定）和空跑 /loop（使用内置 maintenance prompt 继续处理当前会话未完成事项，项目里若有 .claude/loop.md 则优先使用其默认 prompt）。边界方面：/loop 跟随当前会话（新开会话会清掉、--resume 可恢复、递归任务 7 天自动过期），scheduled tasks 需要 Claude Code v2.1.72+，Bedrock/Vertex AI/Foundry 上空跑只打印用法说明；长期无人值守任务应改用 Routines、GitHub Actions 或自己的调度系统。^[raw/articles/2026-06-30-Claude-Code-的-loop-实操-碎片化自动-架构师.md]

工程方法上，文章给出三条实操原则。第一，先留状态少烧 token：用一个小状态文件（status/last_evidence/blocked_reason/check_count）让每轮醒来先读状态再决定轻查还是展开，每轮结果归为 changed/no_change/blocked/done 四类。第二，prompt 写成运行卡片而非愿望：说清状态（先读哪）、范围（只看什么）、证据（什么算有变化）、动作（何时允许处理）、停止（何时交还给人）五件事。第三，一旦 /loop 从观察走向执行（自动分派修复、开 PR、更新 Linear），就要补齐三样部件——Skill（沉淀重复流程）、隔离工作区（独立 worktree/分支）、验证者（测试/CI/reviewer subagent 与执行者分开）。文章还区分了 /loop 与 /goal：前者适合"隔一会儿看一次"，后者适合"持续推进到可验证条件成立"，且 /goal 的 evaluator 是一个小模型、只能看对话内证据，条件必须可证明（如测试退出码为 0）；loop.md 超 25,000 bytes 会被截断，提醒默认巡检 prompt 要短。^[raw/articles/2026-06-30-Claude-Code-的-loop-实操-碎片化自动-架构师.md]

## 关键要点

- 背景：Peter Steinberger 提出除了给 coding agent 写 prompt，还要练习设计能提示 agent 的 loop；Addy Osmani 随后写成 Loop Engineering 一文。
- 错过的触发不会补跑一堆，只会在空下来后跑一次；固定时间任务带一点 jitter，不适合精确到秒的调度。
- PR 评论监控示例：按文件和严重度分组新评论，标记不清或架构性问题为 needs-human，没有新评论就短回复——先归类再决定哪些自动处理。
- /loop 判断表：需要隔一会儿再看、检查成本低、证据明确（exit code/job status/comment）、失败能分类才适合；需要长期无人值守或直接改生产则不适合。
- loop.md 管的是空跑 /loop 时默认做什么，CLAUDE.md 管的是 Agent 进仓库前先知道什么，两者分工不同。

## 来源

- 原文: [[raw/articles/2026-06-30-Claude-Code-的-loop-实操-碎片化自动-架构师.md|Claude Code 的 loop 实操 碎片化自动 架构师]]
