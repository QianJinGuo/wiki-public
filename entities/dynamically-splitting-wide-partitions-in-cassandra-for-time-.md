---

title: "Dynamically Splitting Wide Partitions in Cassandra for Time Series Workloads"
created: 2026-06-10
updated: 2026-09-13
tags: [aws, code, data, observability, open-source, rag, rl, tool-use, vision, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/dynamically-splitting-wide-partitions-in-cassandra-for-time-
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Dynamically Splitting Wide Partitions in Cassandra for Time Series Workloads

## 摘要

Netflix's TimeSeries Abstraction runs on Apache Cassandra 4.x, where the wide partition is the central failure: average read latency climbs from milliseconds to seconds, tail latency becomes read timeouts, and clusters absorb GC pauses, CPU saturation and thread queueing. The remedy is two-stage — a worker that re-partitions *future* time slices at table level, and an asynchronous per-ID pipeline that detects, plans, splits and re-routes reads transparently. Results: average latency at low double-digit milliseconds, tail latency near 200 ms, 500 MB+ partitions paginated while available. ^[raw/articles/dynamically-splitting-wide-partitions-in-cassandra-for-time-.md]

## 核心要点

- Wide partitions, not cluster capacity, are the binding constraint; scaling the fleet is the expensive default.
- Failure chain: elevated read latency → second-scale tail → timeouts → GC pauses, high CPU, thread queueing.
- Table-level re-partitioning helps only table-wide skew; a minority of hot IDs needs per-ID handling.
- Detection is read-triggered: bytes read trip a threshold that emits a Kafka event for planning and splitting.
- Splits take *immutable* partitions first — less surface area, no write-path coordination, most timeouts still removed.
- Checksums, offline Spark verification and shadow-mode byte comparison guard correctness; the original partition is never deleted.

## 深度分析

### Why wide partitions are the core failure mode for time-series data

Cassandra makes the partition the unit of nearly everything: rows are stored contiguously, replicated together, located by one index entry and read whole. Time-series data collides with that model because the key is typically `(time_series_id, time_bucket, event_bucket)` while events accumulate along time, so an entity's rows keep piling into the same bucket. A wide-partition read scans many rows and merges them across the memtable and every SSTable holding a fragment, so I/O and CPU per logical read far exceed the payload; sustained throughput then consumes read-stage threads faster than data, producing queueing, CPU saturation and second-scale tail latency. Heap pressure compounds it: large row iterators and cell objects allocate heavily per in-flight read, so wide-partition-heavy traffic triggers GC pauses that slow every request on the node; such partitions also compact poorly, being rewritten nearly whole with little parallelism. Tombstones are avoided architecturally instead: discrete time slices let old data be dropped with whole tables rather than range deletes. ^[raw/articles/dynamically-splitting-wide-partitions-in-cassandra-for-time-.md]

### The dynamic split pipeline: detection, planning, splitting, serving reads

Dynamic partitioning is an asynchronous three-stage pipeline — detection, planning/splitting, serving reads — triggered from the read path, not the write path. Every read tracks bytes read per partition; exceeding a threshold publishes a Kafka event carrying the slice, the time-series ID, bucket coordinates, an immutability flag and a reserved version, so only data that actually hurts is caught, at the cost of a few temporarily sub-optimal reads. Detection may rest on a partial read, so the planner re-reads the whole partition once and checkpoints; a `wide_row` metadata table holds the split state machine and the routing data reads later need. Splitting is delegated to a strategy such as `EventBucketPartitionSplitStrategy`, which assigns more event buckets to the same time bucket; ultra-wide partitions cap the target bucket count to bound read amplification, since spreading still distributes load over more replicas. A split is marked COMPLETED only when its post-split checksum matches the pre-split one. Serving is invisible to callers: servers load completed-split keys into in-memory Bloom filters, each read checks one in microseconds, and a hit triggers a cached `wide_row` lookup telling the reader where the data now lives before delegating to the existing `PartitionReader` — possible because the split table keeps an identical schema. ^[raw/articles/dynamically-splitting-wide-partitions-in-cassandra-for-time-.md]

### Dynamic per-ID splitting versus static time bucketing

Static bucketing forces the schema and provisioning decision to be right up front, and failures cluster where the post predicts: workloads unknown or misestimated early, workloads that evolve after launch, and outlier IDs receiving far more events than peers. Too coarse yields wide partitions; too fine yields the mirror-image failure — with 60-second buckets, partitions fell under 10 KB and fixed per-partition and per-read overheads dominated, producing read amplification and thread queueing. The escape hatch: data is already cut into time slices, so future slices can adopt a new scheme without rewriting the past; Netflix automated it with a worker that consumes partition histograms, republishes them via a Cassandra virtual table, and retunes future slices whenever density misses its 2–10 MiB target. That fixes table-wide skew but not per-ID skew — the gap dynamic splitting closes. Migration economics diverge: changing a static partition key is a full data move, since Cassandra cannot re-key a partition in place, whereas dynamic splitting copies only the affected ID into a new same-schema table and diverts reads at the application layer — smaller, but bought with pipeline complexity, asynchronous lag and duplicate storage. ^[raw/articles/dynamically-splitting-wide-partitions-in-cassandra-for-time-.md]

### Storage-engine mechanics: SSTables, indexes, flushes and read fan-out

A partition read is a chain repeated per replica: per-SSTable Bloom filter checks discard SSTables that cannot hold the key, the partition index with its sampled summary locates the data, and rows are merged across the memtable and surviving SSTables. Wide partitions inflate the back end — more rows to reconcile, more fragments to merge — so latency scales with breadth even when a query wants only a slice. Over-partitioning breaks the front end inversely: with kilobyte partitions the Bloom check, index seek, replica coordination and merge setup are fixed costs paid per request regardless of payload, which is why density is tuned to a band. Immutable-only splitting also avoids the hardest problem: with no writes arriving, the splitter reads the partition fully and writes new ones through the ordinary write path (commitlog, memtable, flush, fresh SSTables) with no dual-write coordination. The ultra-wide cap admits that fan-out costs money — more partitions parallelize reads across replicas and cut tail latency, but each re-pays Bloom, index, seek and merge overhead. ^[raw/articles/dynamically-splitting-wide-partitions-in-cassandra-for-time-.md]

## 实践启示

1. Treat partition-size distribution as a production metric — histograms are the earliest signal of over- and under-partitioning.
2. Express density targets in absolute bytes (2–10 MiB) so automation has a threshold and reviewers a criterion.
3. When only a minority of data needs remediation, detect on the read path: catch exactly what hurts, leave the write path untouched.
4. Cut the change surface first — immutable-only splits, identical schema and reader reuse make a migration additive and reversible.
5. Verify independently (checksums, offline reconciliation, shadow comparison) and advance read modes in phases.
6. Budget recurring costs — duplicate storage, Kafka and metadata tables, per-server Bloom memory — not just the win.

## 相关实体

- [[entities/netflix-cassandra-wide-partition-dynamic-splitting]] — deeper companion page for the same source
- [[entities/netflix-druid-interval-aware-caching]] — same interval/time-slice reasoning applied to Netflix's Druid caching
- [[moc/observability-monitoring|MOC]] — latency, timeout and tail-latency monitoring

→ [[raw/articles/dynamically-splitting-wide-partitions-in-cassandra-for-time-|原文存档]]
