---

description: Auto-generated placeholder
title: "Hermes Agent 自进化机制源码解析"
created: 2026-05-08
updated: 2026-09-07
type: entity
tags: [llm-agent, hermes-agent, self-evolving, skill-system, memory-management, prompt-engineering]
review_value: 9
review_confidence: 7
sources:
  - raw/articles/hermes-agent-self-evolving-source-analysis
summary: Hermes Agent 基于 Memory + Skills + Session Search 的自进化机制设计，不依赖模型权重更新，通过 skill 沉淀操作流程、memory 记住偏好、session search 找回历史经验
provenance_state: inferred
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.75: 自进化机制9319字rv9版，留14550字版; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Hermes Agent 自进化机制源码解析

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.75**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/hermes-agent-self-evolving.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/hermes-agent-memory-system-openclaw-comparison|深度拆解 Hermes Agent 记忆系统]] — 记忆成本账四层体系15260字rv10最深版
- [[entities/skill-os-learning-skill-curation-self-evolving-agents|SkillOS: Learning Skill Curation for Self-Evolving Agents]] — SkillOS策展RL架构清晰rv10
- [[entities/hermes-agent-skill-crossover-optimization|Hermes Agent Skill 互优化：SkillEvolver × Darwin × EmbodiSkill 4 轮闭环]] — SkillEvolver×Darwin×EmbodiSkill互优化13412字
- [[entities/agent-memory-modular-framework|Agent Memory 模块化框架与评测：Memory in the LLM Era 4 模块 + 10 方案对比 + 新方法 F1 38.79 + 4 条工程原则]] — 四组件统一框架
- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 六框架实践地图：从算法到系统的长程智能体训练]] — RL六框架地图
- [[entities/hermes-agent-deep-dive|Hermes Agent 深度解析（阿里云/飞樰）]] — 自进化内外双路径+四维工程7934字rv9全版
- [[entities/llm-agent脚手架如何具备自进化能力以hermes-agent为例|LLM agent脚手架如何具备自进化能力？——以hermes agent为例]] — Hermes自进化15402字最全版
- [[entities/agent-memory-storage-six-schools-wiki-compile-vs-raw-data-debate|Agent 记忆存储方案深度洞察：6 大流派分歧、Wiki 编译 vs 原始数据之争、Hermes Agent 启示]] — 六派之争全版
- [[entities/harness-engineering-paradigm-comprehensive-2026|Harness Engineering 综合论述：为什么 2026 年真正重要的是它（含 ECC 开源实现案例）]] — 综合论述17305字含ECC案例rv9
- [[entities/agent-harness-engineering-survey-etcvlovg-taxonomy|Agent Harness Engineering: A Survey — ETCLOVG Taxonomy]] — ETCLOVG分类补充
- [[entities/claude-fable-5-prompt-leak-runtime-control-plane-vibecoder-2026|Claude Fable 5 提示词泄漏 — 1585 行 120K 字符的产品运行时控制平面与安全工程启示]] — 1585行控制平面
- [[entities/ai-memory-architecture-deep-dive|AI Memory Architecture: Deep Dive]] — 29k记忆架构深度
- [[entities/memory-in-the-llm-era-iclr2026|Memory in the LLM Era: Modular Architectures and Strategies in a Unified Framework]] — 四组件统一框架10658字
- [[entities/agent-tools-research|深度解析 Hermes Agent 如何实现自进化及其 Prompt / Context / Harness 的设计实践]] — Hermes自进化解析

## 工程实践
- [[entities/didi-ibg-customer-experience-llm-quality-inspection-3-pipelines|滴滴 IBG 智能客服质检系统：3 管线（意图 86% / 合规 90%+ / VOC）+ 企业 LLM 落地方法论]] — 三管线质检14k
- [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2|Qoder Skills 完全指南 + Agent Skill 迭代式编写 — AI 按你的标准执行]] — 菜单菜谱比喻+三级渐进披露18168字rv9全版
- [[entities/hermes-agent-12-layer-full-configuration-guide|Hermes Agent 满配 12 层配置完整指南（从裸装到 24h Agent 团队）]] — 12层满配指南11566字rv9
- [[entities/skill-design-spec-8-block-checklist-winty|企业级 Skill 8 块最小骨架 + 8 条 checklist 设计规范]] — 8块骨架checklist设计规范
- [[entities/harness-engineering-comprehensive-guide-conardli|Harness Engineering 综合性指南（ConardLi 系列 · 含 Beautiful Article 实证 + Reacticle 协议）]] — ConardLi六层架构14634字rv9
- [[entities/skill-hub-organization-asset-winty|Skill Hub：企业级 AI 经验资产化的关键（组织能力视角）— winty 前端Q 3 篇合集：组织资产 + 质量门禁 4 关 + 生命周期 6 阶段治理]] — Skill组织资产化治理五件事
- [[entities/karpathy-claude-md-rules|Karpathy CLAUDE.md — 四条行为准则让 AI 编程 Agent 减少结构性失败]] — CLAUDE.md四行为准则rv9
- [[entities/claude-code-skills-practical-guide-discovery-frontmatter|Claude Code Skills 实战指南 — 发现机制、编写与安全]] — 发现机制与安全
- [[entities/skill-version-management-semantic-versioning-practices-winty|Skill 版本管理五大原则：从越改越差到持续演进]] — skill语义化版本五原则
- [[entities/chromium-ai-coding-development-system|Chromium AI Coding 开发体系]] — Chromium AI基建
