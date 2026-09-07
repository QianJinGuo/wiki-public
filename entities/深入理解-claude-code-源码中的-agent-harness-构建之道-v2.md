---
title: "深入理解 Claude Code 源码中的 Agent Harness 构建之道"
created: 2026-06-11
updated: 2026-09-07
type: entity
tags: [claude-code, harness, agent-architecture, open-source, anthropic, source-code]
sources: [raw/articles/深入理解-claude-code-源码中的-agent-harness-构建之道-v2.md]
provenance_state: raw-linked
review_value: 8
confidence: 0.8
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 同文较短版留16095字; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# 深入理解 Claude Code 源码中的 Agent Harness 构建之道

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/深入理解-claude-code-源码中的-agent-harness-构建之道-v2.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/skillopt|SkillOpt]] — SkillOpt最全8328字rv10
- [[entities/harness-engineering-7-layers-openclaw-hermes-claude-code-p1anu|Harness 到底是什么？看看 OpenClaw、Hermes、Claude Code 的演绎吧]] — 三框架演绎七层模型12857字rv9
- [[entities/claude-opus-4-7-launch|Claude Opus 4.7 发布分析]] — 4.7发布分析
- [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]] — 6588字最全发布分析
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/harness-engineering-paradigm-comprehensive-2026|Harness Engineering 综合论述：为什么 2026 年真正重要的是它（含 ECC 开源实现案例）]] — 综合论述17305字含ECC案例rv9
- [[entities/wangyunhe-harness-optimization-agentsoul|王云鹤眼中的Harness：复杂优化问题，AGI灵魂争夺之战]] — Agent=Models+Harness联合优化
- [[entities/ai-agent-tool-count-trap|AI Agent工具数量陷阱——5个边界清楚的工具胜过20个模糊工具]] — 工具税数据与机制
- [[entities/ai-agent-harness-construction-akshay|深度拆解：AI 智能体 Harness 的构造（译）]] — harness构造译全版
- [[entities/prime-agent-self-improving-rlm-agent|Prime Agent — 以 RLM + Continual Harness 双抽象为核心的自改进编码 Harness]] — RLM+Continual Harness：harness状态可CRUD

## 工程实践
- [[entities/loop-engineering-addy-osmani-challengehub|Loop Engineering:不再写提示词,而是设计替你写提示词的循环——先写刹车再写循环（19 来源深度合并：Addy Osmani / Boris Cherny+Peter Steinberger / 教科书 / 若飞 工程现场 / TechFarrari 批判 / 若飞 实用指南 / 爱范儿 科普批判 / AllenTang Karpathy 尺子 / winty 7架构中文主流视角 / AutoResearch 5 决策 / 三层结构 + 三款产品对比 + Ralph Loop + 准备度总表 / Shubham Saboo PM 视角 / 若飞 吴恩达三层Loop）]] — 19来源合并Loop Engineering巨著73742字rv10
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris访谈rv10全版
- [[entities/claude-code-source-deep-dive-warrior|Claude Code 源码深度解析（13 核心机制）]] — 13机制rv10
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 源码机制18k主版
- [[entities/claude-code-openclaw-memory-comparison|Claude Code Openclaw Memory Comparison]] — 记忆系统对比rv9
- [[entities/claude-code-governance-soft-rules|Claude Code 可控性：软规则无法变成硬约束]] — 200k Ghost治理主版
- [[entities/anthropic-ai-native-startup-handbook|Anthropic发布「AI原生创业公司」手册：涵盖全流程四大核心阶段，一人公司法典来了]] — 创业四阶段手册
- [[entities/opus-4-7-launch-claude-code-best-practices-wechat|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]] — Opus 4.7核心变化+CC六新功能14779字rv9
- [[entities/anthropic-95pct-data-analysis-skill-stack-architecture|Anthropic 内部 95% 数据分析自动化：分析 Agent 技术栈 + Skill 框架（21%→95% 准确率）]] — 95%技术栈26k
- [[entities/harness-engineering-comprehensive-guide-conardli|Harness Engineering 综合性指南（ConardLi 系列 · 含 Beautiful Article 实证 + Reacticle 协议）]] — ConardLi六层架构14634字rv9
- [[entities/claude-code-routines-proactive-agent|Claude Code Routines：从工具到队友的主动 Agent 模式]] — Routines三能力
- [[entities/claude-code-seven-customization-methods-anthropic-official|Claude Code 七种自定义方法：官方全景指南]] — 七种自定义对比
- [[entities/claude-code-multi-agent-harness-source-analysis|Claude Code 多 Agent Harness 源码拆解：留纸条、抠上下文、抠缓存、捆手脚]] — 留纸条抠上下文

## 延伸导航
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]]
- [[moc/loop-engineering|Loop Engineering 主题地图 (MOC)]]
