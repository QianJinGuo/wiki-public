---

title: "Prompt Caching 工程实践 — Anthropic Claude Code 经验总结"
created: 2026-05-06
updated: 2026-09-07
type: entity
tags: [claude-code, prompt-caching, anthropic, agent-architecture, context-management, engineering]
sources: [raw/articles/anthropic-prompt-caching-claude-code-agihunt]
review_value: 8
review_confidence: 8
review_recommendation: strong
review_stars: 5
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 缓存经验重复版; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Prompt Caching 工程实践 — Anthropic Claude Code 经验总结

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/anthropic-prompt-caching-claude-code.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/harness-engineering-7-layers-openclaw-hermes-claude-code-p1anu|Harness 到底是什么？看看 OpenClaw、Hermes、Claude Code 的演绎吧]] — 三框架演绎七层模型12857字rv9
- [[entities/claude-opus-4-7-launch|Claude Opus 4.7 发布分析]] — 4.7发布分析
- [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]] — 6588字最全发布分析
- [[entities/context-window-management-comparison|Context Window Management Comparison]] — 四框架对比rv9
- [[entities/open-claw-tool-bus-subagent-architecture|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]] — 薄抽象显式控制流8802字rv9全版
- [[entities/wangyunhe-harness-optimization-agentsoul|王云鹤眼中的Harness：复杂优化问题，AGI灵魂争夺之战]] — Agent=Models+Harness联合优化
- [[entities/nvidia-agentic-systems-extreme-co-design|Building for the Rising Complexity of Agentic Systems with Extreme Co-Design]] — 三种交互模式+33分钟真实trace+prompt caching挑战
- [[entities/ai-agent-tool-count-trap|AI Agent工具数量陷阱——5个边界清楚的工具胜过20个模糊工具]] — 工具税数据与机制

## 工程实践
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris访谈rv10全版
- [[entities/claude-code-source-deep-dive-warrior|Claude Code 源码深度解析（13 核心机制）]] — 13机制rv10
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 源码机制18k主版
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]] — 并发与延迟加载深析
- [[entities/harness-generator-evaluator-anthropic|Claude Harness 设计：Generator-Evaluator 架构与 Context Reset 演进]] — Generator-Evaluator+context reset 10329字rv9全版
- [[entities/claude-code-openclaw-memory-comparison|Claude Code Openclaw Memory Comparison]] — 记忆系统对比rv9
- [[entities/anthropic-ai-native-startup-handbook|Anthropic发布「AI原生创业公司」手册：涵盖全流程四大核心阶段，一人公司法典来了]] — 创业四阶段手册
- [[entities/opus-4-7-launch-claude-code-best-practices-wechat|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]] — Opus 4.7核心变化+CC六新功能14779字rv9
- [[entities/anthropic-95pct-data-analysis-skill-stack-architecture|Anthropic 内部 95% 数据分析自动化：分析 Agent 技术栈 + Skill 框架（21%→95% 准确率）]] — 95%技术栈26k
- [[entities/harness-engineering-comprehensive-guide-conardli|Harness Engineering 综合性指南（ConardLi 系列 · 含 Beautiful Article 实证 + Reacticle 协议）]] — ConardLi六层架构14634字rv9
- [[entities/claude-code-performance-benchmarking|Claude Code 性能基准评测]] — 性能指标体系
- [[entities/claude-code-first-year-retrospective-boris-cat-2026|Claude Code 一周年回顾：Boris Cherny + Cat Wu 的完整时间线]] — 一周年回顾14k主版
- [[entities/cpu-cache-analogy-agent-context-management-liwen|CPU 缓存类比下的 Agent 上下文管理：L1/L2/L3 层级架构与 execute_code 单工具设计]] — L1/L2/L3缓存类比
- [[entities/claude-design-skill-web-design-engineer|我把 Claude Design 做成了 Skill，人人都能成为顶级网站设计师]] — Claude Design拆解全版
- [[entities/anthropic-prompt-caching-claude-code-agihunt|Anthropic 最新博客：Prompt Caching 是构建 Claude Code 的一切]] — 缓存9条12k全版
- [[entities/claude-code-multi-agent-harness-source-analysis|Claude Code 多 Agent Harness 源码拆解：留纸条、抠上下文、抠缓存、捆手脚]] — 留纸条抠上下文

## 延伸导航
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]]
