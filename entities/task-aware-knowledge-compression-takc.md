---
title: "Task-Aware Knowledge Compression (TAKC)"
created: 2026-07-28
updated: 2026-09-10
type: entity
tags: [rag, knowledge-compression, aws, bedrock, retrieval, enterprise-ai]
sources: [raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Task-Aware Knowledge Compression (TAKC)

> **Task-Aware Knowledge Compression (TAKC)** 将知识库离线预压缩为任务特定表示，由 AWS 于 2026 年 7 月提出并开源参考架构，用以解决 RAG 在跨文档推理与全量覆盖上的固有局限。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 摘要

RAG 相似度检索在跨文档分析任务上会撞到天花板：能捞出片段，却漏掉片段间的连接。TAKC 用 LLM 按任务类型把整个知识库离线预压缩，查询时检索压缩表示而非原文，token 降低 8×–64×。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 核心要点

- **离线预压缩**：每份文档每任务只压一次，成本随查询摊薄。
- **任务感知提示**：显式声明保留什么（数值、实体、因果），区别于通用摘要。
- **四层多速率**：8×/16×/32×/64×，对应约 12.5%/6.25%/3.1%/1.6% 上下文保留。
- **全量覆盖 + 复杂度路由**：压缩整个知识库而非 top-k 片段；按查询长度与类型选档。
- **缓存与成本**：`takc:{task}:{rate}` 写入 ElastiCache，24h TTL、S3 备份回填；Ultra 档输入成本约全上下文的 1.6%。
- **与 RAG 互补**：需审计时用 TAKC 出结论、RAG 取原文。

## 核心思路

TAKC 的核心洞察是：**同一文档在不同任务下需要保留不同信息**——年报的财务分析视图要收入/利润率/现金流，合规审查视图要监管引文/违规历史，通用摘要覆盖一切反而稀释了信息密度。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

它用任务感知提示（task-aware prompt）离线压缩文档，每个任务类型对同一源文档产生不同输出；压缩后用 Redis 缓存多速率结果，细节不足时降到更低档。生产应把提示存入版本化配置（Parameter Store 或专门 S3 前缀）以支持审计与再压缩。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 多速率压缩（Multi-rate Compression）

TAKC 维护四个压缩层级 —— 不同查询需要不同 fidelity：^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

| 层級 | 压缩比 | 上下文保留 | 适用场景 |
|------|--------|-----------|---------|
| Light | 8× | ~12.5% | 多步推理、跨文档综合 |
| Medium | 16× | ~6.25% | 中等复杂度分析查询 |
| High | 32× | ~3.1% | 事实查找、定义明确的问题 |
| Ultra | 64× | ~1.6% | 分类任务、关键词查找 |

查询复杂度分析器按查询长度、问题类型、分析性语言等信号路由到适-当层级：多数企业查询由高压缩层级低成本响应，只有少数复杂查询消耗更大上下文预算。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

**档位如何产生与存放**：每个分块调四次 Bedrock 压缩，四次同提示、只改 `COMPRESSION TARGET`（1/8~1/64）；产物以 `takc:financial:medium` 复合键写入 ElastiCache Serverless 并备份 S3，24 小时 TTL，过期或被驱逐时回落 S3 回填。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 与 RAG 的对比

| 维度 | TAKC 更适合 | RAG 更适合 |
|------|------------|-----------|
| 查询类型 | 跨文档推理、综合 | 窄域事实查找 |
| 知识库稳定性 | 每日变化或更少 | 每小时变化或更多 |
| 任务可预测性 | 任务类型明确 | 查询模式不可预测 |
| 覆盖需求 | 必须考虑全量语料 | 仅需少量相关文档 |
| 来源追溯 | 不需要 | 需要（用户需看原文） |
| Token 预算 | 紧张 | 灵活 |

实践中两者互补：RAG 做快速查找，TAKC 处理分析性查询，分析器可在两者之间路由。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

三条主轴：**成本**上，10 万 token 库每日查 1000 次，全上下文 100,000 token / 100%；RAG（top-10）约 10,000 / 10%；TAKC 四级 12,500 / 12.5%、6,250 / 6.25%、3,125 / 3.1%、1,563 / 1.6%。**延迟**上，压缩只在摄入期发生，查询只剩缓存查找加压缩上下文推理。**精度**上，TAKC 以全量覆盖换跨文档综合质量、放弃来源追溯，受监管负载应组合两者。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## AWS 架构实现

TAKC 在 AWS 上部署为两个解耦的无服务器流程：^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

- **Ingestion**：S3 落地 → S3 事件触发 Lambda 分块（256-token 块、50-token 重叠）→ 异步压缩 Lambda → Bedrock（Claude 3 Haiku / Sonnet）四层压缩 → ElastiCache Serverless + S3 备份
- **Query**：Cognito JWT → API Gateway → WAF → Lambda 复杂度分析 → ElastiCache 取缓存 → Bedrock 推理

**Lambda**（Python 3.12+）承担计算，**ElastiCache Serverless** 按复合键读缓存，**Cognito** 管 JWT，**WAF** 前置限流，**API Gateway** 暴露 REST，**S3** 存数据与备份（KMS 加密），**CloudWatch** 监控；Bedrock 用 Claude 3 Haiku、Claude 3 Sonnet 与 Amazon Titan Text，经 CDK context 配置。参考实现 [aws-samples/sample-bedrock-takc-compression](https://github.com/aws-samples/sample-bedrock-takc-compression) 以单 **CDK** 栈单命令部署，上传文档后约 2–3 分钟出四档，清理时先清空 S3 再 `cdk destroy`。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 适用条件

TAKC 适用于知识库变化不频繁、查询模式可预测、需要跨文档推理的场景（如金融尽调、合规审查）；每小时变化的库则 RAG 更实用。五条同时成立时收益最大：语料每日或更少变动、任务类型可枚举、需考虑全量语料、不需来源追溯、token 预算紧张——压缩成本只有对「变动不频繁且被反复查询」的库才摊得回来。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 深度分析

### 为什么预压缩在语料稳定时胜过查询时检索

RAG 每次查询都重新付费，且只看到命中的少量片段；TAKC 把最贵的压缩前移，稳定语料下这份成本被反复复用。更关键是质量：跨文档推理的连接（供应商条款 ↔ 现金流 ↔ 未决诉讼）没有词面相似性，RAG 捞不到，而 TAKC 压缩时同时看到整库，连接被显式保留。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

### 时效性与驱逐的权衡

离线预压缩的代价是**陈旧**：产物冻结了生成那刻的任务视图，TAKC 用 24 小时 TTL 界定陈旧上限。过期或被驱逐时回落 S3 备份并回填，说明这是性能而非正确性事件；但备份仍是压缩后产物，源文档更新必须重新触发摄入，任务提示一改旧产物即语义失效，故提示必须版本化并触发再压缩。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

### TAKC 在哪里会输

四类场景会输给 RAG：**需要来源追溯**（产物无法回指原段落）；**知识库每小时变化**（再压缩跟不上）；**查询模式不可预测**（任务类型不可枚举，清单外查询白付成本）；**窄域事实查找**（少量文档时 RAG 的 top-10 已足够便宜）。绝对最新数值类查询同样不适用。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md]

## 实践启示

1. **先判语料稳定性**：语料「每日变化或更少」是入场券；每小时变动直接选 RAG。
2. **把任务类型收敛成清单**：任务提示的粒度就是压缩视图的粒度。
3. **用回归对比做质量门禁**：逐档确认最激进的 Ultra 档没伤到关键任务再放行路由。
4. **版本化任务提示并触发再压缩**，并为「未命中 → 回源 S3 → 回填」准备延迟预算。

## 相关实体

- [[concepts/rag-retrieval-augmented-generation|RAG（检索增强生成）]]
- [[entities/amazon-bedrock|Amazon Bedrock]]
- [[concepts/context-engineering|Context Engineering]]

→ [[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a|原文存档]]
