---
title: "Agent 情节记忆 vs 语义记忆"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, memory, episodic, semantic, cognitive-architecture, agent]
sources: [entities/agent-memory-architecture, entities/存之有序治之有矩agent-记忆系统的工程实践与演进]
---

## 定义

借用认知科学 Tulving 1972 分类：episodic memory 是「时间地点情境绑定的事件」（用户上周说了什么），semantic memory 是「去情境的事实知识」（python 列表语法）。Agent 系统需要分开存储，因为读取模式完全不同。

## 核心范式

- **episodic 用时序索引**：按 session_id + timestamp 检索，回答「上次怎么说的」
- **semantic 用主题索引**：按概念/向量/知识图谱检索，回答「这是什么」
- **混淆代价高**：把对话记忆当事实回答 → 模型产生幻觉式自指
- **consolidation 通道**：episodic 中的稳定事实 promote 到 semantic

### 背景与提出

这套分类来自认知心理学家 Endel Tulving 1972 年的经典论文《Episodic and Semantic Memory》，他当时在研究人类长期记忆的组织方式，提出 episodic memory 存储「个人经历的时间地点情境」、semantic memory 存储「脱离情境的抽象知识」。这个二元划分在认知科学界沿用了五十年，直到 2020 年代大模型出现，AI 研究者开始发现 LLM 的内部表征也有类似区分——模型对「上下文中见过的对话」和「训练时学到的世界知识」调用方式截然不同，于是开始有意识地把 episodic 和 semantic 机制引入 agent memory 系统设计 ^[entities/agent-memory-architecture]。

这个区分最初困扰 agent 系统开发者的问题是：为什么同一个 LLM 在某些时候能准确回忆用户上周说过的话，有时候却完全失忆？答案在于两类记忆的检索机制根本不同——episodic 依赖时间索引，semantic 依赖概念索引，混用索引就会导致「用语义检索找对话经历」或「用时间索引查概念定义」的错位，结果就是幻觉或遗漏。

### 范式细节

**episodic 用时序索引**是最直接的实现——每次用户交互生成一个 session_id + timestamp，写入向量数据库时同时存储时间戳。检索时按「最近 N 次对话」或「某个时间窗口内的对话」过滤，命中率极高。典型案例：Claude Code 的 Conversation history 模块按时间倒序排列，每次新 turn 只检索最近 20 条，之外的自动进归档 ^[entities/claude-code-core-internals]。这套机制在单 agent 场景下工作良好，但当 subagent 产生内部对话时，时间戳的语义含义就变得模糊——主 session 的时间轴和 subagent 独立的时间轴需要分别维护。

**semantic 用主题索引**则完全不同——给每条记忆计算 embedding，用向量相似度检索。优点是可以跨 session 找回相关知识，缺点是「相关性」不等于「真实性」：用户说过「我想减肥」和「我讨厌运动」，向量上可能高度相似，但语义相反。解决方案是在向量检索后再加一层 symbolic 过滤，比如检查 source_type 是否为 user_utterance ^[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]。

**混淆代价高**这个问题在生产环境中会造成真实故障。2025 年中某团队上线了一套客服 agent，上下文窗口塞满了用户历史对话，模型在回答时把「用户三个月前说想要取消订阅」当成了「用户现在要求取消」，因为两段 episodic memory 都被 promote 到了 working set，模型以语义方式处理了它们。根因分析发现，consolidation 逻辑里有个阈值设置错误，把高 confidence 的 episodic 条目直接当成了 semantic facts 使用。

**consolidation 通道**需要同时维护两条写入路径：episodic 的访问频率超过阈值时触发 promote，写入 semantic layer 时保留原始 source reference。这条通道本身也是信息衰减的逆过程——consolidation 把噪音过滤掉，但也可能把有用的边缘信息过滤掉。实践中需要在「记忆丰富度」和「记忆精度」之间取平衡。

### 局限与反对声音

Tulving 的二元划分本身在认知科学界就有争议。神经科学发现海马体同时参与 episodic 和 semantic 记忆的编码，暗示两者不是完全独立的系统。引入 agent 设计时，强行二分可能导致过度工程——为两个本可以共享基础设施的层分别建索引、建召回、建衰减，复杂度翻倍但收益不一定线性。

另一个批评是：episodic/semantic 的边界在 LLM 里本来就模糊。GPT-4 的 context window 里既包含对话历史，也包含文档片段，还包含系统 prompt——这些信息在模型内部被统一处理，并不真的分开存储在「情景记忆区」和「语义记忆区」。所谓「分开存储」更多是工程假设，而非模型真实运作方式的反映。

最后，consolidation 通道的实现复杂度常被低估。一个生产可用的 consolidation 系统需要：访问频率追踪（每次检索都要写 access log）、阈值配置（动态调整而非静态常量）、promote 操作的幂等性（同一条目被多次 promote 不能重复写入）、以及 source 失效传播（当 episodic 条目被删除时，semantic 层对应的条目也要同步标记）。这套系统开发工作量不亚于一个小型数据库。

### 现实案例

[[concepts/agent-memory-substrate-three-layer|memory substrate 三层]] 模型的实现是最直接案例：最下层是 episodic storage（session 级别向量 DB），中间层是 semantic storage（知识图谱或结构化 DB），最上层是 working memory（LLM context）。三个层级之间的数据流动完全遵循 episodic→semantic 的 promote 逻辑。

在 [[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes Wiki]] 的 provenance 体系里，每条记忆的 source_type 字段明确区分了 user_utterance（episodic 来源）和 doc_chunk（semantic 来源），检索时按 source_type 路由到不同索引。consolidation cron 每天凌晨运行，把连续 7 天没有被访问的 episodic 条目标记为「冷数据」，30 天没访问的直接删除——这是 access count 衰减算法的工程实现。

## 两种记忆的检索策略对比

episodic 记忆检索适合「最近发生了什么」类问题（用户在 L1 里能直接 recall），semantic 记忆检索适合「我学到了什么」类问题（需要 L3 召回）。两种检索的工程实现差异：episodic 用 FIFO/LRU + 时间窗口过滤，semantic 用向量相似度 + 主题聚类。混合检索（同时跑两种然后融合）是当前研究热点。Hermes Agent 的策略是显式分离——episodic 在 cronjob tick 间传递，semantic 在 wiki 中按需检索，避免两种记忆互相干扰。

## 在 wiki 中的关联

- [[concepts/agent-memory-architecture|agent memory 整体]]
- 认知架构
- [[concepts/agent-memory-substrate-three-layer|memory substrate 三层]]
- [[concepts/memory-consolidation-decay|memory consolidation]]
- [[concepts/agent-memory-system-design|memory system design]]

## 进一步阅读

- [[entities/agent-memory-architecture]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]

## 所属 MOC

- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
