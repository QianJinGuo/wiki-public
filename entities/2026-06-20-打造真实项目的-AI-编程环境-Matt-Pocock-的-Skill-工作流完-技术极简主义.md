---
title: "打造真实项目的 AI 编程环境 Matt Pocock 的 Skill 工作流完 技术极简主义"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-20-打造真实项目的-AI-编程环境-Matt-Pocock-的-Skill-工作流完-技术极简主义]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/2026-06-20-打造真实项目的-AI-编程环境-Matt-Pocock-的-Skill-工作流完-技术极简主义.md|原文存档]]

sha256: 37ae7b4c863e350f0ad8141e2a2a02c7b4a474bc480988d0029acf47adb8f6f6 ^[raw/articles/2026-06-20-打造真实项目的-AI-编程环境-Matt-Pocock-的-Skill-工作流完-技术极简主义.md]

## 摘要

技术极简主义公众号解读 TypeScript 社区知名开发者 Matt Pocock 于 2026 年 3 月开源的 Claude Code skills 仓库（mattpocock/skills，副标题 "Skills for Real Engineers. Straight from my .claude directory"）。它不是多 Agent 编排框架，本质是一组 Markdown 文件，每个 Skill 对应一种工程实践，核心观点是：AI 编程的失败普遍源于工程反馈链的失效，而非模型不够聪明。针对四大问题给出对应 Skill：需求没听懂用 /grill-me 和 /grill-with-docs（让 Agent 反过来拷问用户并把领域语言沉淀到 CONTEXT.md）；代码"形状正确但不 work"用 /tdd（严格 red-green-refactor 垂直切片，一个行为一个失败测试一个最小实现）和 /diagnose（复现→缩小范围→假设→插桩验证→修复）；架构腐化用 /to-prd、/to-issues（按端到端可验收的垂直切片拆任务而非按文件拆）、/zoom-out、/improve-codebase-architecture ^[raw/articles/2026-06-20-打造真实项目的-AI-编程环境-Matt-Pocock-的-Skill-工作流完-技术极简主义.md]

安装一条命令 npx skills@latest add mattpocock/skills，初始化用 /setup-matt-pocock-skills 确认 issue tracker、triage 标签、领域文档位置等约定。文章还把它与 GSD（长周期任务管理）、BMAD（角色分工与流程规范）、Superpowers（TDD）、Spec-Kit（规格驱动）对比，给出选择建议：日常开发优先 Matt Pocock Skills，跨天大任务用 GSD，0 到 1 重流程用 BMAD，重视测试质量参考 Superpowers，需求规范驱动的企业项目用 Spec-Kit。结论：真正困难的从来不是模型能力，而是需求是否清晰、反馈是否及时、测试是否完善、决策是否被记录 ^[raw/articles/2026-06-20-打造真实项目的-AI-编程环境-Matt-Pocock-的-Skill-工作流完-技术极简主义.md]

## 关键要点

- 标准使用流六步：/grill-with-docs 对齐需求 → /to-prd 落成文档 → /to-issues 拆垂直切片 → /tdd 最小实现 → /diagnose 处理难复现 bug → /zoom-out 或 /improve-codebase-architecture 周期性修剪架构。
- /grill-with-docs 把领域术语沉淀到 CONTEXT.md，下次 session Agent 不用重新猜概念，省 token 也统一项目语言。
- /tdd 反对一次性写完所有测试再实现，推荐垂直切片：RED→GREEN 每次只推进一个可观察行为。
- /diagnose 的关键洞察：调试效率取决于反馈速度而非阅读代码速度，理想状态是一条命令几秒内给出成败。
- 初始化 /setup-matt-pocock-skills 会生成领域语言、架构决策记录（docs/adr/）、标签字典、issue 工作流说明等协作文件，让 AI 进入项目时能读到团队既有上下文。

## 来源

- 原文: [[raw/articles/2026-06-20-打造真实项目的-AI-编程环境-Matt-Pocock-的-Skill-工作流完-技术极简主义.md|打造真实项目的 AI 编程环境 Matt Pocock 的 Skill 工作流完 技术极简主义]]
