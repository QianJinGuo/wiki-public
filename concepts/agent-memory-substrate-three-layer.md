---
title: "Agent 记忆 substrate 三层架构"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, memory, substrate, architecture, agent, enterprise]
sources: [entities/enterprise-ai-memory-substrate-three-layer-architecture, entities/存之有序治之有矩agent-记忆系统的工程实践与演进, entities/agent-memory-architecture]
---

## 定义

Agent 记忆 substrate 三层架构：working memory（当前 turn 上下文窗口）/ episodic memory（最近 N 个会话片段）/ semantic memory（沉淀的事实知识库）。三层有不同的写入/读取/淘汰策略，对应 agent 的「短期 / 中期 / 长期」记忆能力。

## 核心范式

- **working layer**：完全在 LLM 上下文窗口内，每 turn 刷新
- **episodic layer**：session-scoped，FIFO / TTL 淘汰，召回近期对话
- **semantic layer**：知识库 / 向量库 / 知识图谱，长期沉淀，按主题检索
- **层间流动**：working → episodic（session 结束）→ semantic（人工/自动 promote）

## 背景与提出

Agent 记忆 substrate 三层架构是把 AI agent 的记忆系统类比为操作系统内存层次：L1 寄存器（当前 context window）、L2 缓存（压缩后的近期对话摘要）、L3 磁盘（长期外部存储）。^[entities/enterprise-ai-memory-substrate-three-layer-architecture] 这个类比来自系统工程的直觉：不同的存储介质有不同的延迟/容量/成本权衡，分层是自然解法。但和 OS 内存管理的关键区别是——agent 的 L1 是概率性的（LLM attention 不保证看到所有 token），这让「缓存一致性」问题更加棘手。

## 范式细节

三层各有不同的工程关注点。L1（context window）：最昂贵、最快、容量最小。优化目标是最小化 token 浪费——每个 token 都要贡献于当前任务。管理工具是 working set 裁剪、工具输出预算、prompt 精简。L2（压缩摘要）：中等延迟、中等容量。优化目标是保留「任务关键信息」同时丢弃噪声。管理工具是 autoCompact、snipCompact、microCompact。L3（外部存储）：最慢、最大、最便宜。优化目标是索引效率——需要时能快速找到。管理工具是向量搜索、wikilink 图谱、SQL 查询。三层之间的 promote/decay 规则决定了记忆的生命周期：高频访问的信息从 L3→L2→L1 提升，长期不访问的信息反向衰减。^[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]

## 局限与反对声音

三层模型的局限在于「层间一致性」。L2 摘要可能丢失 L1 中存在的关键信息，L3 检索可能返回和 L1 当前任务无关的结果。当 agent 从 L2/L3 恢复上下文时，它不知道自己不知道什么——这是一个不可检测的信息缺失。另一个问题是「衰减策略的参数敏感性」：access-count decay 的阈值设高了会遗忘重要信息，设低了会让 L1 膨胀——没有通用最优解，必须按任务类型调参。

## 现实案例

Hermes Agent 的 Memory + Skills + Wiki 三层系统就是 substrate 三层架构的具体实现：Memory = L2（跨 session 的压缩摘要）、Skills = L3（长期外部存储、按需加载）、Wiki = L3（结构化知识库、wikilink 索引）。注意 Hermes 没有显式的 L1 管理——它依赖底层 LLM provider 的 context window 管理，自己不做 working set 裁剪。这是和 Claude Code 的关键区别：Claude Code 在 harness 层面管理 L1（五层压缩策略），Hermes 把 L1 管理交给模型 provider。^[entities/hermes-agent]

## 现实案例

Agent 记忆 substrate 三层架构的三个落地案例。Hermes Agent：L1 = system prompt + Memory + Skills，每 turn 全量加载；L2 = autoCompact 生成的会话摘要，session-scoped；L3 = Obsidian wiki 作为长期知识库，按需通过 wikilink 检索。三层之间的 promote 是显式的：用户在 wiki-life 上手动 promote「重要洞察」到 entity/，AI 在 wiki 中自动 promote「高频访问的 entity」到更高 tag 权重。^[entities/hermes-agent] Claude Code：L1 = 当前 conversation + CLAUDE.md 工作记忆；L2 = autoCompact 摘要；L3 = 文件系统（用户的所有 .md / .py / .json 文件）。Claude Code 没有自动 promote 机制，所有 L3 写入都需要用户显式触发（写文件）。^[entities/claude-code-core-internals] MemGPT（Letta）的设计更激进：把 L1/L2 显式化为「core memory block + archival memory + recall memory」，提供 paginated read/write API，让 agent 自己决定哪些信息进哪层。这是 substrate 模型的工程化最强形态。

