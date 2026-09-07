---
title: "Claude Opus 5 系统提示词疑似泄露：Agent 规则到底该如何定？"
created: 2026-08-06
updated: 2026-08-06
type: entity
tags: [agent, system-prompt, prompt-engineering, rule-placement, harness, claude-opus-5]
sources: [raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Claude Opus 5 系统提示词疑似泄露：Agent 规则到底该如何定？

## 核心问题：规则该放 Prompt 还是交给工具/记忆/权限/评测？

作者（架构师 JiaGouX）指出一个做 Agent 绕不开的规则归属问题：**一条规则，到底该放进 Prompt，还是交给工具、记忆、权限和评测？** 核心判断原则：**离事实越近、失败代价越高，越不能只靠 Prompt。** 团队常见的不良习惯是"每出一次问题就往系统提示词里补一句"，半年后 Prompt 越写越长，却说不清哪些规则仍有效、哪些早该交给工具、代码和权限系统。^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md]

## Opus 5 泄露样本的三个版本

作者收集了三份公开流传的 Claude Opus 5 样本，分别为 135,669 / 202,762 / 224,359 字节，行数、哈希和结构都对不上，无法确认哪份是官方逐字原文。恰恰是这些差异说明：大家口中的"系统提示词"可能早已混入工具 Schema、记忆规则、Skill 注册表和运行环境——提示词与运行时配置的边界正在模糊化。^[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定.md]

## 与 Wiki 现有知识的关联

- 规则归属决策与 [[entities/system-prompt-vs-post-training-behavioral-constraints-2026|System Prompt vs Post-Training 行为约束]] 是同一问题的两面：前者是"放哪一层"，后者是"Prompt 与权重约束哪个更可靠"
- 与 [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering 范式]] 一致：规则应下沉到 harness 的工具/记忆/权限层
- Prompt 膨胀与失效问题见 [[entities/attention-collapse-context-management|Attention Collapse 与上下文管理]]

→ [[raw/articles/claude-opus-5-系统提示词疑似泄露agent-规则到底该如何定|原文存档]]
