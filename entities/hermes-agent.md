---

title: "Hermes Agent"
created: 2026-04-24
updated: 2026-09-07
type: entity
tags: [hermes-agent, nous-research, agent, self-evolving, open-source, skill]
sources: [raw/articles/agent-tools-research]
review_value: 7
review_confidence: 8
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: thin
review_note: "judged thin-0.75: 泛介绍卡2918字，同族已有深度解析; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Hermes Agent

> 本页原内容在 2026-09-07 质量闭环中判定为 **thin-0.75**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/hermes-agent.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/skill-os-learning-skill-curation-self-evolving-agents|SkillOS: Learning Skill Curation for Self-Evolving Agents]] — SkillOS策展RL架构清晰rv10
- [[entities/hermes-agent-skill-crossover-optimization|Hermes Agent Skill 互优化：SkillEvolver × Darwin × EmbodiSkill 4 轮闭环]] — SkillEvolver×Darwin×EmbodiSkill互优化13412字
- [[entities/muse-autoskill-bytebrain-self-evolving-agent-arxiv-2605-27366|MUSE-Autoskill：字节 ByteBrain 自进化 Agent 五阶段技能生命周期，arXiv 2605.27366]] — 五阶段技能生命周期，自生成87.94%超人类68.40%
- [[entities/skill-system-design-three-way-comparison|AI Agent 架构设计（七）：Skills 系统设计（OpenClaw、Claude Code、Hermes Agent 对比）]] — 三框架skill系统设计对比
- [[entities/gbrain|GBrain — YC CEO Garry Tan 的 Postgres-native AI 第二大脑：5 大设计决策 + 零 LLM 知识图谱 + 8 阶段检索 + Brain⊥Source 正交维度]] — GBrain五大设计决策+8阶段检索9529字rv9
- [[entities/hermes-agent-deep-dive|Hermes Agent 深度解析（阿里云/飞樰）]] — 自进化内外双路径+四维工程7934字rv9全版
- [[entities/llm-agent脚手架如何具备自进化能力以hermes-agent为例|LLM agent脚手架如何具备自进化能力？——以hermes agent为例]] — Hermes自进化15402字最全版
- [[entities/huggingface-ai-agent-glossary-model-scaffolding-harness-tool-skill-subagent|Hugging Face AI Agent 术语表：Model / Agent / Scaffolding / Harness / Context Engineering / Policy / Tool / Skill / Sub-agent 完整区分]] — HF术语表16399字：Scaffolding/Harness/Policy辨析
- [[entities/agent-skills-comprehensive-survey|Agent Skills 系统性综述：表示→获取→检索→进化]] — skill综述三元组
- [[entities/harness-engineering-paradigm-comprehensive-2026|Harness Engineering 综合论述：为什么 2026 年真正重要的是它（含 ECC 开源实现案例）]] — 综合论述17305字含ECC案例rv9
- [[entities/skill-self-evolution-three-approaches|Skill自进化三路线：Trace2Skill归纳法 / EvoSkill验证闭环 / SkillOpt训练范式]] — 自进化三路线对比解析
- [[entities/flow2spec-structured-knowledge-routing-ctrip-2026|Flow2Spec：开发过程自然长出知识图谱的 Agent 工程框架]] — Flow2Spec知识路由协议，渐进式上下文
- [[entities/hermes-agent-closed-learning-loop|Hermes Agent 闭环学习机制]] — 闭环学习飞轮+Nudge触发+spawn_background_review
- [[entities/skillx-hierarchical-skill-library|SkillX — 层次化技能知识库]] — 三层技能库最全版本
- [[entities/agent-tools-research|深度解析 Hermes Agent 如何实现自进化及其 Prompt / Context / Harness 的设计实践]] — Hermes自进化解析

## 工程实践
- [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2|Qoder Skills 完全指南 + Agent Skill 迭代式编写 — AI 按你的标准执行]] — 菜单菜谱比喻+三级渐进披露18168字rv9全版
- [[entities/skill-design-spec-8-block-checklist-winty|企业级 Skill 8 块最小骨架 + 8 条 checklist 设计规范]] — 8块骨架checklist设计规范
- [[entities/skill-hub-organization-asset-winty|Skill Hub：企业级 AI 经验资产化的关键（组织能力视角）— winty 前端Q 3 篇合集：组织资产 + 质量门禁 4 关 + 生命周期 6 阶段治理]] — Skill组织资产化治理五件事
- [[entities/impeccable-frontend-design-skill-harness-vibecoder|Impeccable：把 AI 前端设计变成可检查的工作流 — 33.4k Star 开源项目深度分析]] — Impeccable四层架构9210字rv9全版
- [[entities/wiki-evolver|Wiki Evolver]] — 知识库涌现层元系统
- [[entities/gaode-saojie-image-selection-hermesagent-vlm-production-2026|高德扫街榜 HermesAgent 配图系统：VLM + Skill + 语言驱动的生产级 Agent 架构]] — 确定性流水线+Agent巧活，提效48倍rv9
- [[entities/claude-design-skill-web-design-engineer|我把 Claude Design 做成了 Skill，人人都能成为顶级网站设计师]] — Claude Design拆解全版
- [[entities/memos-hermes-plugin|MemOS Hermes 记忆插件]] — MemOS插件：智能去重+混合检索7225字
- [[entities/conardli-skills-7k-star-open-source-agent-2026|啊？我刚开源的 Skills 已经 7K Star 了？！]] — garden-skills全版
