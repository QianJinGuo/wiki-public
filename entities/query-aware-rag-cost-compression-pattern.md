---
title: "Query-Aware Compression: RAG 成本优化的后检索过滤模式"
created: 2026-08-22
updated: 2026-09-11
type: entity
tags: [rag, llm, cost-optimization, retrieval, prompt-engineering, aws]
sources: [raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Query-Aware Compression: RAG 成本优化的后检索过滤模式

## 摘要

RAG 检索为保证召回率会把 top-k 拉高，代价是每次请求都往昂贵的主模型塞数千 token 上下文，输入 token 因此成为规模化 RAG 的主要成本项。Query-aware compression 在检索之后、回答调用之前插入一次"小模型过滤"：更便宜更快的模型（如 Claude Haiku）读取查询与全部检索块，只挑出直接相关的**逐字 span**，主模型（如 Claude Sonnet）只在这份压缩证据上作答。AWS 实测把送入主模型的 token 压到基线 12%（约 8.6×），成本降到 67%，四维答案质量仍为基线 97.5%。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

## 核心要点

- **位置**：唯一新增环节，夹在检索与生成之间，检索器、向量库、主模型均无须改动。
- **本质是抽取不是摘要**：严格约束为"逐字复制相关 span"，禁止改写、总结或自有措辞，保证引用可追溯、原文措辞可审计。
- **支配变量只有两个**：小/主模型单价比，以及压缩比 `c`（送入主模型的 token 缩减倍数）。
- **实测**：token 降至 12%（8.6×）、成本降至 67%（省 33%）；叠加 rerank 后 token 降至 10%（10.1×）、成本降至 64%（省 36%）。
- **质量代价有方向性**：correctness 与基线相差 ≤0.07，completeness 与 citation accuracy 略降，conciseness 略升。
- **幻觉率同步下降**：51% → 44% → 38%（基线 / 压缩 / rerank+压缩），无关上下文被移除缩小了幻觉暴露面。
- **延迟净增**：端到端 +19%（叠加 rerank 后 +12%），主模型少处理的时间只能部分回补。

## 深度分析

### 一、机制：把"相关性判断"前移到便宜模型

标准 RAG 流程：嵌入查询 → 向量索引返回 top-k（常见 5–20）→（可选）reranker 重排 → 全部块拼进 prompt → 主模型作答。检索上下文随 top-k 与块大小线性增长，技术文档与法律负载常达每查询数千 input token。Query-aware compression 不改链路，只在拼接与生成之间加一个条件化简写：让小模型看着"这个问题"决定哪些 span 值得保留。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

这与朴素 top-k 截断的根本区别是**查询无关 vs 查询条件化**：高相似度块可能只有两句能回答问题，机械截断不是浪费 token 就是丢失证据；压缩是 span 粒度，可保留块内关键句、并用 `NO_RELEVANT_EVIDENCE` 在块级别判负整块。相比块粒度的 LLM reranking 它评价更细；相比 [[concepts/lost-in-the-middle|Lost in the Middle]] 的注意力衰减，把几千 token 压成几百 token 高密度片段本身就降低了关键证据被埋没的概率。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

### 二、成本结构与经济学边界

成本模型：单次查询 `R` 个检索输入 token、压缩比 `c>1`、答案 `A` token，小模型单价 `P_small_in`/`P_small_out`，主模型 `P_large_in`/`P_large_out`。基线为 `R·P_large_in + A·P_large_out`；压缩后为 `R·P_small_in + (R/c)·P_small_out + (R/c)·P_large_in + A·P_large_out`——新增小模型整次调用，主模型输入项由 `R` 换成 `R/c`。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

两式相减，净节省正比于 `(R − R/c)·P_large_in` 减去压缩调用成本，由此得出三个成立条件：检索上下文足够大、大小模型单价比足够高（Sonnet/Opus 配 Haiku）、每次查询有足够比例可裁。第三条最易忽略——`c` 是任务的函数非常数：宽问题几乎每段都有用、`c` 趋近 1，压缩变成纯增本，这正是 typical query 省 37%、hard query 只省 26% 的来源。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

### 三、AWS 实现：单 Lambda 双调用与提示词工程

实现刻意最小：一个 AWS Lambda 用两次 Amazon Bedrock Converse API 完成编排。上游检索器（Bedrock Knowledge Bases，底层 Amazon OpenSearch Serverless）返回 top-k 块；Lambda 第一次调用让小模型当压缩器，第二次让主模型当答题器；模型 ID 取自环境变量，客户端启用自适应重试，压缩调用 `temperature=0.0` 固定随机性让小模型稳定"照抄"。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

决定成败的是压缩提示词，须做到四点：抽取 span 而非摘要、禁止改写、保留 chunk 标识符、无证据时输出 `NO_RELEVANT_EVIDENCE`；输出约束为 `[CHUNK_ID: <id>]` + 逐字 span 的严格格式，答题提示只写"仅依据提供的证据回答并标注来源"。这让引用标记从检索块一路透传，是可审计的基础。小模型选型看相对价格、推理速度、模型族对齐（Haiku 配 Sonnet、Nova Micro 配 Nova Pro）。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

### 四、压缩比—质量—延迟的三方权衡与测量方法论

压缩会增加一次调用（成本 + 延迟），换回主模型侧大幅 token 缩减，必须实测。AWS 的评估设计值得照搬：50 万+ 文档、9 类企业来源，500 问分 10 类；三条流水线（不压缩 / 压缩 / rerank+压缩）跑**同一批检索结果**，使差异只归因于压缩。质量由 LLM judge 按四维打分（correctness、completeness、citation accuracy、conciseness）并单独追踪 faithfulness。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

结论：quality 复合分 97.5%、correctness 近乎无损，但 citation accuracy 与 completeness 轻微下滑——这是"只保留 span"的机制性代价，也说明必须逐维度看。延迟上，小模型快且主模型处理更短上下文会回补部分时间，故净增 +19%；先 rerank 后压缩降到 +12% 且更省，说明多个后检索优化叠加有正反馈。边界清楚：亚秒级对话 + 小上下文不适合，应优先 prompt caching 与 Intelligent Prompt Routing。^[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress.md]

## 实践启示

1. **先量 `R` 再决定**：平均检索上下文低于约 5,000 token，或延迟预算无法吸收多出的一次调用（几百毫秒到约 1 秒），就先做 baseline 测量，别急着上。
2. **功夫花在压缩提示词上**：逐字抽取、禁改写、保留 chunk id、无证据输出 `NO_RELEVANT_EVIDENCE`、温度 0.0——这五点决定压缩比能到多少、引用还准不准。
3. **用同一批检索结果做 A/B 并逐维度验收**：correctness / completeness / citation accuracy 对参考答打分，faithfulness 对证据打分，先设阈值再放量。
4. **小模型按单价比 + 推理速度 + 同族对齐收敛**：同族配对能减少两跳间的格式与指令遵循偏差，降低解析失败率。
5. **与既有能力叠加而非替代**：prompt caching、Intelligent Prompt Routing、Rerank API 与本模式是复合关系；先 rerank 再压缩更省且延迟更低。
6. **灰度上线并保留回退**：放在 feature flag 后用小流量跑真实分布再放大；给压缩调用设超时与失败回退到未压缩上下文，避免其成为新单点。

## 相关实体

- [[entities/rag-chunking-vectorization-rerank-distillation|RAG 分块/向量化/召回/重排]]
- [[entities/agentic-retrieval-for-amazon-bedrock-managed-knowledge-base|Amazon Bedrock 托管知识库的 Agentic 检索]]
- [[entities/amazon-bedrock-claude-prompt-cache-strategy|Bedrock Claude Prompt Cache 策略]]
- [[entities/llm-as-a-judge-agent-eval-offline-huolala-2026|LLM-as-a-Judge 离线评测]]
- [[concepts/retrieval-augmented-generation-rag|RAG 检索增强生成]]
- [[concepts/context-management-agent-systems|智能体系统的上下文管理]]
- [[concepts/lost-in-the-middle|Lost in the Middle]]

→ [[raw/articles/reduce-rag-costs-on-amazon-bedrock-with-query-aware-compress|原文存档]]
