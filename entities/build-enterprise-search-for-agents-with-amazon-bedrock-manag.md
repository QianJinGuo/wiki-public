---

title: "Build Enterprise Search for Agents with Amazon Bedrock Managed Ingestion"
created: 2026-07-24
updated: 2026-10-24
type: entity
tags: [aws, bedrock, agent, search, rag]
sources: [raw/articles/build-enterprise-search-for-agents-with-amazon-bedrock-manag]
confidence: 0.65
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Build Enterprise Search for Agents with Amazon Bedrock Managed Ingestion

> Knowledge bases that ground agents and generative AI applications over your enterprise data are hard to build at scale. Teams typically stitch together connectors, parsers, vector stores, knowledge graphs, and retrieval logic, then operationalize all of it for production. Each piece brings its own c

## 摘要

Amazon Bedrock Managed Knowledge Base 正式 GA，把企业级检索从「自组管道」变成「配置即用」：内置 SharePoint、Confluence、Google Drive、OneDrive、S3 与 Web Crawler 六类连接器，按内容类型自动选择解析策略，托管存储层自动扩缩并常开关键词＋语义混合检索，默认配置下三次 API 调用即可跑通从入库到检索的端到端流程。检索侧提供两级 API——低延迟的 Retrieve 与由基础模型拆解子查询、多轮迭代的 Agentic Retrieval；生产侧内置文档级 ACL 与查询时实时权限回源校验、CloudWatch 指标与 trace 事件，并可通过 AgentCore Gateway 以 MCP 工具形式暴露给 Strands、LangChain、CrewAI 等框架。本文以官方「简化接入、更聪明的检索、生产就绪」三支柱为线索，另补对其抽象边界、图与向量取舍、以及自建/采购经济性的分析。

## Build enterprise search for agents with Amazon Bedrock Managed Knowledge Base

Knowledge bases that ground agents and generative AI applications over your enterprise data are hard to build at scale. Teams typically stitch together connectors, parsers, vector stores, knowledge graphs, and retrieval logic, then operationalize all of it for production. Each piece brings its own challenges. You must decide which data sources to connect and how to parse multimodal document types. You must choose between graph and vector databases, then provision and scale them. You must also handle complex queries that reason across diverse content, and layer on the document-level access control, observability, and security that production demands. ^[raw/articles/build-enterprise-search-for-agents-with-amazon-bedrock-manag.md]