## 实践启示

三层架构落地时的核心决策是「promote 规则」——什么信息从 L2 升到 L3、什么从 L3 召回进 L1。Hermes Agent 的实践是显式 promote：用户手动选择「这条洞察值得长期记住」时触发 promote，AI 不自动 promote 避免污染知识库。这个策略适合 wiki 类长尾知识场景——信噪比高但召回成本高。Claude Code 的实践是会话级 L1/L2 + 文件系统 L3，没有自动 promote——用户写文件时 L3 才更新，简单但 L3 可能过期。MemGPT 的实践是 agent 自己管理三层——core memory / archival / recall 三块，agent 自主读写。这个策略适合 agent-as-product 场景（用户不参与记忆管理），但风险是 agent 可能丢失关键信息。Promote 规则的另一个关键是「decay 策略」——长期不用的信息应该自动降级或删除，否则 L1/L2 会膨胀。常见做法是 access-count-based decay（30 天没访问降一级）或 time-based decay（固定 TTL）。选择策略时考虑「忘记的成本」——如果忘记关键决策的成本极高（如医疗），选保守的 decay；如果频繁更新的信息（如新闻），选激进的 decay。

## 与相邻概念的区分

和「RAG（Retrieval-Augmented Generation）」的关系：RAG 是 L3（semantic memory）的实现方式之一——通过向量检索把外部知识召回进 L1。但 RAG 不包含 L2（episodic memory）的概念，也不管理 L1 内部的工作记忆。三层架构比 RAG 更完整。和「向量数据库」的关系：向量数据库是 L3 的一种存储实现（语义检索），但 L3 也可以用 wikilink 图谱（Hermes Agent 风格）、SQL 查询（事实型记忆）、KV 存储（简单事实）。三层架构是设计模式，向量数据库是工具。和「LLM context window 扩展」的关系：扩展 context window（如 Claude 200K）是放大 L1 的容量，不解决 L2/L3 的设计问题。即使 L1 容量扩到 1M token，仍然需要 L2 压缩和 L3 长期存储——三层架构是「在容量限制下优化使用」的方案，不是「消除容量限制」的方案。和「Conversation History」的关系：conversation history 是 L2 的简单实现——保存原始对话而非压缩摘要。但 raw history 比 summary 更占 token，且随时间指数膨胀——L2 应该用摘要 + 关键信息抽取，而非 raw history。

## 三层架构的容量规划

三层架构在容量规划上的目标是「L1 高效 / L2 适中 / L3 近乎无限」。L1 容量 = model context window - 系统开销 - 工具结果 buffer，典型值是 50K-150K token（Claude 200K / GPT-128K），有效利用率 70%。L2 容量 = 压缩后的历史对话，典型保留 5-20 个会话摘要，每个 1K-3K token，总量 ~30K。L3 容量 = 外部存储，原则上无限——但检索质量随库容量下降，需要定期清理 / 索引优化。容量规划的关键是「L1 装得下当前任务 + L2 装得下 L3 召回前的过渡状态 + L3 装得下所有长期知识」。L1 装不下时优先 L2 压缩（autoCompact），再考虑把任务拆小。L3 装不下时优先 L2 摘要 + 定期清理过期数据。三层之间的数据传输带宽是隐性瓶颈——L3 → L1 的向量检索每次 100-500ms，L1 → L2 的 compaction 每次 5-30s。带宽规划影响 agent 的整体延迟。

## 在 wiki 中的关联

- [[entities/enterprise-ai-memory-substrate-three-layer-architecture|企业 AI memory substrate 三层]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进|Agent 记忆工程实践]]
- [[concepts/agent-memory-architecture|agent memory architecture]]
- [[concepts/episodic-vs-semantic-memory-agent|episodic vs semantic]]
- [[concepts/working-set-vs-long-term-memory|working set vs long-term]]

## 进一步阅读

- [[entities/enterprise-ai-memory-substrate-three-layer-architecture]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/agent-memory-architecture]]

## 所属 MOC

- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
