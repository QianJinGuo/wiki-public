---

title: "你不知道的 Agent：原理、架构与工程实践"
type: entity
tags: [mlops, wechat, llm, ai-agent, engineering]
review_value: 7
sources: [raw/articles/你不知道的-agent原理架构与工程实践]
review_confidence: 8
created: 2026-05-16
updated: 2026-09-07
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: thin
review_note: "judged thin-0.78: AI臆测摘要非原文内容; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# 你不知道的 Agent：原理、架构与工程实践

> 本页原内容在 2026-09-07 质量闭环中判定为 **thin-0.78**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/你不知道的-agent原理架构与工程实践.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/gpt-5-6-preview|GPT-5.6 Preview System Card — Community Detection & Benchmarks]] — GPT-5.6系统卡：三模型+安全评估rv9
- [[entities/senior-engineer-guide-inside-llm|Everything a Senior Engineer Needs to Know About What's Inside an LLM]] — LLM内部机制工程师向讲解
- [[entities/hermes-agent-goal-runtime-architecture|Hermes Agent /goal 长任务运行时架构]] — GoalState四部件+Judge保守优先5161字全版
- [[entities/blog-accelerating-gemini-nano-models-on-pixel-with-frozen-multi-token-prediction|Accelerating Gemini Nano models on Pixel with frozen Multi-Token Prediction]] — MTP端侧全版
- [[entities/lean-scaling|Lean Software Scaling Laws]] — Lean scaling laws研究提案3102字全版
- [[entities/tsinghua-popo-group-prioritized-off-policy-optimization-rlvr|POPO (Group Prioritized Off-Policy Optimization)：清华 RLVR 训练高效组级回放框架]] — 组级回放解耦off-policy
- [[entities/国产顶尖模型-benchmark-评分那么高可实际效果为什么差看完-anthropic-这篇博客刷分的因素太单一了|国产顶尖模型 benchmark 评分那么高，可实际效果为什么差？看完 Anthropic 这篇博客，刷分的因素太单一了]] — 评测环境系统性偏差
- [[entities/token级精准控制生成长度3b模型击败gpt-54claude|token级，精准控制生成长度：3B模型击败GPT 5.4、Claude]] — 长度即值函数LenVM

## 工程实践
- [[entities/harness-engineering|'Harness Engineering：AI 从]] — 六层架构+七大反模式+分级决策树19712字rv9
- [[entities/yidian-tianxia-context-engineering-agentic-ai-qcon|一点天下：Context Engineering 与 Agentic AI (QCon)]] — 7114字最全六层上下文版
- [[entities/cursor-复盘-harness模型决定能力上限harness-决定生产下限|Cursor 复盘 Harness：模型决定能力上限，Harness 决定生产下限]] — Cursor复盘主版
- [[entities/three-tools-in-one-gstack-superpowers-openspec-engineering-ai-coding|三器合一：gstack + Superpowers + OpenSpec 工程化 AI 编程实战]] — gstack变体四串联点
- [[entities/claude-code-之父最新访谈编程已经结束harness-将消失claude-code-将只有-100-行代码loop-才是未来|Claude Code 之父最新访谈：编程已经结束、harness 将消失、Claude Code 将只有 100 行代码、loop 才是未来]] — Boris访谈全版
- [[entities/codeindex-让大模型更好地理解你的代码|Codeindex · 让大模型更好地理解你的代码]] — 语义索引+依赖图
- [[entities/在-rds-postgresql-中实现-rabitq-量化|在 RDS PostgreSQL 中实现 RaBitQ 量化]] — 32倍压缩理论误差界
- [[entities/glm5-scaling-pain-inference|GLM-5 Scaling 痛点与推理优化]] — KV Cache竞态排查复盘7544字全版
- [[entities/vivo-agent-brain-body-icu-harness-evolutionary-framework-2026|vivo Agent 系统分析：大模型是大脑不是马，Harness 是 ICU 不是马鞍]] — 大脑身体ICU隐喻框架
- [[entities/autoresearch-marketing-growth-amap-ai-native|高德 Marketing AutoResearch：AI Native 营销增长经营托管框架]] — 营销经营托管
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr|AgentOps: Operationalize agentic AI at scale with Amazon Bedrock AgentCore]] — 四支柱解析版
- [[entities/别再把上下文当聊天记录|别再把上下文当聊天记录]] — 上下文是工作空间四家趋同
- [[entities/告别氛围编程基于-harness-治理和-sdd-的团队级-ai-研发范式演进与实践|告别“氛围编程”：基于 Harness 治理和 SDD 的团队级 AI 研发范式演进与实践]] — 确定性承重层四步骤
- [[entities/我把-karpathy-的-autoresearch-搬到了软件开发领域效果炸了|我把 Karpathy 的 AutoResearch 搬到了软件开发领域，效果炸了]] — val loss换多维评分
- [[entities/精选-10-个开发者常用的-ai-智能体技能agent-skills|精选 10 个开发者常用的 AI 智能体技能（Agent Skills）]] — 四类技能质量筛选节点
- [[entities/你写的-skill及格了吗|你写的 Skill，及格了吗？]] — D1元数据定生死评估框架
