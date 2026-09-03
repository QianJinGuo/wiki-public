---
title: "01"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-18-Loop-Engineering-概念解析-思考与实践-阿里技术]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-18-Loop-Engineering-概念解析-思考与实践-阿里技术.md|原文存档]]

sha256: 2ba08e551bd4caf5dd0442ded8a92b387c3f6e69d4efe5a42a34b048a5c47895 ^[raw/articles/2026-06-18-Loop-Engineering-概念解析-思考与实践-阿里技术.md]

## 摘要

阿里技术公众号系统化解析 Loop Engineering（循环工程）概念的文章。背景是大佬密集发声：Claude Code 负责人 Boris Cherny 表示已不手写 Prompt 而是编写 Loop 驱动工作流，OpenClaw 创始人 Peter Steinberger 主张通过设计 Loop 引导 Agent 行为，Karpathy 强调"你必须把你自己从 Loop 的执行过程中移除出去"，Google AI 总监 Addy Osmani 2026 年 6 月正式定义该概念——"Loop Engineering 就是把你从'给 Agent 提示词的人'这个位置上替换掉，转而设计一套能自动完成这件事的系统"。文章辨析 Agent Loop（底层执行循环，ReAct/Ralph Loop 等变种，已是默认基础设施）与 Loop Engineering（构建在 Harness 之上、面向需求验收的外部 Loop）的本质区别：后者把原本"人在循环（HITL）"里催促模型"继续/报错/回滚"的环节自动化掉，从 Vibe Coding 的"提一个需求"进化为"提一套闭环流程"^[raw/articles/2026-06-18-Loop-Engineering-概念解析-思考与实践-阿里技术.md]

文章按 Addy Osmani 的定义梳理 Loop 六大核心部分：Automations（定时循环，Codex 的 /goal 持续跑到条件满足、Claude Code 靠 Cron/Hook 的 /loop）、Worktrees（Git 工作树隔离解决多 Agent 文件冲突）、Skills（可自我沉淀的"活的知识"）、Connectors/Plugins（MCP 接入外部工具）、Sub Agents（验收 Sub Agent 与主 Agent 解耦、用角色隔离打破"当局者迷"，但并非越多越好）、State（Markdown 或 Linear 追踪完成状态）。实践部分用文本分类任务演示 Loop 写法（准确率 ≥95% 写入 Loop 定义让 Agent 自主打磨），并给出务实建议：固定流程沉淀为脚本、需动态判断的做成 Skill（避免重跑 Loop 费 token 和结果漂移）；提醒 Loop 不是银弹——需求和验证标准必须比 HITL 时代写得更明确，写不清楚时回到人工迭代反而更稳妥 ^[raw/articles/2026-06-18-Loop-Engineering-概念解析-思考与实践-阿里技术.md]

## 关键要点

- Loop vs Agent Loop：底层 Agent Loop（Function Call 循环 + Validation Agent/Max Steps 兜底）是基础设施；Loop Engineering 是面向需求验收的上层外部 Loop，把"继续/报错/回滚"的人工催促自动化。
- 概念演进链条：Coding（写代码）→ Vibe Coding（提需求）→ Loop Engineering（提一套闭环流程：开发→测试→验收→调优→反馈迭代全链路定义好）。
- Loop Pipeline 两种触发方式：人工触发（写一个 Loop 形式的 Pipeline 逐步自动执行）与定时触发（每日 PR 评审合并、周报日报汇总、股票盯盘等周期性任务）。
- 六大核心部分：Automations、Worktrees、Skills、Connectors/Plugins、Sub Agents、State；验收类 Sub Agent 务必独立，避免"既当运动员又当裁判员"。
- 落地心法：Loop 对使用者描述需求和验证的能力要求更高——人退出中途纠偏后，模糊的需求会让 Loop 从一开始就跑偏、白烧 token；需求模糊时回到 HITL 更稳妥。

## 来源

- 原文: [[raw/articles/2026-06-18-Loop-Engineering-概念解析-思考与实践-阿里技术.md|01]]
