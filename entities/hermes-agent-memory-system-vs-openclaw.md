---

title: "Hermes Agent 记忆系统深度拆解"
created: 2026-05-18
updated: 2026-09-10
type: entity
tags: [hermes, openclaw, agent, memory, architecture, cache-aware]
provenance_state: extracted
source_url:
review_value: 9
sources: [raw/articles/hermes-agent-memory-system-vs-openclaw]
review_confidence: 8
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 记忆拆解10142字版，留15260字版; retained as hub (in-links>=20); MOC rewrite candidate"
---
# Hermes Agent 记忆系统深度拆解

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/hermes-agent-memory-system-vs-openclaw.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/hermes-agent-memory-system-openclaw-comparison|深度拆解 Hermes Agent 记忆系统]] — 记忆成本账四层体系15260字rv10最深版
- [[entities/17-agent-architectures-evolution|17种Agent架构演进：控制流设计的完整演化史]] — 17架构系统拆解高价值
- [[entities/skill-system-design-three-way-comparison|AI Agent 架构设计（七）：Skills 系统设计（OpenClaw、Claude Code、Hermes Agent 对比）]] — 三框架skill系统设计对比
- [[entities/pi-openclaw-coding-harness|Coding Harness 工程本质：从 Pi 到 OpenClaw]] — Harness八能力+五工程模式：Context像投影8441字rv9
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/open-claw-tool-bus-subagent-architecture|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]] — 薄抽象显式控制流8802字rv9全版
- [[entities/hermes-agent-closed-learning-loop|Hermes Agent 闭环学习机制]] — 闭环学习飞轮+Nudge触发+spawn_background_review
- [[entities/openclaw-prompt-context-harness|深度解析 OpenClaw 在 Prompt / Context / Harness 三个维度中的设计哲学与实践]] — 三维度源码：23模块拼装+自适应分块+双层Memory
- [[entities/how-ai-agent-memory-works|How AI Agent Memory Works]] — 记忆五层+六架构权衡科普
- [[entities/hermes-skill-system-winty|Skill 系统：Agent 如何把经验沉淀成可复用能力]] — Memory vs Skill本质区别7749字最全版
- [[entities/hermes-agent-vs-openclaw-comparison|Hermes Agent 为什么火了？和 OpenClaw 龙虾比一比]] — 爱马仕vs龙虾：控制面vs成长型定位对比
- [[entities/gepa-optimize-anything|Gepa Optimize Anything]] — ASI+Pareto前沿，声明式通用文本优化API
- [[entities/hermes-self-evolution-closed-loop-skill-reuse-winty|Hermes自进化完整闭环：Skill创建复用修补链路]] — 6阶段闭环+npm案例12→9→6步
- [[entities/gateway-architecture-openclaw-claude-hermes-comparison|AI Agent Gateway 架构设计 — OpenClaw/Claude Code/Hermes 三框架对比]] — 三框架Gateway哲学横向对比，源码级细节
- [[entities/nanobot-agent-framework-architecture-deep-dive|nanobot：4000行极简 Agent 框架架构解析]] — 3935行vs LangChain 43万行的极简哲学
- [[entities/claude-code-vs-hermes-session-vs-goal-lifecycle|Claude Code vs Hermes — Session 工程师 vs Goal Runtime]] — session vs goal
- [[entities/openclaw-hermes-source-code-agent-architecture-review|OpenClaw与Hermes源码架构对比]] — 双框架源码对比：OpenClaw四亮点+Hermes四补充

## 工程实践
- [[entities/impeccable-frontend-design-skill-harness-vibecoder|Impeccable：把 AI 前端设计变成可检查的工作流 — 33.4k Star 开源项目深度分析]] — Impeccable四层架构9210字rv9全版
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 提示词体系源码解析]] — 六大prompt模块全版
- [[entities/memos-hermes-plugin|MemOS Hermes 记忆插件]] — MemOS插件：智能去重+混合检索7225字
- [[entities/anthropic-95pct-data-analysis-jiagoux-data-level-harness-20260606|数据级 Harness：架构师 JiaGouX 解读 Anthropic 95% 数据分析与 5 个反直觉边界]] — 数据级harness解读
- [[entities/autoresearch-marketing-growth-amap-ai-native|高德 Marketing AutoResearch：AI Native 营销增长经营托管框架]] — 营销经营托管
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进|存之有序，治之有矩——Agent 记忆系统的工程实践与演进]] — 写入纪律prompt cache冲突
- [[entities/aliyun-mse-ai-task-scheduling-agent-sandbox-cost-90-percent|阿里云 MSE AI 任务调度 + Agent Sandbox：动态休眠/唤醒 OpenClaw Agent 成本下降 90%+]] — 休眠唤醒短条borderline

## 延伸导航
- [[moc/openclaw-architecture|OpenClaw 的架构设计为什么值得研究？它与 Hermes/Claude Code 的核心差异？]]
- [[moc/agent-memory-architecture-decision-points|Agent Memory 架构选择的关键决策点是什么？]]
- [[moc/agent-engineering-guide|Agent 工程全景指南]]
