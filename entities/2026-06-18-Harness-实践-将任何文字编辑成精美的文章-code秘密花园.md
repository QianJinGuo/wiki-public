---
title: "Harness 实践 将任何文字编辑成精美的文章 code秘密花园"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md|原文存档]]

sha256: e72946bb33e563e6bf2c377e03fc482f0c505b3a35695e4492ef68e18f70ee53 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]

## 摘要

作者 ConardLi 在此前"Agent 自动制作知识讲解视频"的实践基础上验证"好的 Harness 是可以迁移的"，开源了 beautiful-article Skill，用几乎同一套骨架把任意素材（URL、PDF、DOCX、文本）变成排版精美的网页长文 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]。Skill 把流程切成 8 个 Phase 并卡 3 个强制人工检查点（Plan、First Spread、Delivery），底层是 Reacticle 组件协议——一套受约束的语义化 React 组件契约加 Raw 逃生舱，配 11 套编辑级主题，让 AI 只负责组合组件、不操心版式，保证"稳定、不崩版" ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]。实战采用 Claude Code + MiniMax M3（1M 上下文、原生多模态）组合，终审由 Editorial/Visual/Technical 三个独立 SubAgent 分别质检，修复采用最小切片策略 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]。文章最后把视频 Skill 与文章 Skill 逐层对照，指出上下文管理、工具系统、执行编排、状态记忆、评估观测、约束恢复六个 Harness 部件设计几乎一致，且质检修复日志落盘可以反过来驱动 Skill 自进化 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]。

## 关键要点

- Reacticle = React + Article：提供 Article/Hero/Lead/Section/Table/Quote/Formula/CodeBlock 等语义组件平替 Markdown 语法；Raw 自由层可塞任意 HTML/SVG/CSS，但所有样式必须消费约束好的主题 token
- 8 个 Phase：Intake → Source → Editorial Planning → Plan Checkpoint（★）→ First Spread（★）→ Full Article Build → Final Review → Repair → Delivery（★）；检查点禁止替用户做主，每个决策项必须独立等用户答复
- 内置 9 种文章类型，信息保留比例从约 25%（interactive-explainer）到 100%（longform）不等；还有 narrow/regular/wide/full 版式与 none/user-assets/placeholders/ai-generated 配图模式
- 一节一文件是多 Agent 并行的铁律：每个并行 SubAgent 只负责一个 section 文件，大型 Raw 放独立的 raw-blocks/，主 Agent 只负责组装
- 实战环境：Claude Code + MiniMax M3 + CC Switch 切换供应商；产出 CSS+JS 全内联的单文件 HTML（断网可打开）与 PDF 两种交付物

## 来源

- 原文：[[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md|Harness 实践 将任何文字编辑成精美的文章 code秘密花园]]
