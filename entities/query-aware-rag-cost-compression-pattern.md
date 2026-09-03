---
title: "Query-Aware Compression: RAG 成本优化的后检索过滤模式"
created: 2026-08-22
updated: 2026-08-29
type: entity
tags: [rag, llm, cost-optimization, retrieval, prompt-engineering, aws]
sources: [raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress]
confidence: 0.7
---

# Query-Aware Compression: RAG 成本优化的后检索过滤模式

## 核心洞察：小模型过滤检索块

RAG 检索通常调高召回率，返回大量潜在相关块（top-k 常见 5-20），导致每次调用输入 token 数高达数千，成为规模化 RAG 的主要成本。**Query-aware compression** 的核心模式：在检索之后、最终回答调用之前，插入一个**更小、更便宜的模型**，让它读取检索块 + 用户查询，输出仅与问题相关的**逐字 span**（verbatim spans）；主模型（更大的昂贵模型）随后只接收过滤后的上下文生成答案。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

## 经济性原理

收益取决于两个因素：小/主模型的**单价比**（price ratio）和压缩比 `c`。对单次 RAG 查询，输入 token `R`、压缩比 `c>1`、输出 `A` token，通过减小送入主模型的输入 token 数实现成本节省。**次要收益**：移除无关上下文缩小幻觉表面。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

## 架构与叠加

小模型（Claude Haiku）做压缩调用、主模型（Claude Sonnet）做回答调用，两者在同一个 AWS Lambda 中运行；检索器（Amazon Bedrock Knowledge Bases，OpenSearch Serverless 支撑）嵌入查询并返回 top-k 块。该模式可叠加在既有能力上实现复合节省：prompt caching、智能提示路由（Intelligent Prompt Routing）、Rerank API。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

## 与既有 RAG 优化体系的关系

这是 RAG 高级优化 家族中"后检索处理"（post-retrieval processing）的具体实例，与 [[entities/rag-chunking-vectorization-rerank-distillation|分块/向量化/召回/重排]] 互补：后者优化检索质量，本文优化检索后的 token 成本。它与 [[concepts/context-management-agent-systems|上下文管理]] 主题共享"在昂贵模型前精炼输入"的思想。

→ [[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress|原文存档]]
