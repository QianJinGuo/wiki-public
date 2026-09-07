---
title: "Zero-Mem — LLM Agent 的零 Token 记忆操作"
created: 2026-08-06
updated: 2026-09-07
type: entity
tags: [agent-memory, llm-agent, context-engineering, token-efficiency, memory-architecture, arxiv]
sources: [raw/articles/zero-mem-zero-token-memory-operations]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Zero-Mem — LLM Agent 的零 Token 记忆操作

## 核心问题

LLM agent 需要记忆才能在长交互中保持一致行为，但多数系统用额外的 LLM 调用去操作记忆：生成中间记录、调解检索都会产生持续的 token 与时间成本，且省略/合并的细节会遮蔽原始证据。Zero-Mem（arXiv 2607.29377）提出的问题是：**结构化记忆访问是否必须依赖生成**？^[raw/articles/zero-mem-zero-token-memory-operations.md]

## 方案：零 Token 记忆操作

Zero-Mem 引入 **zero-token memory operations**：除最终问答外，没有任何一步调用 LLM 或消耗 LLM 输入/输出 token（encoder 计算单独核算）。其设计要点：^[raw/articles/zero-mem-zero-token-memory-operations.md]

- **原始轨迹即唯一记录源**：不生成中间表示，直接保留 interaction traces 作为 source of record。^[raw/articles/zero-mem-zero-token-memory-operations.md]
- **双重视图组织**：entity–context graph 暴露跨交互的连接，temporal hierarchy 保留会话局部性与 session 状态。^[raw/articles/zero-mem-zero-token-memory-operations.md]
- **查询时双视图加权**：对每个 query 加权两个视图、从两者检索，并沿其结构恢复支持性关系或周边上下文。^[raw/articles/zero-mem-zero-token-memory-operations.md]
- **确定性校准**：先丢弃冲突证据，再让 reader 的答案 grounded 在检索到的 traces 上；只有最终 QA reader 调用 LLM。^[raw/articles/zero-mem-zero-token-memory-operations.md]

## 结果

在长记忆与长上下文 QA benchmark 上，Zero-Mem 达到竞争性性能，同时把记忆操作中的 LLM 调用与 token 消耗清零；在相同 final-QA reader 与 context 预算下，相比最快的 baseline，记忆操作时间成本降低 **57.6%**。Ablations 支持两个视图及其查询相关协调各自的贡献。结论：结构化 agent 记忆未必需要生成过去的中间表示。^[raw/articles/zero-mem-zero-token-memory-operations.md]

## 意义与定位

Zero-Mem 与「记忆即生成」的主流路线（如用 LLM 摘要/压缩记忆）正面竞争，属于 [[concepts/agent-memory-architecture|Agent Memory Architecture]] 中「存储原样数据 vs 编译后数据」争论的激进一方（对比 [[entities/agent-memory-storage-six-schools-wiki-compile-vs-raw-data-debate|compile vs raw-data 之争]]）：以检索侧图结构 + 确定性校准换取零生成成本。它在 token 经济学上直接相关 [[concepts/context-window-economics|Context Window Economics]]，并与同期的检索侧记忆工作（[[entities/memreranker-reasoning-aware-agent-memory-reranking-2026|MemReranker]]、[[entities/mragent-memory-reconstructed-not-retrieved-nus-icml2026|MrAgent]]）形成对照。^[raw/articles/zero-mem-zero-token-memory-operations.md]

## 相关

- [[concepts/agent-memory-systematic-framework|Agent 记忆系统框架]]
- [[concepts/agent-memory-system-design|Agent 记忆系统设计]]
- [[entities/agent-memory-architecture|Agent 记忆架构]]
- [[entities/agent-memory-four-schools-comparison-2026-07-22|Agent 记忆四学派对比]]
- [[entities/state-of-memory-in-agent-harness-mem0-2026|Harness 中的记忆现状 (mem0)]]
- [[entities/memory-vs-rag-agent-memory-systematic-framework|记忆 vs RAG 框架]]
- [[entities/agent-memory-injection-5-dimensions-4-papers-agent-shouji-2026|记忆注入五维]]
- [[entities/context-engineering-three-memory-paradigms|Context Engineering 三范式]]

→ [[raw/articles/zero-mem-zero-token-memory-operations|原文存档]]