[Amazon Bedrock](<https://aws.amazon.com/bedrock/>) now offers [Managed Knowledge Base](<https://aws.amazon.com/bedrock/knowledge-bases/>) in general availability, a fully managed agentic retrieval solution that handles scaling, high-accuracy retrieval, and document access control on your behalf. You can connect your enterprise data sources or crawl the web and start ingesting. Getting started through the [AWS Management Console](<https://console.aws.amazon.com/>) requires no model selection. Sensible defaults take you from zero to your first retrieval in minutes, compared to the days or weeks typically needed to assemble a comparable pipeline from scratch. When you’re ready to customize, you have control over embedding models, rerankers, chunking strategies, and more. ^[raw/articles/build-enterprise-search-for-agents-with-amazon-bedrock-manag.md]

In this post, we walk through the three pillars that make this possible: simplified setup, smarter retrieval, and production readiness. We also show you [code examples](<https://github.com/aws-samples/amazon-bedrock-samples/blob/main/rag/managed-knowledge-bases/03-use-case-example/01-end-to-end-example-with-ac-gateway/01-bmkb-with-agentcore-gateway.ipynb>) for setting up a knowledge base and retrieving from it. ^[raw/articles/build-enterprise-search-for-agents-with-amazon-bedrock-manag.md]

## Simplified setup

Developers today typically procure and bui

→ [[raw/articles/build-enterprise-search-for-agents-with-amazon-bedrock-manag|原文存档]] ^[raw/articles/build-enterprise-search-for-agents-with-amazon-bedrock-manag.md]

---

## 深度分析

官方的三支柱叙事（简化接入、更聪明的检索、生产就绪）顺滑得几乎掩盖了真正的难点。把它倒过来读——先问生产环境会以什么方式失败，再看托管的边界画在哪里——得到的判断会更接近工程现实。

### 成本中心在粘合层，不在模型选择

连接器、解析器、向量库、图存储、检索逻辑这五段，任何一段单独看都不难，难的是它们之间的契约：增量同步如何判定「已修改」、解析失败如何回退、格式 schema 漂移如何通知下游、多模态表格被切成碎片后如何重建语义。这类工作不产生任何 demo 效果，却决定了系统能不能连续运行六个月。

更关键的是，这套复杂度是乘法而非加法——数据源数 × 格式数 × 权限模型数 才是真实工作量，每加一个 SharePoint 站点或一种扫描件格式，组合数就翻一倍。托管服务的核心价值因此不是「检索更准」，而是把这层组合复杂度收进一个 SLA：你把一个会无限膨胀的工程债换成了若干配置项。 ^[raw/articles/build-enterprise-search-for-agents-with-amazon-bedrock-manag.md:20-26]

### 托管抽象掉了运维不确定性，没抽象掉语义不确定性

被买断的部分很清晰：存储容量与向量维度、相似度度量与索引配置、分片与备份、补丁与容量管理、解析策略的选择、chunking 的默认值、以及 ACL 校验的执行路径。这些恰好是「做对了没人夸、做错了全公司骂」的部分。

没被买断的部分同样清晰：文档本身是否自描述、同一制度是否存在多个互相冲突的权威版本、源系统的权限语义如何映射到文档级 ACL（共享链接、群组嵌套、继承关系通常是映射失真的源头）、以及「答对了没有」的判定标准。换句话说，托管把运维不确定性换成了配置不确定性，而语义不确定性原封不动留给你。

这条边界决定了一个常见误判：把接入完成当成检索可用。接入只是把语料搬进了索引，效果好不好取决于语料治理与评测集，这两件事任何托管服务都无法代劳。

### 图还是向量：用查询期迭代置换索引期建模

原文把「选图库还是向量库」列为难点之一，而 Managed Knowledge Base 的答案是绕开这个选择：统一存储层常开关键词＋语义混合检索，不要求你建图；跨文档推理交给 Agentic Retrieval 的多轮子查询去逼近。

这是一个相当聪明的成本置换——把多跳推理的开销从 ETL 阶段（预先抽实体、建关系、维护本体）挪到 query 阶段（拆子查询、并行检索、评估充分性、必要时再迭代若干轮）。冷启动几乎为零，不需要事先知道数据里有什么实体；代价是延迟与 token 成本随问句复杂度上升，而且「多跳」只是概率性逼近，不是可验证、可解释的路径。 ^[raw/articles/build-enterprise-search-for-agents-with-amazon-bedrock-manag.md:98-118]

真正需要显式图的场景并没有消失：当实体关系本身就是答案（供应链上下游、股权穿透、案件关联）、当必须给出可解释的推理路径、或需要对关系做聚合统计时，反复捞文本再让模型拼装会退化成昂贵的猜测。判断标准不是「哪个更先进」，而是「你要的答案是文本片段，还是结构化的关系」。相关讨论见 [[entities/graphrag-needed-aws-9-rag-comparison-2026|AWS 九组 RAG 方案对比]] 与 [[entities/three-rag-architectures-classic-graph-agentic|经典/图/Agentic 三种 RAG 架构]]。

### 生产就绪与 build-vs-buy：把哪部分不确定性买断

生产环境的三道坎在原方案里各有对应机制，但每道都留了残余风险。权限上，预检索过滤加上查询时实时回源校验解决了权限漂移（转岗、改共享设置后索引滞后），代价是每次查询多一跳外部依赖，源系统可用性直接成为检索 SLA 的一部分，因此必须有 fail-closed 的降级设计。可观测上，CloudWatch 指标与 agentic trace 覆盖了调用量、错误率以及每一轮规划／检索／评估，但它们衡量的是「系统有没有正常工作」，而不是「答案对不对」——质量指标只能自建。延伸阅读 [[concepts/llm-observability-4-layer-model|LLM 可观测性四层模型]] 与 [[concepts/agent-security-architecture|Agent 安全架构]]。

第三道坎最容易被忽略：可替换性。AgentCore Gateway 把知识库 ID 藏在 MCP 工具名之后，agent 只按名字发现和调用工具，这使「更换检索后端」从改代码降级为改网关配置。对一个生命周期很可能长于所选服务的系统而言，这条解耦比任何单点功能都值钱，也与 [[entities/amazon-bedrock-agentcore-gateway-mcp-extension|AgentCore Gateway 的 MCP 扩展]] 所主张的网关隔离思路一致。

经济性上，这套方案把「容量预留」换成了「用量线性」：多模态解析、托管 embedding 模型与托管 reranker 都含在价格内，只对原始数据存储、检索调用以及 agentic 多轮推理收费。对内部知识搜索这种低频高价值负载（人均日查询个位数）几乎必然划算；对每次请求都要触发检索的高频场景，多轮 agentic 的调用成本会很快超过自建一套固定索引的边际成本。锁定成本也要计入：数据落在托管存储里，退出的代价是重新解析与重建索引，缓解手段是在业务侧保留自己的检索抽象层，把知识库当作可替换实现而非系统地基。这套判断标准与 [[concepts/rag-retrieval-augmented-generation|RAG]] 的经典分层是一致的——分层的意义正是让每一层都能被换掉。

## 实践启示

1. **分两阶段推进**：先只用创建知识库、挂数据源、启动作业三次调用跑通端到端，确认链路与权限无误，再进入效果调优。管道没通就去调 embedding，纯属浪费。
2. **先建评测集再选型**：准备 50–100 条真实业务问句与期望命中文档，把它作为更换 embedding 模型、reranker、chunking 策略的唯一裁决标准；否则每次都只是换了种错觉。
3. **按查询形状分流**：直查与 FAQ 走 Retrieve 保低延迟，比较／多跳／研究型问句才走 Agentic Retrieval，并用最大迭代轮数与规划模型做延迟-准确率旋钮，不要全局开启多轮迭代。
4. **ACL 以源系统为唯一事实来源**：不在索引里复制权限副本，同时为实时校验那一跳准备降级策略——源不可用时宁可拒答（fail-closed），也不要放行。
5. **把知识库藏在工具名之后**：用 Gateway 或自建 API 层隔离，agent 代码里不出现知识库 ID。这一步几乎零成本，却是未来换后端的唯一保险。
6. **同时盯三个数字**：检索 P95 延迟、单问句的检索调用次数、答案正确率。前两个 CloudWatch 直接给，第三个必须自建评测回路——没有它，所有可观测性都只是在证明系统「在跑」，而不是「有用」。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

