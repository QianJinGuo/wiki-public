---

title: "Agent架构关键变化：Harness正在成为新后端"
created: 2026-05-10
updated: 2026-09-10
type: entity
tags: [ai-agent, engineering, agent-tools, wechat]
review_value: 6
review_confidence: 7
sources:
  - raw/articles/agent架构关键变化harness正在成为新后端
score_validated: 2026-09-05
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.85: 与17k全版条重复; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Agent架构关键变化：Harness正在成为新后端

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.85**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/agent架构关键变化harness正在成为新后端.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-harness-engineering-survey-2026|Agent Harness Engineering: A Survey]] — harness工程survey
- [[entities/huggingface-ai-agent-glossary-model-scaffolding-harness-tool-skill-subagent|Hugging Face AI Agent 术语表：Model / Agent / Scaffolding / Harness / Context Engineering / Policy / Tool / Skill / Sub-agent 完整区分]] — HF术语表16399字：Scaffolding/Harness/Policy辨析
- [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]] — 6588字最全发布分析
- [[entities/skill-formal-theory-survey-10papers|10篇论文看懂AI Agent Skill：表示、执行、评估与进化]] — 技能六元组形式化综述
- [[entities/hermes-agent-goal-runtime-architecture|Hermes Agent /goal 长任务运行时架构]] — GoalState四部件+Judge保守优先5161字全版
- [[entities/国产顶尖模型-benchmark-评分那么高可实际效果为什么差看完-anthropic-这篇博客刷分的因素太单一了|国产顶尖模型 benchmark 评分那么高，可实际效果为什么差？看完 Anthropic 这篇博客，刷分的因素太单一了]] — 评测环境系统性偏差
- [[entities/token级精准控制生成长度3b模型击败gpt-54claude|token级，精准控制生成长度：3B模型击败GPT 5.4、Claude]] — 长度即值函数LenVM

## 工程实践
- [[entities/harness-engineering|'Harness Engineering：AI 从]] — 六层架构+七大反模式+分级决策树19712字rv9
- [[entities/opus-4-7-launch-claude-code-best-practices-wechat|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]] — Opus 4.7核心变化+CC六新功能14779字rv9
- [[entities/yidian-tianxia-context-engineering-agentic-ai-qcon|一点天下：Context Engineering 与 Agentic AI (QCon)]] — 7114字最全六层上下文版
- [[entities/harness-engineeringai-能在真正出事会炸的后端系统里写代码吗|'Harness Engineering：AI 能在真正]] — 腾讯CDN LEGO五层架构+对抗式CR全版
- [[entities/complexity-ratchet-garry-tan|你的AI代码越写越乱，他72小时合了14个PR每个都更好——差距只在一个机制]] — 棘轮机制rv9主版
- [[entities/claude-design-skill-web-design-engineer|我把 Claude Design 做成了 Skill，人人都能成为顶级网站设计师]] — Claude Design拆解全版
- [[entities/qunar-ai-coding-platform-practice-l0-l5-harness|去哪儿网 AI Coding 研发平台实践：L0-L5 自动化分级 + Harness 四把锁 + QunarDevCenter + 天弦 QDO]] — L0-L5分级+四把锁+度量体系2289字rv9
- [[entities/cursor-复盘-harness模型决定能力上限harness-决定生产下限|Cursor 复盘 Harness：模型决定能力上限，Harness 决定生产下限]] — Cursor复盘主版
- [[entities/three-tools-in-one-gstack-superpowers-openspec-engineering-ai-coding|三器合一：gstack + Superpowers + OpenSpec 工程化 AI 编程实战]] — gstack变体四串联点
- [[entities/harness不是目的知识才是护城河-一个ai工程交付团队的知识沉淀实践|Harness不是目的，知识才是护城河 —— 一个AI工程交付团队的知识沉淀实践]] — 知识五层存储×五类型×三成熟度15831字
- [[entities/别再把上下文当聊天记录|别再把上下文当聊天记录]] — 上下文是工作空间四家趋同
- [[entities/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践|告别“氛围编程”：基于 Harness 治理和 SDD 的团队级 AI 研发范式演进与实践]] — 确定性承重层四步骤
- [[entities/我把-karpathy-的-autoresearch-搬到了软件开发领域效果炸了|我把 Karpathy 的 AutoResearch 搬到了软件开发领域，效果炸了]] — val loss换多维评分
- [[entities/精选-10-个开发者常用的-ai-智能体技能agent-skills|精选 10 个开发者常用的 AI 智能体技能（Agent Skills）]] — 四类技能质量筛选节点
- [[entities/你写的-skill及格了吗|你写的 Skill，及格了吗？]] — D1元数据定生死评估框架
- [[entities/ollama-已经不是-2024-年那个了一键配齐-claude-codecodexopenclaw|Ollama 已经不是 2024 年那个了！一键配齐 Claude Code/Codex/OpenClaw]] — GGUF解锁+launch：从Docker到入口层
- [[entities/从不敢发到天天发ai-agent-时代的-cicd-生存指南|从「不敢发」到「天天发」：AI Agent 时代的 CI/CD 生存指南]] — 分层门禁逃生舱机制

## 关联

- 同题异语种孪生页：[[entities/harness-engineering-reliable-long-term-agent]]（归并候选，提案卡 #11 批1）
- 同题异语种孪生页：[[entities/agentcore-managed-harness]]（归并候选，提案卡 #11 批1）

