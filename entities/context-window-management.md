---

title: "Agent 上下文窗口管理对比"
created: 2026-04-27
updated: 2026-09-07
type: entity
tags: [agent, context-window, openclaw, claude-code, memory, pi-mono, memory-management, compaction]
sources:
  - raw/articles/context-window-management-comparison
related:
  - "concepts/claude-code-deep-architecture-analysis"
  - "concepts/openclaw-architecture"
  - "concepts/hermes-agent"
  - "entities/agent-harness-context-management-working-set"
  - "entities/agent-memory-architecture"
  - "entities/agent-context-management-architecture-patterns"
review_value: 9
review_confidence: 9
provenance_state: inferred
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 四框架对比重复版; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Agent 上下文窗口管理对比

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/context-window-management.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/harness-engineering-7-layers-openclaw-hermes-claude-code-p1anu|Harness 到底是什么？看看 OpenClaw、Hermes、Claude Code 的演绎吧]] — 三框架演绎七层模型12857字rv9
- [[entities/skill-system-design-three-way-comparison|AI Agent 架构设计（七）：Skills 系统设计（OpenClaw、Claude Code、Hermes Agent 对比）]] — 三框架skill系统设计对比
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/open-claw-tool-bus-subagent-architecture|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]] — 薄抽象显式控制流8802字rv9全版
- [[entities/tencentdb-agent-memory-hierarchical|TencentDB Agent Memory：符号化短期记忆+分层式长期记忆]] — 8661字最全分层记忆版
- [[entities/openclaw-prompt-context-harness|深度解析 OpenClaw 在 Prompt / Context / Harness 三个维度中的设计哲学与实践]] — 三维度源码：23模块拼装+自适应分块+双层Memory
- [[entities/how-ai-agent-memory-works|How AI Agent Memory Works]] — 记忆五层+六架构权衡科普
- [[entities/hermes-agent-vs-openclaw-comparison|Hermes Agent 为什么火了？和 OpenClaw 龙虾比一比]] — 爱马仕vs龙虾：控制面vs成长型定位对比
- [[entities/agent-memory-architecture-past-influence-future-ruofei|Agent 记忆架构：先别急着把 Memory 当数据库]] — 记忆影响未来的治理
- [[entities/openclaw-agent-loop-design-patterns|OpenClaw 与 Claude Code 的 Agent Loop 设计范式]] — 五级跃迁史+循环管控三硬约束5696字全版
- [[entities/agentmemory-source-analysis-coding-agent-local-memory|AgentMemory 源码分析：给 Coding Agent 装上本地长期记忆]] — 源码级解析互补
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]] — agent核心组件实现
- [[entities/claude-code-7-layer-memory-architecture|Claude Code 七层记忆架构]] — 七层防御金字塔
- [[entities/claude-code-and-what-comes-next|Claude Code and What Comes Next]] — 压缩/Skills/Subagents
- [[entities/openclaw-hermes-source-code-agent-architecture-review|OpenClaw与Hermes源码架构对比]] — 双框架源码对比：OpenClaw四亮点+Hermes四补充

## 工程实践
- [[entities/claude-code-openclaw-memory-comparison|Claude Code Openclaw Memory Comparison]] — 记忆系统对比rv9
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 提示词体系源码解析]] — 六大prompt模块全版
- [[entities/memos-hermes-plugin|MemOS Hermes 记忆插件]] — MemOS插件：智能去重+混合检索7225字
- [[entities/coze-3-multimagent-team-orchestration-wangheige|扣子 3.0 多 Agent 协同实战：指挥所有 Agent 的 Agent + 5 人团队 6 步流水线]] — 三案例实战报告
- [[entities/claude-code-openclaw-usage-ettin|Claude Code Openclaw Usage Ettin]] — Ettin rerank集成
- [[entities/claude-code-agent-memory-four-levels-analysis|Claude Code Agent Memory Systems — L0~L3 四层记忆方案]] — L0-L3演化
- [[entities/imclaw通过微信飞书操控claude-code-coodex-gemini-clipi-agent蜂群|IMClaw：通过微信/飞书操控ClaudeCode/Codex/GeminiCLI/Pi Agent蜂群]] — ACP协议N+M解耦+网关架构6224字全版
- [[entities/aliyun-mse-ai-task-scheduling-agent-sandbox-cost-90-percent|阿里云 MSE AI 任务调度 + Agent Sandbox：动态休眠/唤醒 OpenClaw Agent 成本下降 90%+]] — 休眠唤醒短条borderline
- [[entities/harness-engineering-14-step-roadmap|Harness 工程 14 步路线图：从单 Agent 到自改进系统]] — 三层楼模型14步渐进构建

## 延伸导航
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]]
- [[moc/openclaw-architecture|OpenClaw 的架构设计为什么值得研究？它与 Hermes/Claude Code 的核心差异？]]
- [[moc/agent-memory-architecture-decision-points|Agent Memory 架构选择的关键决策点是什么？]]
