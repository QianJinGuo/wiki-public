---

title: "Agent Memory 架构本质"
created: 2026-04-27
updated: 2026-09-07
type: entity
tags: [agent-memory, memory-system, belief-tracking, context-management, agent-architecture, knowledge-governance, harness]
sources: [raw/articles/agent-memory-architecture-essence, raw/articles/self-improving-memory-for-agents]
review_value: 8
review_confidence: 9
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.85: 与04-30版memory本质重复; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Agent Memory 架构本质

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.85**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/agent-memory-architecture.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agentscope-java-harness-framework-enterprise-distributed|AgentScope Java Harness Framework 2.0 — 企业级 Agent 分布式场景的 Harness 实现 (Java 2.0 重大升级)]] — AgentScope Java全版
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计与实现：生产级 Agent 系统落地指南]] — 七层金字塔生产指南
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/ai-memory-architecture-deep-dive|AI Memory Architecture: Deep Dive]] — 29k记忆架构深度
- [[entities/ai-coding-agent-memory-system|AI Coding Agent 记忆系统]] — 分层记忆设计
- [[entities/nvidia-agentic-systems-extreme-co-design|Building for the Rising Complexity of Agentic Systems with Extreme Co-Design]] — 三种交互模式+33分钟真实trace+prompt caching挑战
- [[entities/self-harness-shanghai-ai-lab-agent-improves-harness|Self-Harness：上海AI Lab 提出的 Agent 自我改进 Harness 范式]] — Self-Harness范式14191字深度
- [[entities/prime-agent-self-improving-rlm-agent|Prime Agent — 以 RLM + Continual Harness 双抽象为核心的自改进编码 Harness]] — RLM+Continual Harness：harness状态可CRUD

## 工程实践
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei|'长周期 Agent 详解：从 Ralph Loop 到可接管 Harness']] — 三类漂移+5张卡治理12390字rv10全版
- [[entities/loop-engineering-addy-osmani-challengehub|Loop Engineering:不再写提示词,而是设计替你写提示词的循环——先写刹车再写循环（19 来源深度合并：Addy Osmani / Boris Cherny+Peter Steinberger / 教科书 / 若飞 工程现场 / TechFarrari 批判 / 若飞 实用指南 / 爱范儿 科普批判 / AllenTang Karpathy 尺子 / winty 7架构中文主流视角 / AutoResearch 5 决策 / 三层结构 + 三款产品对比 + Ralph Loop + 准备度总表 / Shubham Saboo PM 视角 / 若飞 吴恩达三层Loop）]] — 19来源合并Loop Engineering巨著73742字rv10
- [[entities/karpathy-vibe-coding-agentic-engineering-v4|Karpathy 最新访谈：从 Vibe Coding 到 Agentic Engineering]] — v4 8090字rv10：可验证性上限+MenuGen警示
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 源码机制18k主版
- [[entities/claude-code-openclaw-memory-comparison|Claude Code Openclaw Memory Comparison]] — 记忆系统对比rv9
- [[entities/harness-engineering-comprehensive-guide-conardli|Harness Engineering 综合性指南（ConardLi 系列 · 含 Beautiful Article 实证 + Reacticle 协议）]] — ConardLi六层架构14634字rv9
- [[entities/cpu-cache-analogy-agent-context-management-liwen|CPU 缓存类比下的 Agent 上下文管理：L1/L2/L3 层级架构与 execute_code 单工具设计]] — L1/L2/L3缓存类比
- [[entities/claude-fable-5-agent-runtime-contract-ruofei-2026|Fable 5 的信号:Agent 开始拼 Runtime — 架构师若飞的 Runtime Contract 工程化拆解]] — Runtime Contract拆解
- [[entities/claude-code-openclaw-usage-ettin|Claude Code Openclaw Usage Ettin]] — Ettin rerank集成
- [[entities/ruofei-personal-ai-workbench-18-actions|Personal AI 工作台：Claude 18 动作框架]] — Personal Harness六层工作台：环境工程>提示词技巧
- [[entities/frontend-ai-coding-problem-to-solution-taobao|场景营销前端 AI Coding — 从问题到方案]] — 注意力坍塌+外置DeepResearch分离
- [[entities/harness-engineering-systematic-explainer|Harness Engineering 系统性解读]] — 李宏毅课程解读7933字最全版
- [[entities/tencent-token-optimization-agent-architecture|腾讯 Token 优化实战 — 省 Token 和用好 AI 是同一件事]] — context rot四步工程化
- [[entities/claude-code-27-tips-engineering-upgrade-jiagoux-2026|Claude Code 27 条技巧：从工具清单到工程升级路径]] — 27技巧全版
- [[entities/如何利用-agentcore-openviking-快速搭建具备高效记忆的-agent|如何利用 AgentCore + OpenViking 快速搭建具备高效记忆的 Agent]] — 双方案记忆选型四维
- [[entities/百亿补贴-c-端-ai-coding-实战基于-sdd-的服务端-ai-coding-实践|百亿补贴 C 端 AI Coding 实战：基于 SDD 的服务端 AI Coding 实践]] — spec唯一真相源自进化记忆
