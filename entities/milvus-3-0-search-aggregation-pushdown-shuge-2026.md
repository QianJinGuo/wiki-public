---
title: "Milvus 3.0 聚合下推：3 层隔离、2 条路径、4 倍安全因子"
created: 2026-08-02
updated: 2026-10-26
type: entity
tags: [milvus, vector-database, search-aggregation, group-by, order-by, segcore, proxy, ann-search, source-code-analysis, shugex, database-internals]
sources: [raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026]
confidence: 0.6
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Milvus 3.0 聚合下推（Search Aggregation & Order By）

Milvus 3.0 把分组、聚合、排序从应用层搬进数据库内部，在 ANN 搜索的近似性之上用一层隔离架构补上分析层。核心设计取舍：向量检索本身是近似的，建立在它之上的聚合**承认近似，用工程手段在精度和延迟之间权衡**。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

## 三层隔离架构：Segcore 只做最笨的事，Proxy 包揽所有聪明的事

| 层 | 职责 | 聚合感知 | 嵌套感知 | 排序感知 |
|---|---|---|---|---|
| Segcore | 多字段 flat composite-key group-by | 否 | 否 | 否 |
| QueryNode / Delegator | 合并多 segment 结果，按 composite key 去重 | 否 | 否 | 否 |
| Proxy | 解析 GroupBy 树 → 构建 context → 接收 flat 结果 → 重建层级 + 指标 + 排序 | 是 | 是 | 是 |

Segcore 眼里没有聚合、嵌套、指标，只认「composite key 列表 → 按 key 分组 → 每组返回 top-N」。所有复杂逻辑（层级展开、bucket 指标、排序）压到 Proxy 层。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

**Top-K 放大公式**：`segcore_topk = Π(level.size) × group_count_safe_factor`，上限 65535。例：10 个 category × 每个 5 个 brand → 10×5×4=200，多拉 4 倍数据降漏桶概率。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

**设计动机**：Segcore 是 C++ 跑在查询引擎热路径上，复杂聚合/嵌套重建会严重影响延迟；重逻辑放 Go 写的 Proxy 层——改聚合逻辑不需要动 Segcore 二进制。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

## 两条路径：Legacy GroupBy vs Search Aggregation

| | SearchGroupBy（legacy） | SearchAggregation（新） |
|---|---|---|
| 触发条件 | `group_by_field_id > 0` | `agg_info != nil` |
| 线载体 | SearchResultData.group_by_field_value (field 8) | SearchResultData.fields_data |
| 能力 | 单字段去重 | 多字段 composite key、分组内指标、bucket 排序、嵌套子分组 |

用户文档严重落后于实现：v3.0.x 官方 grouping-search 页面只覆盖基础单字段参数，新聚合 API 只字未提——与功能默认关闭有关。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

## 近似不是 Bug，是设计选择

官方口径：Search aggregation 在 ANN 检索结果集上进行而非全量数据，分面计数是近似值；需要精确计数用 query-side aggregation（全量扫描）。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

近似三层面：
1. **Bucket 存在性不保证**——ANN 检索池外的 key 不可见
2. **count 只统计检索池中的行**——实际 20 个片段只召回 5 个，count 就是 5
3. **嵌套指标偏差随层级传播放大**——层级越深偏差越严重

缓解：group_count_safe_factor（默认 4），类似 Elasticsearch 的 shard_size。判断标准：实时推荐场景近似聚合够用；BI 报表要精确计数用 query-side。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

## Search Order By vs Query Order By

**Search Order By**：完全在 Proxy 层执行（`reduce → merge_ids → requery → gen_ids → organize → pick → order_by`，internal/proxy/search_pipeline.go:1808）。限制：必须 requery 获取字段值（多一次网络往返）；排序无下推。支持多字段（优先级递减）、JSON 子路径（metadata["price"]）、sort.SliceStable 保持等值行原始顺序。微妙点：按每组 **top entity** 的标量值排序组（取每组第一行 price，非平均值）。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

**Query Order By**：下推 Segcore 执行，SortBuffer 组件（internal/core/src/exec/SortBuffer.h）Velox 风格——只排序指针（8 字节）不移动行数据；多字段独立 ASC/DESC 和 NULLS FIRST/LAST；limit < n/2 时 partial_sort 做 O(n log k)。已知限制：无 max_sort_rows 守卫，超大结果集排序有内存风险。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

