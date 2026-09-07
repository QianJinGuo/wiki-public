---

title: "Claude Code Prompt 与上下文 Harness 设计"
type: entity
tags: [agent, claude, coding, context, harness, prompt]
created: 2026-05-21
updated: 2026-09-07
review_value: 7
review_confidence: 7
sources: [raw/articles/claude-code-prompt-context-harness]
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: prompt模块重复版; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Claude Code Prompt 与上下文 Harness 设计

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/claude-code-prompt-context-harness.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agentscope-java-harness-framework-enterprise-distributed|AgentScope Java Harness Framework 2.0 — 企业级 Agent 分布式场景的 Harness 实现 (Java 2.0 重大升级)]] — AgentScope Java全版
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/pi-openclaw-coding-harness|Coding Harness 工程本质：从 Pi 到 OpenClaw]] — Harness八能力+五工程模式：Context像投影8441字rv9
- [[entities/anthropic-n-days-frontier-agent-vulnerability-research|Anthropic N-days: Frontier Agent Vulnerability Research]] — N-day研究
- [[entities/agent-harness-12-components-7-decisions|一篇看懂 Agent Harness 的结构！ — 12组件+7决策完整框架]] — harness 12组件框架
- [[entities/openclaw-prompt-context-harness|深度解析 OpenClaw 在 Prompt / Context / Harness 三个维度中的设计哲学与实践]] — 三维度源码：23模块拼装+自适应分块+双层Memory
- [[entities/glm-52-is-the-step-change-for-open-agents|GLM-5.2 is the step change for open agents]] — Interconnects评GLM-5.2开放Agent跃迁
- [[entities/agent-harness-architecture-deep-dive-aksahy|Agent Harness 解析：智能体架构深度拆解]] — harness解剖深度
- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道|深入理解 Claude Code 源码中的 Agent Harness 构建之道]] — 16095字源码8步循环
- [[entities/from-prompt-to-harness-claude-official|从 Prompt 到 Harness：Claude 官方学习资料]] — Harness五子系统闭环解读
- [[entities/claude-code-search-architecture-tencent-2026|原始文章存档]] — ripgrep五层过滤
- [[entities/claude-opus-47|Claude Opus 4.7 并不是一次全面升级，甚至部分能力大幅衰退]] — 4.7衰退面分析
- [[entities/fudan-peking-ahe-agentic-harness-engineering|复旦北大 AHE：Agentic Harness Engineering 瓶颈分析]] — AHE三支柱可观测性5622字深析版
- [[entities/claude-code-and-what-comes-next|Claude Code and What Comes Next]] — 压缩/Skills/Subagents

## 工程实践
- [[entities/tencent-vibe-coding-to-agentic-engineering-backend|Tencent Vibe Coding to Agentic Engineering Backend]] — 全流程串终端会话实践
- [[entities/gaode-sdd-harness-team-ai-coding-paradigm-ibjfu|高德广告工程 Harness/SDD 体系演进：从\]] — SDD+Harness团队级范式11119字
- [[entities/build-a-serverless-image-editing-agent-with-amazon-bedrock-a|Build a serverless image editing agent with Amazon Bedrock AgentCore harness]] — 图像编辑agent
- [[entities/aws-bedrock-agentcore-quality-optimization-flywheel|AWS Bedrock Agentcore Quality Optimization Flywheel]] — 质量飞轮
- [[entities/ai-production-development-workflow-openspec-superpowers-gstack|AI 生产开发工作流：OpenSpec 规范驱动 + Superpowers 工具链]] — 三件套工作流
- [[entities/2-小时0-行手写代码我用-claude-做了一个生产级-vscode-插件|2 小时，0 行手写代码，我用 Claude 做了一个生产级 VSCode 插件]] — 实践复盘有具体经验教训
- [[entities/claude-code-context-engineering-anthropic-thariq|Claude Code 上下文工程 —— Anthropic 团队的工程实践]] — 上下文工程官方表述
- [[entities/anthropic-managed-agents-scaling|Anthropic Managed Agents：用 K8s 思路虚拟化 Agent 组件]] — 宠物到牛群
- [[entities/长周期-agent-详解-从-ralph-loop-到可接管-harness|长周期-agent-详解-从-ralph-loop-到可接管-harness]] — Ralph loop到接管harness
- [[entities/准备开一个新坑从零复刻一个-claude-codenn目标是在这个过程中和大家一起学习-claude-code-的-harness-是如何做的nnclaude-|从零复刻 Claude Code：Harness 构建学习笔记]] — easy-agent复刻路线图

## 延伸导航
- [[moc/agent-engineering-guide|Agent 工程全景指南]]
