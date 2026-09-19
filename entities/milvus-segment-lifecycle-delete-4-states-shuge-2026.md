---
title: "拆解 Milvus Segment 生命周期：L0/L1/L2 三级分层与删除机制源码分析"
authors:
  - 术哥
created: 2026-07-05
updated: 2026-09-19
source: wechat
url:
type: entity
tags: [milvus, vector-database, segment, compaction, deletion, delta-log, source-code-analysis, shugex, database-internals]
review_value: 9
review_confidence: 9
review_stars: 5
provenance_state: extracted
sources:
  - raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 拆解 Milvus Segment 生命周期：L0/L1/L2 三级分层与删除机制源码分析

## 核心概述

Milvus 向量数据库的 Segment 是数据的物理组织单位，也是 compaction 的操作对象。本文基于 Milvus 源码（`internal/datacoord/`）和官方 design docs，深入拆解 Segment 的 L0/L1/L2 三级分层体系、Growing→Sealed→Flushed→Dropped 四状态生命周期，以及专为删除数据设计的阻塞机制。^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

→ [[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026|原文存档]]

## 三级分层演进

Segment 层级定义在 `pkg/proto/data_coord.proto`，包含 `Legacy=0`、`L0=1`、`L1=2`、`L2=3` 四个枚举值。`Legacy` 是 proto zero value，代表分层引入前的老 segment，按 L1 处理。^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

| Level | 职责 | 初始状态 |
|-------|------|----------|
| L0 | delta data（删除/更新增量），channel 级别 | 直接 Flushed |
| L1 | 正常写入数据（insert binlogs），partition 级别 | Growing |
| L2 | 带额外数据分布信息 | Flushed + IsInvisible |

## L1 状态机

L1 Segment 经历四个核心状态：^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

1. **Growing** — 新建，接收写入
2. **Sealed** — 触发 seal（6 种策略：容量/生命周期/binlog 文件数/空闲超时/L0 阻塞/总体积超限）
3. **Flushed** — DataNode 写完 binlog 上报
4. **Dropped** — 被 compaction 消费后标记，GC 确认后物理删除

`SaveBinlogPaths` 是状态转换的核心入口（`services.go:627-669`），同时处理 Flushed 和 Dropped 两种转换。

## L0 删除机制

L0 是设计最精妙的层级：^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

- **删除不进 segment 路由**：删除消息不直接写到对应 partition 的 segment，而是攒到 channel 级别的 L0（中转站）
- **一出生就是 Flushed**：DataNode flush delete buffer 后直接注册为 Flushed
- **`sealByBlockingL0` 阻塞策略**：L0 积压到阈值时，强制 seal 时间范围重叠的 growing segment，保证删除可收敛（用写入吞吐换删除一致性）
- **L0 compaction**：删除标记合并进早于触发位置的 L1/L2 segment 的 deltalogs 中

删除在 Milvus 中的完整旅程：用户 delete → DML channel → DataNode 内存 → L0 deltalogs → compaction 合并进 L1/L2 → 查询时过滤。

## L2 的尴尬定位

L2 是正在被淡化的层级：^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

- v2.4：Clustering Compaction 过程中临时标记 L2，完成后清回 L1（中间态标记）
- v2.5.0+：过程中不再标记 L2，失败的结果直接标 Dropped
- 当前：L2 从中间态退化成了 Clustering 产物的身份标签

## 设计取舍

| 选择 | 优点 | 代价 |
|------|------|------|
| L0 单独存 delta | 删除无需在 segment 间路由 | 查询需额外加载 L0 deltalogs，删除多了变慢 |
| sealByBlockingL0 | 保证删除收敛 | 牺牲 growing segment 写入吞吐 |
| GC 安全距离 | 防止删除过早导致查询失败 | 老 binlog 多占存储 |

## 深度分析

### 分层存在的理由：把删除的路由代价从写入路径挪到查询路径

Milvus 的数据按 segment 物理切分，删除消息若直接写进对应 partition 的 segment deltalogs，一个 collection 可能成百上千个 segment，路由开销随 segment 数增长。源码把 L0 定义在 channel 级别（proto 注释 `for current channel`），让删除先落进整个 DML channel 共享的"中转站"，避开 segment 级路由。这本质上是把成本从写入侧换到查询侧：写入侧不必关心 PK 落在哪，查询侧则必须额外加载 L0 deltalogs 做过滤。

分层因此不只是"数据分几类"，而是划出了不同生命周期数据的合并策略边界——通用 compaction 路径的过滤条件同时排除 L0 与 L2，只有 L1 的 Flushed segment 走通用逻辑。同一逻辑也解释了 segment 大小的度量演进：`max_row_num` 被标记 `deprecated`，注释明说改用二进制大小而非估算行数来控制 segment 大小，因为不同向量维度、不同数据类型下同样的行数体积差太多。^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

### 状态机是一份 compaction 契约，跳过一步就要靠强制封口来补

L1 的 Growing→Sealed→Flushed→Dropped 不是纯粹的记账状态。`SaveBinlogPaths`（`services.go:627-669`）是状态转换的唯一核心入口，四十来行同时处理 Flushed 与 Dropped 两种转换；Growing→Sealed 则由 SegmentManager 注册的六种 seal 策略触发（容量、生命周期、binlog 文件数、空闲超时、`sealByBlockingL0`、总体积超限）；最终物理删除在 `garbage_collector.go:767` 的 `recycleDroppedSegments`。

