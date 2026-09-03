---
title: "Snowflake vs Databricks 2026 Summit：Agent 时代湖仓的三层架构进化"
created: 2026-06-30
updated: 2026-06-30
type: comparison
tags: [snowflake, databricks, lakehouse, agent, architecture, data-plane, context-plane, agent-control-plane, comparison, summit-2026]
sources:
  - raw/articles/snowflake-databricks-2026-agent-lakehouse
  - raw/articles/snowflake-agentic-enterprise-summit-2026-infoq
  - raw/articles/databricks-storage-ecosystem-opensharing-govern-everything-2026
---

# Snowflake vs Databricks 2026 Summit：Agent 时代湖仓的三层架构进化

> 本文基于吴炳锡（Databend Labs 联合创始人）的深度分析，拆解 Snowflake 和 Databricks 在 2026 Summit 上释放的同一信号：湖仓的竞争正在从「SQL 多快、存储多便宜」，转向「Agent 多可靠、业务任务多安全」。 ^[raw/articles/snowflake-databricks-2026-agent-lakehouse.md]

## 2026 Summit 产品路线对比

| 维度 | Snowflake 2026 | Databricks 2026 |
|------|---------------|-----------------|
| 主题 | Agentic Enterprise / Making AI Real | Apps and agents that work |
| 业务 Agent | CoWork | Genie One |
| 开发者 Agent | CoCo | Genie Code / Agent Bricks |
| 上下文层 | Horizon Context / Cortex Sense | Genie Ontology |
| 语义指标 | Semantic Views / Semantic Studio | Unity Catalog Metrics |
| AI 治理 | AI Agent Identity / Horizon Guardrails | Unity AI Gateway |
| Operational DB | Snowflake Postgres / Crunchy Data | Lakebase |
| OLTP/OLAP 打通 | Postgres Mirroring / pg_lake / Unistore | LTAP |
| 实时分析 | Interactive workloads / Dynamic Tables / Snowflake RT | Lakehouse RT / Reyden |
| 开放性 | Iceberg + Polaris + OSI + MCP | Delta + Iceberg + open agent frameworks |
| 战略基因 | Enterprise data cloud + governed SaaS-like platform | Open lakehouse + ML/AI platform |

两家公司虽然产品命名不同、技术基因不同，但战略方向已高度一致：业务入口从 BI 变成 Agent，数据目录从 metadata catalog 变成 AI governance/control plane，湖仓底座向 operational data 和 real-time serving 延伸。 ^[raw/articles/snowflake-databricks-2026-agent-lakehouse.md]

Snowflake 更偏「受控的一体化企业平台」，Databricks 更偏「开放的开发者与 AI 平台」。 ^[raw/articles/snowflake-databricks-2026-agent-lakehouse.md]

## Agent 智能湖仓三层架构

Snowflake 和 Databricks 在 Agent 湖仓架构上基本一致，分为三层 ^[raw/articles/snowflake-databricks-2026-agent-lakehouse.md]:

### 第一层：Data Plane — 数据存算

传统湖仓核心，解决「数据能不能被存储和计算」。包括数据存储、计算执行、SQL 查询、批/流处理、表、基础 catalog、数据接入清洗建模、性能和成本优化。

- Snowflake 优势：企业级数仓、弹性计算、数据共享、治理体验
- Databricks 优势：Spark、Delta Lake、开放 Lakehouse、ML/AI 工作负载

### 第二层：Context Plane — 业务语义

Agent 时代最关键的一层，解决「AI 能不能理解业务语义」。Agent 不是直接消费表和文件，而是需要知道用哪个指标、哪个口径、哪个数据源可信。

Context Plane 包括：指标口径、semantic metrics、ontology、商业术语、表/字段描述、数据血缘、data quality signals、usage signals、certified datasets、user/role context、agent memory。

- Snowflake：Horizon Context / Cortex Sense / Semantic Views / Semantic Studio
- Databricks：Genie Ontology / Unity Catalog Metrics / Business Glossary / Domains

