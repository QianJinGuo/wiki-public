---

title: "Claude Code Harness 深度分析"
type: entity
tags: [agent, claude, context, harness, llm, prompt]
created: 2026-05-21
updated: 2026-09-07
review_value: 7
review_confidence: 7
sources: [raw/articles/claude-code-harness-deep-dive-founder-park]
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.75: harness分析重复版; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Claude Code Harness 深度分析

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.75**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/claude-code-harness-deep-dive-founder-park.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agentscope-java-harness-framework-enterprise-distributed|AgentScope Java Harness Framework 2.0 — 企业级 Agent 分布式场景的 Harness 实现 (Java 2.0 重大升级)]] — AgentScope Java全版
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/anthropic-n-days-frontier-agent-vulnerability-research|Anthropic N-days: Frontier Agent Vulnerability Research]] — N-day研究
- [[entities/perplexity-search-as-code-generation|Rethinking Search as Code Generation]] — Search as Code：查询变可执行代码对象7660字rv9
- [[entities/agent-harness-12-components-7-decisions|一篇看懂 Agent Harness 的结构！ — 12组件+7决策完整框架]] — harness 12组件框架
- [[entities/openclaw-prompt-context-harness|深度解析 OpenClaw 在 Prompt / Context / Harness 三个维度中的设计哲学与实践]] — 三维度源码：23模块拼装+自适应分块+双层Memory
- [[entities/code-as-agent-harness-survey|Code as Agent Harness 综述]] — 102页综述
- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道|深入理解 Claude Code 源码中的 Agent Harness 构建之道]] — 16095字源码8步循环
- [[entities/from-prompt-to-harness-claude-official|从 Prompt 到 Harness：Claude 官方学习资料]] — Harness五子系统闭环解读
- [[entities/claude-code-search-architecture-tencent-2026|原始文章存档]] — ripgrep五层过滤
- [[entities/claude-code-and-what-comes-next|Claude Code and What Comes Next]] — 压缩/Skills/Subagents
- [[entities/claude-perceived-degradation-anthropic-effort-model-explanation-2026|全网骂Claude变笨，Anthropic下场揭秘：坑你的不是模型]] — Model vs Effort框架

## 工程实践
- [[entities/karpathy-vibe-coding-agentic-engineering-v4|Karpathy 最新访谈：从 Vibe Coding 到 Agentic Engineering]] — v4 8090字rv10：可验证性上限+MenuGen警示
- [[entities/andrej-karpathy-claude-md-134k-stars-2026|最佳 Claude Code 配置：Andrej Karpathy 的 CLAUDE.md，134+k star了！]] — CLAUDE.md四规则解析
- [[entities/anthropic-95pct-data-analysis-jiagoux-data-level-harness-20260606|数据级 Harness：架构师 JiaGouX 解读 Anthropic 95% 数据分析与 5 个反直觉边界]] — 数据级harness解读
- [[entities/vivo-agent-brain-body-icu-harness-evolutionary-framework-2026|vivo Agent 系统分析：大模型是大脑不是马，Harness 是 ICU 不是马鞍]] — 大脑身体ICU隐喻框架
- [[entities/autoresearch-marketing-growth-amap-ai-native|高德 Marketing AutoResearch：AI Native 营销增长经营托管框架]] — 营销经营托管
- [[entities/claude-code-context-engineering-anthropic-thariq|Claude Code 上下文工程 —— Anthropic 团队的工程实践]] — 上下文工程官方表述
- [[entities/anthropic-managed-agents-scaling|Anthropic Managed Agents：用 K8s 思路虚拟化 Agent 组件]] — 宠物到牛群
- [[entities/claude-code-best-community-fork-evolution-vibecoder|Claude Code 泄露后的漏网之鱼 claude-code-best 这两个月到底演进了什么]] — 社区fork演进短条
- [[entities/coze-3-0-local-agent-codex-claude-code-project|扣子 3.0 离谱更新：把 Codex、Claude Code 拉进一个项目工作？]] — coze-bridge本地接入
- [[entities/claude-code-skills-workflow-encapsulation-costa-long|Skills：让 Claude 记住「怎么做」，告别重复教学]] — context:fork隔离
- [[entities/iqsixinp9lxnkg7avfhfcq|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris新访谈：IDE→Agent控制台控制点迁移

## 延伸导航
- [[moc/agent-engineering-guide|Agent 工程全景指南]]