契约的关键在 Growing：**growing segment 不参与 compaction**。L0 compaction 要把 delta 应用到该 channel 上所有早于 L0 dmlPos 的 sealed/flushed segment，可只要一个 growing 还活着、还在接收写入，删除就无法完整收敛——应用完 delta 之后它完全可能再写进一条同 PK 的数据。所以系统宁可强制 seal 时间范围重叠的 growing，用写入吞吐换删除一致性，这也是"删除风暴导致写入被强制封口"的完整因果。

状态机还为失败留了回退路径：`LastLevel` 配合 `RevertSegmentLevelOperator`，改 level 前先存旧值，compaction 失败时回滚——level 变更本身被当作可能中途失败的写操作。^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

### 删除在物理消失前只是 delta 加一个时间边界

删除标记在很长一段时间里并不"落在数据上"。L0DeleteCompaction 挑目标 segment 的条件在 `compaction_task_l0.go:305-340`：level 不能是 L0，且 `segmentEffectiveTs(info.SegmentInfo) < taskProto.GetPos().GetTimestamp()`——一批 L0 的删除只被合并进**早于该 L0 触发位置**的 L1/L2 segment，更新的 segment 不在这轮范围内。删除的可见边界因此是被时间戳划出来的，这也是 L0 必须和 growing 封口协作的原因（参见 [[entities/milvus-3-0-search-aggregation-pushdown-shuge-2026|Milvus 3.0 检索聚合下推分析]]）。

查询侧同样按这条线付费。`handler.go:360-371` 的 segment view 分类把 L0 单独拎进 `levelZeroIDs`，QueryNode 需要额外加载 L0 deltalogs，在结果里过滤掉已删 PK，开销随 L0 积压增长；直到 L0 compaction 把 delta 合并进 L1/L2 的 deltalogs（那部分会被索引加速）才降下来。物理回收更晚：`recycleDroppedSegments` 必须确认 `compactTo` 目标 segment 已建好索引、且未被任何 QueryNode 加载，才允许删源文件的 binlog——用空间换可靠性，防止"源删了、目标还没顶上"导致查询失败。^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

### L2 的尴尬定位是一类"边跑边改"的演进信号

L2 的问题不是它没用，而是语义在版本之间漂移。v2.4 里 Clustering Compaction 过程会临时把 segment 标成 L2、完成后清回 L1，L2 只是中间态标记；v2.5.0 之后过程中不再标记 L2，失败的中间结果直接标 Dropped（`don't mark segment level to L2 before clustering compaction after v2.5.0`）。但 L2 也没被废弃——当前版本 Clustering 产出新 segment 时仍标记 L2（`meta.go:2371`）并带 `IsInvisible: true`，等 stats 和 index 建完才可见，它从"中间态"退化成"产物的身份标签"。

同一时期源码里还有一串同类信号：`max_row_num`、`SegmentIDRequest.Level` 相继标记 deprecated，`AssignSegmentID` 被新的 `AllocSegment` 取代。结论很直接：**按 level 做运维抓手（监控、容量规划、外部工具判断）会踩到版本差异**，判断某条路径处理哪一层，应看 compaction policy 的过滤条件而非层级的字面语义。^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

## 实践启示

1. **先看 L0 积压，再怪写入侧**：`BlockingL0SizeInMB` / `BlockingL0EntryNum` 是 `sealByBlockingL0` 的触发阈值；超限时系统强制 seal 时间范围重叠的 growing segment，写入侧表现为吞吐骤降、封口频繁。见到这类抖动先查 L0 数量与体积，而不是先调写入参数。
2. **删除后的时滞要写进预期**：查询必须先加载 L0 deltalogs 过滤已删 PK，所以删除"看起来还没生效"或 P99 上升，在 L0 compaction 完成前都是预期行为。
3. **注意删除的时间边界**：L0 compaction 只把 delta 应用到 `effectiveTs` 早于该 L0 触发位置的 segment。删除后紧接着写入的同一 PK，收敛依赖 L0 与 growing 封口的协作，一次 compaction 并不覆盖；批量删除后立即重写同一批 PK 是高危模式。
4. **不要把 L2 当作运维抓手**：L2 在 v2.4 与 v2.5.0+ 行为不一致，且 Clustering 产出期间带 `IsInvisible: true`——stats 和 index 建完才对查询可见。Clustering 刚跑完查不到全量数据，通常不是丢数据，而是可见性还没切换。
5. **删数据不等于马上释放对象存储**：Dropped 只是标记，`recycleDroppedSegments` 还要等目标 `compactTo` segment 建成索引且未被 QueryNode 加载才回收 binlog。磁盘水位下降总是滞后于删除；若 compaction 或建索引卡住，老 binlog 会持续占地——监控要同时盯 Dropped 队列与 GC 进度。
6. **升级时核对依赖 level 语义的工具与接口**：`Legacy = 0` 的老 segment 一律按 L1 处理，按 level 分配的 `AssignSegmentID` 已被 `AllocSegment` 取代，`SegmentIDRequest.Level` 也已 deprecated。自研脚本若按 level 分配/判断，升级前需重新验证；容量估算要用二进制体积口径而非行数。^[raw/articles/milvus-segment-lifecycle-delete-4-states-shuge-2026.md]

## 相关实体

- [[entities/zilliztech-mfs-open-tag-claude-tag-shuge-2026|MFS：Zilliztech 的 Agent 统一上下文 harness]] — 同作者（术哥）关于 Zilliztech/Milvus 相关项目
- [[entities/hermes-agent-v014-architecture-shugex|Hermes Agent v0.14 架构源码分析]] — 同作者源码分析系列
- [[concepts/rag-retrieval-augmented-generation|RAG 检索增强生成]] — Milvus 作为向量数据库的核心应用场景