### 第三层：Agent Control Plane — 安全治理

解决「AI 能不能安全上生产」。Agent 在生产环境中可能访问敏感数据、跨系统查询、调用 CRM/Jira/Slack/邮件、生成报告、开工单、触发审批、写回业务系统。

Agent Control Plane 包括：agent identity、per-agent RBAC、tool access control、MCP governance、model routing、spend cap/budget、guardrails、prompt injection 防护、data exfiltration detection、audit log、eval/benchmark、human-in-the-loop approval。

- Snowflake：AI Agent Identity、Horizon AI Guardrails、AI Security Posture Management、Natoma MCP Gateway、AI Cost Controls
- Databricks：Unity AI Gateway、Agent Bricks、Omnigent、Catalog Federation、Automatic Identity Management

### 三层产品映射

| 分层 | 解决问题 | Snowflake | Databricks |
|------|---------|-----------|------------|
| Data Plane | 数据能不能存、算、查 | Warehouse、Dynamic Tables、Iceberg、Polaris、Datastream、OpenFlow、Snowflake PostgreSQL | Lakehouse、Delta Lake、Iceberg、Spark、Lakeflow、Lakebase、Lakehouse RT |
| Context Plane | AI 能不能理解数据 | Horizon Context、Cortex Sense、Semantic Views、Semantic Studio、Horizon Catalog | Genie Ontology、Unity Catalog Metrics、Business Glossary、Domains、Unity Catalog |
| Agent Control Plane | Agent 能不能安全上生产 | AI Agent Identity、Horizon AI Guardrails、AI Security Posture Management、Natoma MCP Gateway、AI Cost Controls、CoWork/CoCo governance | Unity AI Gateway、Agent Bricks、Omnigent、Catalog Federation、Automatic Identity Management、Genie governance |

## 为什么必须上移到 Agent 智能湖仓

底层计算能力上限容易达到，过分的 benchmark 效益不明显。同时 Databend、Clickhouse、BigQuery、Redshift、DuckDB、Dremio、Athena、Trino、Spark、Flink、RisingWave、TiDB Lake 等产品的成熟给两者带来巨大压力——在单纯湖仓方向很容易变成给云厂家打工。 ^[raw/articles/snowflake-databricks-2026-agent-lakehouse.md]

随着 AI 出现，Agent 成为新的工作入口，通过系统工程给 AI 提供完整的语义、权限管理、上下文、成本控制和审计成为产品更重要的部分。 ^[raw/articles/snowflake-databricks-2026-agent-lakehouse.md]

## 对 Databend 的启示

Databend 的机会不是复制 Snowflake CoWork 或 Databricks Genie，而是把自己做成 **Agent-ready data platform** 的底层执行与上下文供给层 ^[raw/articles/snowflake-databricks-2026-agent-lakehouse.md]:

1. **开放的数据执行层** — 让 Agent 以 SQL/API 的方式访问可信数据
2. **低成本的上下文供给层** — 承载 JSON、trace、日志、文档、向量等多形态数据
3. **Snowflake-like 但更开放的替代底座** — 通过 S3-native、Parquet-native、开源内核和更透明的成本模型，降低企业把 Agent 接入数据平台的复杂度和成本

## 核心判断

> 湖仓下一站不只是更快的 SQL，也不只是更便宜的存储，而是：**让 AI Agent 能够在可信数据、明确语义和可控权限之上，安全地完成业务任务**。这会是下一代数据平台竞争的核心。 ^[raw/articles/snowflake-databricks-2026-agent-lakehouse.md]

→ [[raw/articles/snowflake-databricks-2026-agent-lakehouse|原文存档]]
→ [[entities/snowflake-agentic-enterprise-summit-2026|Snowflake Agentic Enterprise]]
→ [[entities/databricks-storage-ecosystem-opensharing-govern-everything-2026|Databricks Storage Ecosystem]]
