---

title: "ClickHouse Ingestion at Scale: An Open-Source Zepto Engineering Story"
description: "Deep technical engineering story with specific performance tuning details, batching optimizations, and open-source contributions. High practical value for scale engineers."
created: 2026-06-22
updated: 2026-07-27
type: entity
tags: [data-engineering, clickhouse, cdc, analytics, startup, architecture]
source: [[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story]]
sources:
  - raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story
review_value: 8
review_confidence: 9
review_recommendation: strong
review_stars: 4
---

# ClickHouse Ingestion at Scale: An Open-Source Zepto Engineering Story

Markdown Content:
[![Image 1: Zepto Tech](https://miro.medium.com/v2/da:true/resize:fill:64:64/0*Zdo4al9KE5LuqNxm)](https://medium.com/@tech.culture?source=post_page---byline--7f57309e2175---------------------------------------) ^[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story.md]

![Image 2](https://miro.medium.com/v2/resize:fit:700/0*RaaiaRuTYvCKxyMs.png)

Much like our journey described in[_Debezium at Scale_](https://blog.zeptonow.com/debezium-at-scale-an-open-source-cdc-story-from-zepto-aa4b12e32bf7), our architecture relies heavily on real-time data flow. To understand user journeys, track operational metrics, and power our growth, we built **Lucid** — Zepto’s completely in-house product analytics engine designed to replace Mixpanel. ^[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story.md]

Lucid captures millions of events per minute, routes them through Kafka, and dumps them into ClickHouse to give us lightning-fast, high-precision insights without the third-party SaaS pricing trap. We use **Confluent Cloud** to manage our Kafka infrastructure and the**in-house** ClickHouse Sink Connector. It was seamless — until our scale broke the default physics. ^[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story.md]

Every hyper-growth engineering team eventually hits a wall where managed abstractions turn from a blessing into a bottleneck. For us, that wall appeared right at the intersection of Apache Kafka and ClickHouse. ^[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story.md]

To ingest billions of events into ClickHouse for **Lucid** at a sustained throughput of **10 MB/s (peaking up to 15–20 MB/s),** we hit a wall with Confluent Cloud’s managed infrastructure because its managed nature restricted our access to low-level broker and connector tuning. Instead of migrating our entire Kafka ecosystem, we proved our batching hypothesis on an In-house Kafka Proof-of-Concept, and then built that buffering logic directly into the open-source ClickHouse Kafka Connect framework. By rewriting core parts of the connector, we boosted ingestion by **45%**, eliminated crippling GC pauses, and drastically reduced ClickHouse insert pressure. ^[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story.md]

This is the story of how we overcame the **black box of managed cloud**, the hidden performance killers we found in the open-source connector, and the two major pull requests we merged to fix them and contribute back to the community. ^[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story.md]

## The Inciting Incident: The Confluent Cloud Black Box

→ [[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story|原文存档]] ^[raw/articles/clickhouse-ingestion-at-scale-an-open-source-zepto-engineering-story.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

