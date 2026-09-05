---
title: "Everything Claude Code（ECC）——社区 Harness 蒸馏集"
created: 2026-09-05
updated: 2026-09-05
type: entity
tags: [claude-code, harness-engineering, community, agents, skills, hooks, probe]
confidence: 0.6
provenance_state: extracted
status: probe-archive
---

# Everything Claude Code（ECC）——社区 Harness 蒸馏集

> 本地探针存档页（2026-09 外部系统透镜轮）：`~/projects/everything-claude-code`（公开源 URL 待补）。社区维护的 Claude Code 生产级配置蒸馏集——**30 个专业 agents、135 skills、60 commands、自动化 hook 工作流**（SOUL.md 自述），1756 个 md 文件。

## 结构速览

- **身份层**：`SOUL.md`——自我描述为"ECC 共享身份、治理与技能的**可移植层**（gitagent surface）"；agent 人格/身份作为独立工件版本化管理。
- **规范层**：`RULES.md`/`CLAUDE.md`/`AGENTS.md`/`agent.yaml`/`manifests/`/`schemas/`——配置即代码，含 schema 校验与 manifest 分发。
- **执行层**：`agents/`（30 个分工 agent）、`commands/`（60 命令）、`skills/`（135 技能）、`hooks/`（自动化钩子）、`mcp-configs/`。
- **元层**：`EVALUATION.md`（自评估）、`REPO-ASSESSMENT.md`（仓库体检）、`ecc_dashboard.py`（可观测）、`the-longform-guide.md`/`the-shortform-guide.md`/`the-security-guide.md`（三层文档）、`research/`。

## 相关

探针碰撞结论见 [[drafts/wiki-emergent-viewpoints-2026-09-external-probe|外部系统透镜涌现稿]]；正方簇：[[concepts/harness-engineering-framework|Harness Engineering 框架]]；同类探针：[[entities/openclaw-architecture-8-part-summary|OpenClaw 架构]]
