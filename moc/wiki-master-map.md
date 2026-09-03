---
title: Topic Map
created: 2026-04-30
updated: 2026-06-11
type: moc
tags: [meta]
sources: []
confidence: high
---
# Wiki Topic Map 的最佳实践与结构设计？
这页是知识库的主题导航层，用来从“收藏夹式列表”切到“专题路径”。优先从每组的第一批页面读起，再进入相关 raw source。
## Agent Harness 与工程架构
- [[entities/agent-engineering-principles-architecture-practice|agent-engineering-principles-architecture-practice — 12 维工程实践与评测优先原则。
- [[entities/langchain-anatomy-agent-harness|The Anatomy of an Agent Harness 解读 — Agent=Model+Harness 的组件化拆解。
## Memory 与 Context 管理
- [[queries/agent-memory-system-design]] — Memory System 设计检查表。
- [[entities/agent-memory-architecture-essence|Agent Memory 架构本质 — Memory 难点在治理，不在容量。
- [[entities/agent-memory-modular-framework|Agent Memory 模块化框架与评测：Memory in the LLM Era 4 模块 + 10 方案对比 + 新方法 F1 38.79 + 4 条工程原则 — ICLR 2026 模块化框架与评测。
- [[entities/agent-harness-context-management-working-set|Agent Harness 上下文管理：工作集视角 — Context 作为 working set 的视角。
- [[entities/hermes-agent-memory-system-three-layer-architecture|拆解 Hermes Agent 的记忆系统：一个生产级 AI 记忆是怎么设计的 — Hermes/OpenClaw 记忆系统对比。
## Skill、Sub-Agent 与团队编排
- [[entities/agent-skill-writing-guide|'从 0 到 1 教你写 Agent Skill，让 AI 懂你的 — Skill 编写与渐进式披露。
- [[entities/skill-design-patterns|Skill 设计模式 — 线性、决策树、循环、接力棒、多阶段模式。
- [[entities/claude-code-subagent-context-hygiene|Claude Code Subagent 上下文卫生 — Subagent 作为上下文卫生工具。
- [[entities/sub-agent-vs-agent-team-selection|Sub-Agent vs Agent Team 选型与编排原语 — Sub-Agent vs Agent Team 选型。
- [[comparisons/skill-system-design-comparison]] — OpenClaw / Claude Code / Hermes 三方对比。
## Claude Code 与 AI Coding
- [[entities/claude-code-architecture|Claude Code 架构解析 — Claude Code 七大模块。
- [[entities/claude-code-source-architecture|Claude Code 源码拆解：从启动到多 Agent 扩展层 — 源码级八大模块拆解。
- [[entities/claude-code-agent-engineering|Claude Code 的 Agent 工程 — StreamingToolExecutor、上下文压缩、小模型记忆。
- [[entities/claude-code-core-developer-lessons-action-space-design|claude code core developer lessons action space design — Action Space 与 AskUserQuestion 设计。
- [[entities/ai-era-git-version-control-agentic-coding-practices|AI 时代的 Git 版本管理最佳实践 — AI 时代 Git/Worktree/Stacked PR 实践。
## Agent 工具与运行时
- [[entities/crawler-vs-opencli-doubao|crawler vs opencli doubao — 万物 CLI 化的 AI 原生运行时。
- [[entities/cli-anything-wechat-demo|让 Agent 自主完成任务 — 将软件转为 Agent 可调用 CLI。
- [[entities/autocli|AutoCLI — Rust 网页信息获取 CLI。
- [[entities/agent-browser|AgentBrowser — Agent 专用浏览器。
- [[comparisons/cli-tools-comparison]] — 四类工具横向对比。
## 模型、推理与训练
- [[concepts/attention-mechanism]] — 注意力机制与 AttnRes 变体。
- [[concepts/transformer-architecture]] — Transformer 架构演进。
- [[concepts/scaling-laws]] — Scaling Laws 与训练效率。
- [[concepts/inference-optimization]] — KV Cache、PD 分离、投机采样与量化。
- [[entities/glm5-scaling-pain-inference|glm5-scaling-pain-inference — Coding Agent 推理稳定性复盘。
## 知识库与学习系统
-  — 基于 review metadata 的阅读优先级队列。
-  — 低置信、冲突、过期与未评分页面的维护视图。
- [[concepts/tencent-ai-team-knowledge-harness|腾讯 AI Team 知识沉淀体系（Harness Engineering 实践） — AI Team 知识沉淀体系。
- [[entities/learning-path-to-senior|百万年薪学习计划 — 面向长期成长的学习路径。
- [[comparisons/ai-knowledge-tools-comparison]] — AI 知识管理工具横向对比。