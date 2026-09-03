---
title: "RAG 检索增强生成"
created: 2026-06-30
updated: 2026-08-01
type: concept
tags: [concept, rag, retrieval, llm, grounding, hallucination-reduction]
sources: []
---

## 定义

RAG（Retrieval-Augmented Generation，检索增强生成）是一种将外部知识检索与 LLM 生成相结合的架构模式。系统先从知识库中检索相关文档片段，再将检索结果注入 LLM 的上下文窗口，使模型基于真实数据生成回答，而非仅依赖参数化记忆。RAG 是当前降低 LLM 幻觉、保持知识时效性的主流工程方案。

## 核心范式

- **检索-生成两阶段**：先用向量检索（dense）或关键词检索（sparse）召回相关片段，再拼接到 prompt 中让 LLM 生成
- **Chunk 策略**：文档切分为 256-1024 token 的语义块，过大丢失精度，过小丢失上下文
- **Embedding 模型**：text-embedding-3-large、BGE-M3、E5-Mistral 等将文本编码为高维向量，余弦相似度检索
- **Reranking**：粗检索（top-50）→ 精排（top-5）两阶段，提升最终注入上下文的质量
- **Hybrid Search**：BM25 稀疏检索 + 向量稠密检索混合，兼顾精确匹配和语义理解

## 背景与提出

RAG 概念最早由 Facebook AI Research（Lewis et al., 2020）在论文 "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" 中提出。核心动机是解决 LLM 的两个根本缺陷：参数化知识的时效性问题（训练数据有截止日期）和幻觉问题（模型会自信地编造事实）。通过将知识外挂到检索系统，RAG 让 LLM 能访问最新信息且有据可查。^[raw/articles/rag-retrieval-augmented-generation-2020]

2023-2025 年 RAG 迅速成为企业 AI 应用的标配架构。LangChain、LlamaIndex 等框架将其标准化；向量数据库（Pinecone、Weaviate、Milvus）成为新基础设施；GraphRAG（微软，2024）引入知识图谱增强检索；Agentic RAG 将检索作为 tool 暴露给 agent，实现多步推理和自适应检索策略。

## 局限与反对声音

- **上下文窗口瓶颈**：即使 128K 窗口，注入过多检索片段也会稀释关键信息（Lost in the Middle 效应）
- **检索质量上限**：生成质量的天花板是检索质量——检索不到正确文档，LLM 无法凭空推理
- **延迟与成本**：每次请求额外的 embedding + 检索 + rerank 步骤增加 200-500ms 延迟和 token 成本
- **维护复杂度**：知识库需要持续更新、chunk 策略需要调优、embedding 模型需要版本管理
- **Fine-tuning 替代论**：部分场景下，直接 fine-tune 模型内化知识比 RAG 更高效（无需检索基础设施）

## 实践启示

1. **从简单开始**：先用 BM25 + 固定 chunk 大小建立 baseline，再逐步引入向量检索、reranking、hybrid search
2. **评估先行**：用 faithfulness、relevance、context precision 等 RAG 评估指标驱动优化，不要盲目调参
3. **Chunk 不是越小越好**：过小的 chunk 丢失上下文语义，512 token 是常见起点
4. **多路召回**：单一检索策略有盲区，hybrid search（BM25 + dense）显著提升召回率
5. **Agentic RAG 是趋势**：让 agent 自主决定何时检索、检索什么、是否需要多轮检索，比固定 pipeline 灵活得多

## 相关实体

- [[concepts/lost-in-the-middle]] — RAG 检索片段过多时的注意力衰减问题
- [[concepts/speculative-decoding]] — 推测解码可加速 RAG 场景的生成阶段
- [[concepts/moe-mixture-of-experts-2025]] — MoE 架构可降低 RAG 推理成本
- [[entities/dream-dense-retrieval-autoregressive-modeling-challengehub-2026]] — Dream dense retrieval 研究
- [[entities/graphrag-needed-aws-9-rag-comparison-2026]] — GraphRAG 与 RAG 方案对比

## 所属 MOC

- [[moc/rag-knowledge-retrieval|Rag Knowledge Retrieval]]
