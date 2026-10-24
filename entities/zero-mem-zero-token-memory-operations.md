---
title: "Zero-Mem — LLM Agent 的零 Token 记忆操作"
created: 2026-08-06
updated: 2026-10-25
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

## 深度分析

### 一、「零 token」是账本重划，不是成本消失

论文把 zero-token 定义为：除最终 QA 外，构建、组织、路由、检索、证据闭合与两次校准都不调用 LLM、不消耗输入输出 token，encoder 单独核算^[raw/articles/zero-mem-zero-token-memory-operations.md:137-140]。它也自划边界——作者写明 zero-token 不等于零计算，encoder、组织、检索与校准仍有开销^[raw/articles/zero-mem-zero-token-memory-operations.md:1508]。成本是被搬走而非消失：从「调用次数 × 上下文长度」搬到「一次性 encoder 前向 + CPU 侧检索」，前者随轮数复利，后者随语料近似线性、可离线摊销。量级上最省的 baseline 仍要 569 token/查询，Zero-Mem 为 0、0.22 秒/查询^[raw/articles/zero-mem-zero-token-memory-operations.md:1285-1345]；移出账本的是 LLM token，不是 GPU 秒。

### 二、双视图不是 ensemble，是两种失败模式的分工

论文诊断：平铺的词法或稠密检索会把不同用户、session、时间状态下语义相似的 trace 混在一起，并在证据分散于多次交互时失效^[raw/articles/zero-mem-zero-token-memory-operations.md:125]。两个视图各对付一半：entity–context graph 用 NER 实体共现加 trace 邻接边，解决跨交互的分布式证据；temporal hierarchy（turn / window / episode / local）保留顺序与会话状态^[raw/articles/zero-mem-zero-token-memory-operations.md:291-437]。消融显示 HotpotQA 56K 下全模型 72.07，只留图视图 62.50，只留层级 54.88^[raw/articles/zero-mem-zero-token-memory-operations.md:1514]——各塌陷 10 至 17 分，说明两者互为前提：图找得到跨文档连接，但要层级给出周围会话才解读得了；层级能定位事件，却拼不出多跳链。增益主要来自结构本身，closure 与 calibration 只是边际修正。

### 三、确定性校准是 fail-open 的约束满足

证据层先 Filter 掉违反 [[concepts/memory-source-provenance|provenance]] 与 query boundary 的候选，再按 subject / temporal / answer-type 排序，只排序不改写；答案层抽出 evidence-local 候选，初始答案有支撑且格式合法就保留，否则只做归一化、抽取式截短或列表剪枝，标量答案仅在存在唯一 type-compatible 候选时替换^[raw/articles/zero-mem-zero-token-memory-operations.md:1154-1284]。这是明确的 fail-open：宁可不动，也不引入无依据的改写。方向与 LLM-mediated memory 相反——后者能抽象合并，但被合并的主体与模糊的时间更新让证据不可追溯；前者每段证据都落回 unit_id 与原文，代价是表达力被规则封顶，修不了「格式对但事实错」。

### 四、故障迁到索引侧，RAG 被降级为种子

生成式记忆的典型故障是 [[concepts/memory-consolidation-decay|摘要漂移]]：中间表示一错，后续检索全在错误前提上跑，而原文已被覆盖。Zero-Mem 关掉这条路径——原文即 source of record，图与层级只是可重建的派生索引^[raw/articles/zero-mem-zero-token-memory-operations.md:291-292]。故障迁到索引侧：NER 漏实体则图上无边、传播不到，出现召回空洞；profile 抽错会选错视图，但 ρ=0.6 的融合让错路由只降权不归零^[raw/articles/zero-mem-zero-token-memory-operations.md:482-493]；主候选不足时 closure 补不回来，Top-1 到 Top-5 的 52.59→59.15 是全文最大跃迁^[raw/articles/zero-mem-zero-token-memory-operations.md:1536-1563]。运维上的关键差异：索引错了能重建并 A/B，摘要错了只能回滚重跑。