分化原因：Query 全量扫描数据在 Segcore 本地，排序跟着数据走可下推；Search 是 ANN 结果分散多 segment，必须 Proxy 合并后排序。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

## 共享聚合框架 internal/agg/

AggregateBase 接口五方法（Update / NewState / UpdateState / Terminate / ToPB），Sum/Count/Min/Max/Avg（拆 sum+count 两 slot）实现，Search 和 Query 两条路径复用同一套指标计算。时间线：20260130 三层架构 → 20260203 Query 排序 → 20260413 Search 聚合完整 MEP。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

## 落地状态与行业位置

`proxy.search.embeddedAggregation.enabled` 在 3.0 GA 默认 **off**。Phase 1 边界：指标只支持 count/sum/avg/max/min；bucket 排序支持 _count/_key/指标别名；不支持 hybrid search + group_by、highlight、JSON 字段、limit 与 group_by 同用；metric_safe_factor 是 proto 保留字段（no-op）。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

Milvus 3.0 不是变成 Elasticsearch：ES 聚合基于全量倒排索引（精确），Milvus 基于 ANN 检索结果集（近似）。Airbyte 测试：纯向量搜索 Milvus 快约 15%、p95 好约 20%；混合负载仍是 ES 强项。行业方向一致：SingleStore 实时聚合+向量、pgvector 混合负载、MariaDB 2026 加向量搜索——向量数据库从「存向量搜向量」挪向分析型数据库。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

→ [[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026|原文存档]]

---

## 深度分析

### 下推的本质：把计算搬到基数最小的一侧

聚合下推不是「把 SQL 塞进向量库」，而是「把计算搬到基数最小的一侧」。ANN 检索天然把 N 条压成 K 条，K 就是全链路最窄的截面；把 top-K 拉回应用层再 group by 要付网络、序列化、GC 三笔钱，放 Proxy 只省最后一跳，放 Segcore 才省整条路。

但收益有边界。设召回池 R 行、G 组、聚合字段 f：聚合约 O(R·f) 与 G 弱相关，输出约 O(G×层数)，压缩比 R/G 决定值不值——G≪R（category、brand、source_doc 这类低基数维度）时下推把大段数据压成几十行，收益最大；G≈R（高基数字段上 group by）时聚合退化成排序，下推是纯开销。

`segcore_topk = Π(level.size) × safe_factor` 是关键旋钮：R 随层级 size 的乘积增长而非随 N，嵌套代价因此是乘性的。这也解释了深度为何锁死 4 层——第 5 层的乘积撞上 65535，安全因子被吃光。而越界是静默截断不报错：4 层 × 每层 20 组 × 4 倍 = 32 万，系统只能截，用户拿到的「部分结果」没有任何标志位——最危险的降级形态。

### 近似聚合的「正确性契约」

SQL 聚合有精确语义：GROUP BY 必返回所有组，COUNT 是定值。Search Aggregation 继承的是 ANN 的近似，语义更像「在召回集上做一次有偏抽样」：想算全量 COUNT(g)，实际拿到池内 COUNT(g)，两者相等仅当该组全部成员都落在 top-K 内或组内成员在池内均匀采样。而 ANN 池按相似度排序，绝不均匀——偏差是系统性的，靠近 query 的组被高估，边缘组直接消失。抽样误差能靠扩大样本收敛，系统性截断不能。

四件事必须知道：count 的分母是召回池，跨 query 不可横向比较；缺失的 bucket 不等于 0，而等于「未知」；嵌套偏差乘性传播，第一层漏桶后第二层在错误集合里再漏一次；Phase 1 里 `metric_safe_factor` 是 proto 保留字段（no-op），只有 `group_count_safe_factor` 起作用。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

对照 [[entities/elasticpp重塑elasticsearch查询性能的c内核引擎|Elasticsearch 的 C 内核查询引擎]]：ES 的 terms 聚合建于全量倒排，`doc_count` 精确，`shard_size` 只应对分片间 top-N 截断；Milvus 的近似来自数据根本没被全部看到。搬 ES 的聚合经验过来会低估偏差量级。

### 分层隔离与双路径并存：一次教科书式的渐进交付

