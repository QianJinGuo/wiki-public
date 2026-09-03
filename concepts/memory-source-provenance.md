---
title: "Agent 记忆 provenance 与可追溯"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, memory, provenance, trust, trace, agent]
sources: [entities/存之有序治之有矩agent-记忆系统的工程实践与演进, entities/hermes-wiki-9-step-auto-growing-knowledge-network]
---

## 定义

memory provenance 要求 agent 长期记忆中每条事实都绑定来源（用户原话 / 工具调用结果 / 外部文档片段）。没有 provenance 的记忆 = 不可信幻觉源——三个月后回看一条「事实」，分不清是用户告诉的还是模型自己编的。

## 核心范式

- **写入时强制标 source**：source_type + source_id + timestamp 三元组
- **读取时返回 source**：让 LLM 自己决定要不要相信，而不是无条件采信
- **冲突检测**：同事实多 source 不一致时触发「待验证」标记
- **source 失效传播**：source 被删除时自动标记下游 memory 为「unverified」

### 背景与提出

memory provenance 的概念来自数据库领域的 lineage/tracing 研究——1990 年代数据仓库研究中就提出「每条派生数据必须能追溯到原始 source」。在 AI 领域，这个需求变得尤为迫切，因为 LLM 有一个根本性的特性：它会「忘记自己不知道什么」，然后用听起来自信的幻觉填充空白。2023 年的一项用户调研显示，在有记忆功能的 AI 助手中，超过 40% 的用户曾在后续对话中发现 AI「记错了」自己之前说过的话，但无法定位错误源头。

这个问题的根源是：LLM 的 context window 是一个高度同质的信息池——用户原话、系统指示、工具输出、模型生成的内容混在一起，模型对它们一视同仁地用作推理依据。provenance 的核心思想是打破这种平等：给每条记忆打上来源标签，让模型（和开发者）能够区分「用户明确说过的事实」「工具计算出的数据」「模型自己推断的结论」——对这三者的信任级别应该完全不同 ^[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]。

[[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes-Wiki]] 是 provenance 范式最完整的工程实践。它的设计动机来自于知识库长期运行后的一个典型症状：三个月前写入的一条「事实」，现在模型用它作为推理依据，但没人记得这条事实是从哪里来的——是用户说的？是爬虫抓的？是模型自己推的？不同来源的可信度差异巨大，但都被平等对待了。Hermes-Wiki 通过强制 source 标注解决了这个问题。

### 范式细节

**写入时强制标 source**是系统设计中最难落实的一条，因为需要在所有写入路径上统一执行。如果有十条写入路径但只有九条执行了 source 标注，provenance 体系就出现漏洞。工程实现上，通常在 memory 写入 API 层统一拦截，强制要求调用方提供 source三元组（source_type, source_id, timestamp），不提供的写入请求直接拒绝。这条约束在 [[concepts/agent-memory-substrate-three-layer|memory substrate 三层]] 的最底层（episodic layer）实施，因为那是所有记忆的最初入口 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]。

**读取时返回 source**意味着每次模型获取记忆时，不仅返回记忆内容，还同时返回这条记忆的来源描述。实现上有两种路径：一是把 source 信息直接塞进 prompt，让模型自己决定是否采信；二是用独立的 verification layer 在模型推理之前过滤掉低可信度记忆。两种路径各有权衡——前者实现简单但把判断责任推给模型（模型可能判断错误），后者更安全但引入额外的过滤延迟。

**冲突检测**是 provenance 体系里最实用的副产品：当同一条 facts（比如「用户偏好 dark mode」）出现多个 source 且内容不一致时（比如一个 source 说 dark mode，一个说 light mode），系统自动创建「冲突票」交给用户仲裁。这个机制把 AI 记忆错误从「静默扩散」变成「显式报警」——错误不会在后续对话中持续存在并放大，而是被立即捕获。实际运营数据显示，Hermes Wiki 上线冲突检测后，用户反馈「AI 记错了」的数量下降了 70%，因为很多错误在产生时就被检测到了。

**source 失效传播**是整个体系里最复杂的一条，但也最关键。假设用户上传了一份 PDF 报告，系统从中提取了 20 条 facts 写入 memory；后来用户删除了这份 PDF——这 20 条 facts 变成了「来源已删除」的状态，不能继续作为可靠知识使用。实现方式是在 source 表里维护「source → 派生 memory 条目」的反向索引，当 source 被删除时，批量更新所有派生条目的状态为 unverified ^[entities/agent-self-improvement-six-mechanisms]。

### 局限与反对声音

provenance 最大的实施障碍是「历史债务」——为一个已经运行了一段时间、没有 source 标注的记忆系统添加 provenance，需要对现有记忆批量补标。批量补标有两个选择：让 AI 自动猜测 source（用 AI 为每条记忆推断来源，可信度有限）或者要求用户手动确认（成本高，可能用户已经忘了）。无论哪种，都有严重的可用性问题。

另一个批评是：provenance 会显著增加 memory 系统的存储开销和管理复杂度。每条记忆多存储三个字段（source_type, source_id, timestamp），索引体积增加约 20-30%；source 表本身需要独立维护，支持 source→memory 的反向查询；冲突检测需要额外的比较运算。这些开销在中小规模系统上可以接受，但当记忆量达到百万级别时，存储和计算成本变得相当可观。

更重要的是，provenance 无法解决「source 本身不可靠」的问题。如果用户上传了一份错误的 PDF，provenance 会忠实地记录「这条知识来自那份 PDF」，但不会告诉你「那份 PDF 的数据是错的」。provenance 是「可追溯性」保证，不是「正确性」保证。一个完整的 trust 体系还需要 fact verification 机制（如外部数据交叉验证），这比 provenance 本身要复杂得多。

### 现实案例

[[concepts/source-first-knowledge-compilation|source-first 编译]] 采纳了 Hermes Wiki 的 provenance 设计：知识编译的输入是带明确出处的文档片段，编译过程中每个 derived knowledge 都保留原始出处的 wikilink。当知识被其他 entity 引用时，引用链被同时记录——这意味着任意一条知识，可以一直回溯到最原始的 source document。这个引用链在 [[concepts/agent-memory-substrate-three-layer|memory substrate 三层]] 的 semantic layer 实现，使得 data quality pipeline 能够对「高引用数且高 source 可信度」的记忆给予更高的 retrieval ranking 加权。

[[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes-Wiki]] 的 provenance 还有一个实用的副产品：它让 AI 的「胡说八道」变得可以审计。当用户质疑模型的某个回答时，开发者可以通过 memory trace 找到模型用了哪条记忆、那条记忆的 source 是什么——如果 source 是模型自己生成的而非用户提供的，就说明是 hallucination。这个审计能力在医疗、法律等高风险应用场景中尤为重要。

## 在 wiki 中的关联

- [[concepts/source-first-knowledge-compilation|source-first 编译]]
- [[concepts/agent-memory-substrate-three-layer|memory substrate 三层]]
- agent observability
- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes-Wiki provenance 范式]]
- data quality pipeline

## 进一步阅读

- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
