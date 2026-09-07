---
title: "Anthropic 14 个 Agent Skills 设计模式"
created: 2026-05-10
updated: 2026-09-07
type: entity
tags: [anthropic, skill, agent, design-patterns, best-practices]
sources: [raw/articles/anthropic-agent-skills-design-patterns-14]
review_value: 7
review_confidence: 7
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: 与18k版skill模式重复; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Anthropic 14 个 Agent Skills 设计模式

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/anthropic-agent-skills-design-patterns-14.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/muse-autoskill-bytebrain-self-evolving-agent-arxiv-2605-27366|MUSE-Autoskill：字节 ByteBrain 自进化 Agent 五阶段技能生命周期，arXiv 2605.27366]] — 五阶段技能生命周期，自生成87.94%超人类68.40%
- [[entities/harness-engineering-7-layers-openclaw-hermes-claude-code-p1anu|Harness 到底是什么？看看 OpenClaw、Hermes、Claude Code 的演绎吧]] — 三框架演绎七层模型12857字rv9
- [[entities/skill-system-design-three-way-comparison|AI Agent 架构设计（七）：Skills 系统设计（OpenClaw、Claude Code、Hermes Agent 对比）]] — 三框架skill系统设计对比
- [[entities/anthropic-google-agent-skills-design-patterns|从 Anthropic 到 Google：Agent Skills 进入设计模式阶段]] — ADK五模式对比
- [[entities/huggingface-ai-agent-glossary-model-scaffolding-harness-tool-skill-subagent|Hugging Face AI Agent 术语表：Model / Agent / Scaffolding / Harness / Context Engineering / Policy / Tool / Skill / Sub-agent 完整区分]] — HF术语表16399字：Scaffolding/Harness/Policy辨析
- [[entities/agent-skills-comprehensive-survey|Agent Skills 系统性综述：表示→获取→检索→进化]] — skill综述三元组
- [[entities/mythos_offensive_security_xbow_evaluatio|Mythos for Offensive Security: XBOW's Evaluation]] — XBOW评测6874字rv9全版：live-site悖论
- [[entities/skill-self-evolution-three-approaches|Skill自进化三路线：Trace2Skill归纳法 / EvoSkill验证闭环 / SkillOpt训练范式]] — 自进化三路线对比解析
- [[entities/anthropic-n-days-frontier-agent-vulnerability-research|Anthropic N-days: Frontier Agent Vulnerability Research]] — N-day研究
- [[entities/open-claw-tool-bus-subagent-architecture|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]] — 薄抽象显式控制流8802字rv9全版
- [[entities/wangyunhe-harness-optimization-agentsoul|王云鹤眼中的Harness：复杂优化问题，AGI灵魂争夺之战]] — Agent=Models+Harness联合优化
- [[entities/agent-skill-spec-building-design-patterns|Agent Skill 规范、构建与设计模式]] — skill规范与模式

## 工程实践
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris访谈rv10全版
- [[entities/harness-generator-evaluator-anthropic|Claude Harness 设计：Generator-Evaluator 架构与 Context Reset 演进]] — Generator-Evaluator+context reset 10329字rv9全版
- [[entities/claude-code-openclaw-memory-comparison|Claude Code Openclaw Memory Comparison]] — 记忆系统对比rv9
- [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2|Qoder Skills 完全指南 + Agent Skill 迭代式编写 — AI 按你的标准执行]] — 菜单菜谱比喻+三级渐进披露18168字rv9全版
- [[entities/impeccable-frontend-design-skill-harness-vibecoder|Impeccable：把 AI 前端设计变成可检查的工作流 — 33.4k Star 开源项目深度分析]] — Impeccable四层架构9210字rv9全版
- [[entities/claude-code-first-year-retrospective-boris-cat-2026|Claude Code 一周年回顾：Boris Cherny + Cat Wu 的完整时间线]] — 一周年回顾14k主版
- [[entities/tencent-skill-writing-complete-playbook-jackjchou|鹅厂 Skill 写作完整 Playbook：14 章节 end-to-end 实战 + 工程化评估（腾讯一线踩坑 + Anthropic 官方做法整合）]] — 14章节skill写作playbook
- [[entities/anthropic-mcp-revisited-tool-search-code-orchestration|Anthropic 最新博客：MCP 没死，它又来了]] — MCP三条路
- [[entities/harness-design-long-running-apps|长时间运行应用的 Harness 设计]] — GAN式generator/evaluator长时harness 9745字
- [[entities/anthropic-14-skill-patterns-best-practices|Anthropic 官方技能最佳实践：14 个可复用的 Agent Skills 设计模式]] — 14个skill模式全版
- [[entities/anthropic-claude-skill-9-categories-datawhale-2026|Anthropic Claude Skill 9 类任务分类法]] — 9类分类法

## 延伸导航
- [[moc/claude-code-complete-guide|Claude Code 生态完全指南]]