它也不是反 [[concepts/rag-retrieval-augmented-generation|RAG]]。内部仍用 BM25 与 BGE-M3，但限定其只服务索引、播种与打分^[raw/articles/zero-mem-zero-token-memory-operations.md:437]：RAG 让相似度直接决定进入 context 的内容，Zero-Mem 只让相似度决定从哪个实体出发。与 [[entities/agent-harness-context-management-working-set|harness working set]] 则是正交问题——它管跨 session 从原始轨迹捞哪一段，harness 管本轮放什么进窗口，属 [[concepts/working-set-vs-long-term-memory|working-set 与长期记忆]] 的存储/检索分层；也不同于 [[entities/rag-vs-agentic-retrieval-deep-dive-yunduojun-datastudio-2026-09-01|agentic retrieval]] 的多轮搜索：那是用 token 换精度，这是用结构换 token。

| 维度 | Zero-Mem 占优 | 应改用生成式 / 向量记忆 |
|---|---|---|
| 写入模式 | 高频 append，写一次不调 LLM | 写入时就要语义归并 |
| 证据追溯 | 强，可回指 unit_id 与原文 | 弱，摘要不可回指 |
| 模糊语义召回 | 弱，靠 dense 兜底 | 强，embedding 原生 |
| 跨模态 / 冲突消解 | 不覆盖 / 硬过滤 | 原生支持 / 可叙述覆盖 |
| 成本形态 | encoder + CPU，可预测摊销 | token，随轮数复利 |

## 实践启示

1. **先划账本边界再谈省钱。** 把成本拆成 LLM token、encoder GPU 秒、检索 CPU 秒三栏再判断是否真省：0 token 对应 0.22 秒/查询，LightMem 是 569 token 与 0.51 秒/查询^[raw/articles/zero-mem-zero-token-memory-operations.md:1285-1345]。只有 token 本身是瓶颈时这笔置换才划算。

2. **原文即 source of record，派生索引必须可重建。** 落地是一张 trace 表加 unit_id、session 时间与 boundary 元数据，实体图与层级切分全部作为可丢弃、可重建的索引^[raw/articles/zero-mem-zero-token-memory-operations.md:291-292]。可重建性等于可回滚，这是相对生成式记忆最大的运维优势。

3. **用 NER 建图，不要用 LLM 抽三元组。** spaCy 级 NER 犯的错是「漏」，漏掉的实体可被 dense 打分兜底；LLM 抽三元组犯的错是「编」，编出来的关系无从发现。图上只记观察到的共现与邻接，不生成推断关系^[raw/articles/zero-mem-zero-token-memory-operations.md:291-312]。

4. **召回宽度优先于精排。** 按 episode → window → turn → local 逐层下降，先把主候选压到 Top-5 至 Top-10 再做 closure：Top-1 到 Top-5 带来 6.5 分 F1，Top-5 到 Top-10 只再涨 0.65^[raw/articles/zero-mem-zero-token-memory-operations.md:1536-1563]。预算先花在宽度上，并按问题类型分设阈值——单跳早饱和，多跳、时序、开放域需要更宽覆盖。

5. **校准只做可判定的事，并用四个指标盯住记忆层。** 硬过滤 provenance 与 boundary、校验 answer-type、格式归一、列表剪枝都安全；用规则改写语义则必然引入静默错误，故保持 fail-open^[raw/articles/zero-mem-zero-token-memory-operations.md:1269-1284]。观测上 recall@k 按 single-hop / multi-hop / temporal / open-domain 分开看，另加 stale rate 与 dedup precision，才能把零 token 背后的质量损失定位到具体模块，可参照 [[concepts/memory-derivation-observability|记忆派生可观测性]] 与 [[entities/agent-memory-evaluation-landscape-taobao-survey|Agent 记忆评测全景]]。

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