三层隔离（Segcore 无感知 / Delegator 无感知 / Proxy 全感知）表面为性能而设，实则是一条变更成本隔离带：能力演进全部收敛在 Go 写的 Proxy 里，Segcore 的二进制契约不动。这相当于把控制面（「用户要什么分组树」）与数据面（「按这个 key 分组的前 N 行」）拆开，接缝选在两者最窄处——一个扁平 composite key 列表。接口越窄，演进越自由；代价是 Proxy 承担全部复杂度，成为瓶颈就无处可推，Search Order By 多出的那次 requery 往返就是账单。

Legacy GroupBy 与新路径并存，是同一思路在时间轴上的复刻：独立触发条件（`agg_info != nil` vs `group_by_field_id > 0`）加独立线载体（`fields_data` vs field 8）挂在同一份 `SearchResultData` 上，互不干扰。不改协议、不加版本协商，靠「触发条件正交 + 载体独立」做出事实上的特性开关——这正是有状态引擎里零停机弃用的标准形态，与 [[entities/milvus-segment-lifecycle-delete-4-states-shuge-2026|Segment 生命周期分级]] 那种「靠状态分层而非大版本切换完成替换」同源。代价同样真实：双路径维护、文档分叉（官方 grouping-search 页面至今只写 legacy）、共享 `internal/agg/` 后的隐性耦合。^[raw/articles/milvus-3-0-search-aggregation-pushdown-shuge-2026.md]

### 对 RAG / Agent 检索的推论

[[concepts/rag-retrieval-augmented-generation|RAG]] 默认「top-K + 上下文」，K 由 token 预算而非统计需求决定；但越来越多问题需要「top-K + 统计」：这类条款有几个版本、哪个来源覆盖了大部分证据、还有哪些类别可选。答案不是一条更好的 chunk，而是一个分布。Search Aggregation 把「拉 200 条到应用层用 Counter 数一遍」变成一次查询：先 group by source_doc 看证据集中在哪几个文档，再对这些文档加深检索，比盲目扩 K 更省 token。但 count 是池内计数，「覆盖率」「占比」不可用，可用的只有存在性与相对排序——排序在近似池内仍单调。分野是：**用它的序，不要用它的值**。

## 实践启示

1. **上线前先算 topk 乘积，再决定嵌套深度。** 把 Π(level.size) × safe_factor 算出来并留余量；越界是静默截断不报错。3 层 × 20 组已逼近 65535，4 层只适合每层个位数分组。
2. **只在高压缩比字段上 group by。** R/G 决定收益：category、brand、source_doc 值得下推；user_id、session_id、chunk_id 近似唯一，group by 等于排序外加一层聚合开销。
3. **用 ratio 做精度门槛，而不是靠感觉。** 抽样 query 并行跑 search-side 与 query-side count，ratio 与组集合都稳定才允许进看板。
4. **把「序」和「值」分开消费。** bucket 排序（`_count` / `_key` / 指标别名）在池内可靠；绝对 count、占比、覆盖率不可靠。UI 说「最相关的前 5 个来源」安全，说「共 137 条」就是误导。
5. **别依赖 Search Order By 的组排序。** 它按每组 top entity 的标量值排序（取首行 price，不是均值），还要多一次 requery 往返；要按指标排组用 bucket 排序。与 [[entities/rag-chunk-embedding-rerank-pipeline|分块-嵌入-重排流水线]] 对接时，group by 放在召回后、重排前，用分布收窄候选集比加大 rerank top-N 更省算力。
6. **3.0 GA 上先别默认开。** Phase 1 不支持 hybrid search + group_by、highlight、JSON 字段、limit 与 group_by 同用；把聚合当独立查询发，别和混合检索塞进同一条 pipeline，否则出错位置难以定位。

## 关联
- 姊妹篇: [[entities/milvus-segment-lifecycle-delete-4-states-shuge-2026|Milvus Segment 生命周期（L0/L1/L2 三级分层与删除机制）]] — 同为术哥源码分析系列，聚合 vs 生命周期
- 同类: [[entities/3-倍于-vectordbbench-榜首火山-milvus-如何把向量检索拉到新高度|火山 Milvus 向量检索性能]] — 检索性能 vs 检索后分析
