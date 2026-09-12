---

title: "Harness Engineering 系统梳理"
created: 2026-04-30
updated: 2026-09-10
type: entity
tags: [harness-engineering, agent, control-loop, context-engineering, generator-evaluator]
sources:
  - raw/articles/harness-engineering-systematic-explainer
  - raw/articles/claude-code-engineering-truth-1.6-98.4
review_value: 7
review_confidence: 8
review_recommendation: strong
review_stars: 4
strategic_context: [[queries/research-frontier-map|Frontier 1 — Harness 从执行框架进化成策略层]]
related:
  - concepts/harness-engineering-framework
  - entities/agent-engineering-principles-architecture-practice
  - concepts/openclaw-architecture
provenance_state: inferred
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.75: 同源课程解读5341字版，留7933字版; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Harness Engineering 系统梳理

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.75**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/harness-engineering-systematic-framework.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-harness-engineering-survey-2026|Agent Harness Engineering: A Survey]] — harness工程survey
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计与实现：生产级 Agent 系统落地指南]] — 七层金字塔生产指南
- [[entities/harness-engineering-7-layers-openclaw-hermes-claude-code-p1anu|Harness 到底是什么？看看 OpenClaw、Hermes、Claude Code 的演绎吧]] — 三框架演绎七层模型12857字rv9
- [[entities/hermes-agent-deep-dive|Hermes Agent 深度解析（阿里云/飞樰）]] — 自进化内外双路径+四维工程7934字rv9全版
- [[entities/huggingface-ai-agent-glossary-model-scaffolding-harness-tool-skill-subagent|Hugging Face AI Agent 术语表：Model / Agent / Scaffolding / Harness / Context Engineering / Policy / Tool / Skill / Sub-agent 完整区分]] — HF术语表16399字：Scaffolding/Harness/Policy辨析
- [[entities/harness-engineering-self-improvement-survey-lilian-weng|Harness Engineering for Self-Improvement — 翁荔 Lilian Weng 系统梳理 Harness 自我提升研究全景]] — 翁荔RSI全景：ACE→MCE→DGM谱系
- [[entities/agentic-loop-engineering-handbook-empirical-framework|Agentic Loop Engineering 工程手册：17 种 Loop 工程化技术的可复现实证框架]] — 17种loop实证
- [[entities/production-harness-12-components-framework-comparison|生产级 Harness 的 12 大组件以及主流框架对比]] — 12组件+OS类比：TerminalBench 30名外→第5名9722字rv9
- [[entities/wangyunhe-harness-optimization-agentsoul|王云鹤眼中的Harness：复杂优化问题，AGI灵魂争夺之战]] — Agent=Models+Harness联合优化
- [[entities/is-grep-all-you-need-pwc-retrieval-harness-coupling|Is Grep All You Need? — 检索 × Harness × 交付方式耦合三元组（PwC 论文 arXiv 2605.15184 解读）]] — grep vs vector×harness×交付耦合三元组rv9
- [[entities/hidden-technical-debt-agent-harness|Hidden Technical Debt of AI Systems: Agent Harness]] — Harness层五维技术债
- [[entities/ai-coding-entropy-framework-baidu-geek-2026|AI Coding 的底层框架：一切优化都是在对抗熵增——信息论视角]] — 信息论统一框架
- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道|深入理解 Claude Code 源码中的 Agent Harness 构建之道]] — 16095字源码8步循环
- [[entities/刚刚翁荔博客又上新通过harness工程实现ai自我提升|刚刚，翁荔博客又上新：通过Harness工程实现AI自我提升]] — RSI路径与七挑战

## 工程实践
- [[entities/harness-engineering|'Harness Engineering：AI 从]] — 六层架构+七大反模式+分级决策树19712字rv9
- [[entities/harness-generator-evaluator-anthropic|Claude Harness 设计：Generator-Evaluator 架构与 Context Reset 演进]] — Generator-Evaluator+context reset 10329字rv9全版
- [[entities/claude-code-agentic-harness-design-patterns|深度拆解 Claude Code：12 个可复用的 Agentic Harness 设计模式]] — 12个harness模式
- [[entities/harness-engineering-comprehensive-guide-conardli|Harness Engineering 综合性指南（ConardLi 系列 · 含 Beautiful Article 实证 + Reacticle 协议）]] — ConardLi六层架构14634字rv9
- [[entities/impeccable-frontend-design-skill-harness-vibecoder|Impeccable：把 AI 前端设计变成可检查的工作流 — 33.4k Star 开源项目深度分析]] — Impeccable四层架构9210字rv9全版
- [[entities/subagents-详解claude-code-如何避免上下文污染|Subagents 详解：Claude Code 如何避免上下文污染]] — 上下文卫生subagent详解
- [[entities/super-individual-to-super-organization-tencent-research-2026|超级个体到超级组织：李志飞 CodeBanana 组织转型实践]] — 超级组织转型CodeBanana
- [[entities/claude-code-source-leak-lifecycle-analysis|CLAUDE.md]] — 8步生命周期10k
- [[entities/qq-music-harness-engineering-monorepo-microservices|QQ音乐 Harness Engineering 实践（大仓多服务场景）]] — 代码产出=AI能力×上下文质量（乘法）15606字rv9
- [[entities/tdsql-harness-subtraction-l0-l3-tencent-2026-08-06|Harness 减法工程——删掉 61% 之后什么该留（L0-L3 四层归属）]] — 减法工程L0-L3四层归属

## 关联

- 同题异语种孪生页：[[entities/harness-engineering耗时一周我是如何将应用的ai-coding率提升至90的]]（归并候选，提案卡 #11 批1）
