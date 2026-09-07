---
title: "Task-Aware Knowledge Compression (TAKC)"
created: 2026-07-28
updated: 2026-09-07
type: entity
tags: [rag, knowledge-compression, aws, bedrock, retrieval, enterprise-ai]
sources: [raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Task-Aware Knowledge Compression (TAKC)

> **Task-Aware Knowledge Compression (TAKC)** 是一种将知识库离线预压缩为任务特定表示的技术方案，由 AWS 在 2026 年 7 月提出并在 AWS 上实现开源参考架构。TAKC 解决了 RAG 在跨文档推理和大规模知识库全面覆盖上的固有局限。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 核心思路

TAKC 的核心洞察是：**同一文档在不同任务下需要保留不同信息**。一张年报的财务分析视图需要收入/利润率/现金流数据，而合规审查视图需要监管引文/违规历史。通用摘要试图覆盖所有内容反而稀释了信息密度。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

TAKC 通过任务感知提示（task-aware prompt）对文档进行离线压缩，每个任务类型对同一源文档产生不同的压缩输出。压缩后系统用 Redis 缓存存储多速率压缩结果，查询时直接检索压缩表示而非原始文档。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 多速率压缩（Multi-rate Compression）

TAKC 维护四个压缩层级 —— 不同查询需要不同 fidelity：^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

| 层級 | 压缩比 | 上下文保留 | 适用场景 |
|------|--------|-----------|---------|
| Light | 8× | ~12.5% | 多步推理、跨文档综合 |
| Medium | 16× | ~6.25% | 中等复杂度分析查询 |
| High | 32× | ~3.1% | 事实查找、定义明确的问题 |
| Ultra | 64× | ~1.6% | 分类任务、关键词查找 |

查询复杂度分析器根据查询长度、问题类型、分析性语言等信号自动路由到适-当层级。大多数企业查询可从高压缩层级低成本响应，只有少数复杂查询消耗更大上下文预算。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 与 RAG 的对比

| 维度 | TAKC 更适合 | RAG 更适合 |
|------|------------|-----------|
| 查询类型 | 跨文档推理、综合 | 窄域事实查找 |
| 知识库稳定性 | 每日变化或更少 | 每小时变化或更多 |
| 任务可预测性 | 任务类型明确 | 查询模式不可预测 |
| 覆盖需求 | 必须考虑全量语料 | 仅需少量相关文档 |
| 来源追溯 | 不需要 | 需要（用户需看原文） |
| Token 预算 | 紧张 | 灵活 |

实践中两者互补：RAG 做快速查找，TAKC 处理分析性查询。查询复杂度分析器可在两者之间路由。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## AWS 架构实现

TAKC 在 AWS 上部署为两个解耦的无服务器流程：^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

- **Ingestion Pipeline**：文档落地 S3 → S3 事件通知触发 Lambda 分块（256-token 块，50-token 重叠）→ 异步调用压缩 Lambda → Amazon Bedrock（Claude 3 Haiku / Sonnet）按四层压缩 → 存入 ElastiCache Serverless + S3 备份
- **Query Pipeline**：Cognito JWT 认证 → API Gateway → WAF 防护 → Lambda 查询复杂度分析 → ElastiCache 获取压缩缓存 → Bedrock 推理响应

参考实现开源在 [aws-samples/sample-bedrock-takc-compression](https://github.com/aws-samples/sample-bedrock-takc-compression)，单命令 CDK 部署。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 适用条件

TAKC 适用于知识库变化不频繁、查询模式可预测、需要跨文档推理的场景（如金融尽调、合规审查）。对于每小时变化的知识库，RAG 的按查询检索模型更实用。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

→ [[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a|原文存档]]
