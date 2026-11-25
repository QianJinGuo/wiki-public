---
title: "Task-Aware Knowledge Compression (TAKC)"
created: 2026-07-28
updated: 2026-11-26
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

RAG 每次查询都重新付费，且只看到命中的少量片段；TAKC 把最贵的压缩调用前移到摄入期一次完成，查询路径只剩缓存查找加压缩上下文推理。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:68] 更关键的是质量：跨文档推理的连接（供应商付款条款 ↔ 现金流 ↔ 未决诉讼）没有词面相似性，相似度检索捞不到，而 TAKC 压缩时同时看到整库，连接被显式保留——它提供的是全库的压缩形态，而非 top-k 片段。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:16-24]

经济账写在成本表里：10 万 token 库每日千次查询，全上下文相对输入成本 100%，RAG top-10 约 10%，TAKC 四档 12.5%/6.25%/3.1%/1.6%——只有「库变动不频繁且被反复查询」时压缩成本才摊得回来，每小时变化的库退回 RAG 的按查询检索更划算。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:76-89] 路由经济学补上另一面：多数企业查询本就是简单查找，可由高压缩档以极低成本服务，只有少数复杂查询才消耗更大的上下文预算。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:32-34]

任务感知压缩的本质是信息密度问题——通用摘要试图覆盖一切，反而稀释了任何具体用例的密度；同一份 10-K 按财务视角压出来保留收入/利润率/现金流，按法律风险视角压出来「长得完全不同」。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:20-24] 所以压缩质量的上限由任务清单的粒度决定，而不由压缩模型决定。

### 压缩的真实机制：任务提示是算法，压缩比是唯一旋钮

四档产物来自同一任务提示的四次调用：提示显式声明保留什么（数值与指标、实体及属性、因果关系与依赖），唯一改动的变量是 COMPRESSION TARGET（1/8 到 1/64）。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:50-62] 「任务感知」的全部语义都落在提示里，换掉提示等于换掉压缩算法——所以提示必须版本化存放（Parameter Store 或专门 S3 前缀），保证可审计并在更新时触发再压缩。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:20]

压缩质量没有现成指标，唯一可操作的门禁是回归对比：把每档压缩上下文生成的响应与全量未压缩文档生成的响应逐一对比，参考实现自带现成测试脚本。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:34]

### 时效性与驱逐的权衡

离线预压缩的代价是**陈旧**：产物冻结了生成那刻的任务视图，TAKC 用 24 小时 TTL 界定陈旧上限；缓存过期或被驱逐时回落 S3 备份并在读取时回填，说明这是性能事件而非正确性事件。但 S3 备份存的仍是压缩后产物，源文档更新必须重新走摄入流程，压缩视图才追得上源。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:62] 沿 [[concepts/claim-half-life|论断半衰期]] 的框架看，产物半衰期由两层决定：源变更频率决定内容层时效，提示版本决定语义层时效——提示一改旧产物即刻失效，这是版本化再压缩必须成为硬流程的原因。

### TAKC 在哪里会输

四类场景会输给 RAG：**需要来源追溯**——压缩产物无法回指原段落；**知识库每小时变化**——再压缩跟不上变更速率；**查询模式不可预测**——任务类型不可枚举，清单外查询白付成本；**窄域事实查找**——少量文档时 top-10 已足够便宜，压缩是多余动作。绝对最新的数值查询同样不适用。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:93-104]

路由层自带兜底：复杂度分析器用关键词信号与查询长度做启发式判断，置信度低时默认落到 Medium 档作为安全回退——宁可多花上下文，也不让复杂查询饿死在 Ultra 缓存里。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:66]

### 组合形态：分析走 TAKC，追溯走 RAG

受监管负载的标准形态是组合：TAKC 生成跨文档分析结论，RAG 取回支撑原文供审计，分析器在两者之间路由。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:104] TAKC 的档位路由与既有 RAG 优化（元数据过滤、查询重写）互补——先用后者缩小文档集，再施加压缩。^[raw/articles/beyond-rag-task-aware-knowledge-compression-for-enterprise-a.md:34] 这把 TAKC 从「RAG 替代品」重新定位为分析型查询的预处理层。

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
