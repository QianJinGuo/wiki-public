---

title: "智能体编排层中的上下文管理架构"
type: entity
tags: [agent, architecture, context]
created: 2026-05-21
updated: 2026-09-10
review_value: 7
review_confidence: 7
sources: [raw/articles/agent-context-management-architecture-patterns]
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: dup-correction: 同主题保留更全版本; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# 智能体编排层中的上下文管理架构

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/agent-context-management-architecture-patterns.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计与实现：生产级 Agent 系统落地指南]] — 七层金字塔生产指南
- [[entities/17-agent-architectures-evolution|17种Agent架构演进：控制流设计的完整演化史]] — 17架构系统拆解高价值
- [[entities/skill-system-design-three-way-comparison|AI Agent 架构设计（七）：Skills 系统设计（OpenClaw、Claude Code、Hermes Agent 对比）]] — 三框架skill系统设计对比
- [[entities/rein-go-agent-4-modules-5-type-boundaries|Rein：4 模块 + 5 类型边界防止 agent.go 膨胀到 3000 行]] — 4模块+5类型边界+7不变量：数据契约防上帝文件
- [[entities/pi-openclaw-coding-harness|Coding Harness 工程本质：从 Pi 到 OpenClaw]] — Harness八能力+五工程模式：Context像投影8441字rv9
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/open-claw-tool-bus-subagent-architecture|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]] — 薄抽象显式控制流8802字rv9全版
- [[entities/hermes-agent-closed-learning-loop|Hermes Agent 闭环学习机制]] — 闭环学习飞轮+Nudge触发+spawn_background_review
- [[entities/mira-mpa-deep-principle-ai4s-40-sota|MIRA + MPA：深度原理 AI Scientist 递归自训练打造材料基座模型，40 项实验全面 SOTA]] — AI Scientist递归自训练，35/40胜前SOTA
- [[entities/the-coming-loop|The Coming Loop]] — Ronacher两种循环区分
- [[entities/openclaw-prompt-context-harness|深度解析 OpenClaw 在 Prompt / Context / Harness 三个维度中的设计哲学与实践]] — 三维度源码：23模块拼装+自适应分块+双层Memory
- [[entities/agent-harness-12-components-7-decisions|一篇看懂 Agent Harness 的结构！ — 12组件+7决策完整框架]] — harness 12组件框架
- [[entities/code-as-agent-harness-survey|Code as Agent Harness 综述]] — 102页综述
- [[entities/grok-bot-agent-runtime-five-layer-vibecoder-2026|Grok Bot 0.18 运行时重建：Agent 的五层运行时与可靠性协议]] — 145万行bundle重建五层运行时+可靠性协议

## 工程实践
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 源码机制18k主版
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]] — 并发与延迟加载深析
- [[entities/claude-code-agent-teams-task-decomposition-ruofei|Claude Code Agent Teams 实战：怎么拆任务、控权限、收证据]] — 拆任务控权限
- [[entities/yidian-tianxia-context-engineering-agentic-ai-qcon|一点天下：Context Engineering 与 Agentic AI (QCon)]] — 7114字最全六层上下文版
- [[entities/impeccable-frontend-design-skill-harness-vibecoder|Impeccable：把 AI 前端设计变成可检查的工作流 — 33.4k Star 开源项目深度分析]] — Impeccable四层架构9210字rv9全版
- [[entities/subagents-详解claude-code-如何避免上下文污染|Subagents 详解：Claude Code 如何避免上下文污染]] — 上下文卫生subagent详解
- [[entities/tmall-marketing-ai-workflow-best-practices|淘天营销中后台生码工作流最佳实践]] — 10826字最全生码工作流
- [[entities/hermes-agent-kanban-deep-test-by-wjjagi-2026|Hermes-Agent 官方 Kanban 深度实测：让商业 CLI 工具当 Orchestrator]] — Kanban深度实测+七条bug 7101字rv9全版
- [[entities/精选-8-个-ui-设计师必备的-ai-智能体技能agent-skills|精选 8 个 UI 设计师必备的 AI 智能体技能（Agent Skills）]] — 三层技能架构persist设计
- [[entities/claude-code-architecture|Claude Code 架构解析]] — 架构解析母篇

## 延伸导航
- [[moc/agent-memory-architecture-decision-points|Agent Memory 架构选择的关键决策点是什么？]]
- [[moc/agent-engineering-guide|Agent 工程全景指南]]

## 关联

- 同题异语种孪生页：[[entities/构建基于多智能体架构的深度思考交易系统-v2]]（归并候选，提案卡 #11 批1）
