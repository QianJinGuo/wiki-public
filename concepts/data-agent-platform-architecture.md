---
title: "数据智能体平台架构"
created: 2026-05-25
updated: 2026-08-01
type: concept
tags: [data-agent, nl2sql, multi-agent, code-interpreter, analytics, llm-gateway, architecture, enterprise]
confidence: 0.85
provenance_state: inferred
sources: []
---

# 数据智能体平台架构

> **Background**：基于火山引擎 Data Agent 官方文档建立。参考：产品概述页（`/docs/85637/1563626`）、智能问数Agent页（`/docs/85637/1544066`）、营销策略Agent页（`/docs/85637/1587862`）。

数据智能体平台（Data Agent Platform）是一类企业级 AI 应用，通过自然语言界面连接企业数据资产，自动完成数据查询、分析、可视化全流程。与传统 BI 的最大区别是**端到端自动化**——用户提需求，平台直接输出结论，而非等待数据分析师编写 SQL。

## 一、产品形态与能力谱系

### 1.1 核心能力层级

| 层级 | 能力 | 代表产品 |
|------|------|---------|
| **L1 查询** | 自然语言转 SQL，单次查询 | AWS QuickSight Q, Databricks SQL |
| **L2 分析** | 多轮对话 + 归因 + 趋势检测 | 火山引擎智能问数 Agent |
| **L3 研究** | 自动分析框架 + 报告生成 | 火山引擎深度研究 Agent |
| **L4 自动化** | 目标→策略→执行→反馈闭环 | 智能营销策略 Agent |
| **L5 自主** | 跨系统编排 + 自主决策 | 用户研究 AI Agent |

### 1.2 与传统 BI 的本质差异

```
传统 BI:    需求 → 数据分析师 → SQL → 可视化 → 结论
Data Agent: 需求 → LLM(NL2SQL) → 执行 → 结论 (分钟级 vs 天级)
```

关键区别：
- **零 SQL 门槛**：任何业务人员可直接提问
- **多数据源联合**：自动 Join 跨表跨库数据
- **上下文记忆**：多轮对话保持分析上下文
- **自动化归因**：自动 drill-down 到关键因子

---

## 二、架构分层

```
┌─────────────────────────────────────────────────────────┐
│                      SaaS 层                             │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────┐ │
│  │ 智能问数     │  │ 深度研究      │  │ 营销/用户研究   │ │
│  │ Agent       │  │ Agent         │  │ Agent           │ │
│  └──────┬──────┘  └──────┬───────┘  └───────┬────────┘ │
│         └────────────────┼──────────────────┘          │
│                    Multi-Agent Orchestrator             │
│         ┌────────────────┼──────────────────┐          │
│         │                │                  │          │
│  ┌──────▼──────┐  ┌───────▼───────┐  ┌──────▼──────┐  │
│  │ 状态机编排   │  │ 跨Agent共享   │  │ 并行任务调度 │  │
│  │ (LangGraph) │  │ Context Bus   │  │              │  │
│  └─────────────┘  └───────────────┘  └──────────────┘  │
│                          │                              │
│  ┌───────────────────────▼────────────────────────┐   │
│  │              LLM Gateway                         │   │
│  │  路由分发 │ 负载均衡 │ 限流 │ 模型博弈 │ 熔断     │   │
│  └───────────────────────┬─────────────────────────┘   │
└──────────────────────────┼────────────────────────────┘
                           │
┌──────────────────────────┼────────────────────────────┐
│                      工具层                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ SQL执行器 │ │ Python   │ │ API调用   │ │ 文件读写  │  │
│  │          │ │ REPL      │ │          │ │          │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
│                           │                             │
│  ┌───────────────────────┼─────────────────────┐      │
│  │  Schema元数据 RAG     │  分析框架知识库       │      │
│  │  (表结构/关系/样例数据) │  (归因方法/统计检验)   │      │
│  └───────────────────────┼─────────────────────┘      │
└──────────────────────────┼────────────────────────────┘
                           │
┌──────────────────────────┼────────────────────────────┐
│                      数据层                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ClickHouse│ │PostgreSQL│ │  DataLake │ │ 企业API   │  │
│  │ (OLAP)   │ │ (OLTP)   │ │ (Parquet) │ │ (REST)   │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
└──────────────────────────────────────────────────────┘
```

---

## 三、核心模块详解

### 3.1 NL2SQL（自然语言转 SQL）

**技术方案选型**：

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|---------|
| **vSQL / SQLCoder** | 开源、可微调、延迟低 | 复杂 Join 准确率 <80% | 内部可控场景 |
| **GPT-4 + Few-shot** | 无需训练、覆盖广 | 成本高、延迟高 | 通用 SaaS |
| **CoT Prompt** | 无需微调、可解释 | Token 消耗大 | 复杂分析 |
| **混合方案** | 准确率高、可控 | 工程复杂度高 | 企业级产品 |

**核心挑战**：
1. **Schema Linking**：正确识别用户 query 中的表名/列名（同名歧义问题是最大难点）
2. **复杂 Join**：跨 3+ 表的 Join 容易漏掉中间表
3. **聚合嵌套**：子查询 + 窗口函数组合出错率高
4. **方言差异**：ClickHouse / PostgreSQL / MySQL 语法差异需要独立适配

**最佳实践**：
- Schema RAG：每次查询前先从元数据库检索相关表结构，作为上下文注入 prompt
- Self-Correction：让 LLM 先写 SQL，再写一条"验证 SQL"，如果结果不符合预期则自动修正
- 多轮纠错：用户可以追问"为什么是这个结论"，Agent 自动补充验证 SQL

**准确率基线**：
- Simple queries (单表, 无 Join): 90%+
- Medium queries (2表 Join, 简单聚合): 75-85%
- Complex queries (3+ 表, 嵌套子查询): 60-70%

### 3.2 深度研究 Agent（自动分析框架）

**核心流程**：
```
用户问题 → 分析框架生成 → SQL执行 → Python分析 → 结论 → 报告生成
           ↓
      LLM 生成假设验证链路
      e.g. "GMV下降原因"
        → 按地区分解
        → 按品类分解
        → 按时间序列分解
        → 显著性检验
```

**SQL + Python 混合执行**：
- 复杂归因和异常检测用 Python（pandas/scipy）
- 大数据量聚合用 SQL（ClickHouse/PostgreSQL）
- LLM 作为"分析器调度器"，决定哪部分用 SQL 哪部分用 Python

**报告生成**：
- 模板化 Markdown 输出
- 自动插入图表（Matplotlib/Plotly 生成 base64）
- 支持导出 PDF/PPT

### 3.3 LLM Gateway（路由层）

**核心功能**：
- **路由**：根据任务类型选择最优模型（简单查询用便宜模型，复杂分析用最强模型）
- **负载均衡**：多 API Key 轮询，避免单点限流
- **限流**：按组织/用户级别限制 QPS 和 Token
- **熔断**：某模型响应超时/错误率超标时自动切换备选
- **成本追踪**：按用户/部门/任务类型统计 Token 消耗

**实现方案**：
- **LiteLLM**：开源，支持 100+ 模型统一接口，Proxy 模式
- **自研 Proxy**：公司内部定制需求（审计、合规）时自建

### 3.4 Multi-Agent 编排层

**编排模式**：

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **层级式** | 一个主 Agent 分解任务给子 Agent | 复杂分析任务 |
| **对等式** | 多 Agent 并行，各自负责一个维度 | 归因分析 |
| **状态机** | LangGraph DAG 定义执行流程 | 营销策略闭环 |

**上下文共享**：
- Agent 之间通过 `Context Bus` 共享中间结果（不是共享完整上下文窗口）
- 主 Agent 的 system prompt 定义各子 Agent 的职责边界

**关键问题**：
- 长流程断点恢复（Agent 执行中断后从哪个状态恢复）
- Agent 间状态一致性问题（分布式执行时的数据一致性）
- 超时熔断（单个 Agent 执行超时不影响整体流程）

### 3.5 工具层

**SQL 执行器**：
- 语法验证 + 权限校验（用户只能查询有权限的表）
- 慢查询熔断（单次查询超过 N 秒自动取消）
- 结果集缓存（相同 SQL 的结果缓存 N 分钟）

**Python REPL**：
- 沙箱安全执行（禁止 os/system 网络等危险操作）
- 超时控制（单次执行不超过 N 秒）
- 预置分析库（pandas, numpy, scipy, matplotlib）

---

## 四、技术栈建议

| 层级 | 推荐选型 | 备选 |
|------|---------|------|
| 前端 | Next.js + Tailwind + Apache ECharts | React + D3.js |
| 后端 | Python FastAPI + LangGraph | Node.js + Temporal |
| LLM 路由 | LiteLLM | 自研 Proxy |
| SQL 引擎 | ClickHouse | PostgreSQL, DuckDB |
| 向量库 | Qdrant | Chroma |
| 状态/缓存 | Redis + PostgreSQL | etcd |
| 部署 | Kubernetes | Railway, Fly.io |

---

## 五、关键难度与解决方案

### 5.1 NL2SQL 准确率（商业可用门槛 >85%）

**核心解法**：
1. Schema RAG 优化：每次查询注入相关表结构（表名、列名、样例数据、关系图）
2. 微调：基于业务 SQL 历史数据微调 SQLCoder，垂直领域准确率提升 10-15%
3. Self-Correction：双 SQL 生成 + 结果验证，自动纠错机制
4. 置信度兜底：低于置信度阈值时，输出备选 SQL 让用户确认

### 5.2 Code Interpreter 稳定性

**沙箱安全**：
- 使用 Docker 容器隔离执行环境
- 禁止命令：os, subprocess, system, eval, exec
- 内存和执行时间硬限制

**超时熔断**：
- 单次 Python 执行超时（默认 30 秒）
- 累计执行时间超限终止整个分析任务
- 异常结果检测（inf, nan, 空结果集）自动重试

### 5.3 企业级 SLA

| 指标 | 目标 | 实现方案 |
|------|------|---------|
| 慢查询取消 | <10 秒自动熔断 | 超时控制 + 异步队列 |
| 并发上限 | 单租户 50 QPS | Redis 令牌桶限流 |
| 数据权限 | 严格隔离 | 每次查询前注入权限过滤器 |
| 审计日志 | 全量记录 | 异步写 ClickHouse |

---

## 六、参考产品分析

### 6.1 火山引擎 Data Agent

**产品结构**：
```
Data Agent
├── 智能分析 Agent
│   ├── 智能问数 Agent（NL2SQL，多数据集/归因/多轮）
│   └── 深度研究 Agent（自动分析框架，SQL+Python，报告）
├── 智能营销 Agent
│   ├── 智能会话助手（对话式营销中枢）
│   └── 智能营销策略 Agent（目标→策略→任务→优化闭环）
└── 用户研究 Agent
    ├── AI 行为研究 Agent（用户行为特征挖掘）
    └── AI 虚拟调研（数字人+问卷生成）
```

**定位**：企业级 SaaS，非框架。底层依赖 ByteHouse（OLAP）。

### 6.2 AWS QuickSight Q

- NL2SQL 能力，覆盖 QuickSight 数据集
- 限制：只能查询已关联的数据集，无法跨数据集 Join
- 优势：与 AWS 生态深度集成（IAM, Lake Formation）

### 6.3 Databricks SQL AI

- 基于 Lakehouse 架构，支持自然语言查询
- 优势：Unity Catalog 元数据天然作为 Schema RAG 来源
- 劣势：需要 Databricks 环境，非通用 SaaS

### 6.4 Strands Agents (Amazon Bedrock)

- 基于 Bedrock 的数据分析 Agent
- 支持 Python code interpreter
- 集成 AWS 数据服务（Athena, Redshift, S3）

---

## 七、Build Roadmap

### Phase 1: 核心引擎（4–6 周）

**目标**：NL2SQL 单模块可用

- [ ] 选择 NL2SQL 方案（GPT-4 + Few-shot 或 SQLCoder 微调）
- [ ] Schema RAG 实现（表结构元数据注入）
- [ ] 基础 SQL 执行层（权限校验 + 超时熔断）
- [ ] Web 界面（输入框 + 表格结果展示）
- [ ] LLM Gateway 基础版（路由 + 限流）

**交付物**：单数据集自然语言查询，Web 界面返回 SQL + 结果表

### Phase 2: 智能分析 Agent（6–8 周）

**目标**：追齐 L2 + L3 能力

- [ ] 多数据集联合查询（schema linking 自动 Join）
- [ ] 归因分析（breakdown / drill-down）
- [ ] 多轮对话上下文管理
- [ ] 深度研究 Agent（自动分析框架 + 报告生成）
- [ ] Python code interpreter（沙箱安全）

**交付物**：完整 L2/L3 分析能力，支持导出报告

### Phase 3: Multi-Agent 编排层（4–6 周）

**目标**：支持复杂工作流

- [ ] Agent Registry（注册/发现/版本管理）
- [ ] LangGraph 状态机编排（DAG 定义）
- [ ] 跨 Agent Context Bus（中间结果共享）
- [ ] 断点恢复（长流程容错）
- [ ] 可视化 DAG Builder

**交付物**：可视化工作流编辑器 + 自动执行引擎

### Phase 4: 智能营销 Agent（6–8 周）

**目标**：L4 自动化闭环

- [ ] 对话式营销助手（RAG + 话术推荐）
- [ ] 营销策略 Agent（KPI 树 → 任务拆解 → 反馈优化）
- [ ] 用户画像 CDP 集成
- [ ] A/B 测试统计显著性检验

**交付物**：营销全链路自动化

### Phase 5: 用户研究 Agent（4–6 周）

**目标**：L5 自主研究

- [ ] AI 行为研究（行为序列分析 + 聚类）
- [ ] AI 虚拟调研（问卷生成 + 数字人会话）

**交付物**：端到端用户研究自动化

---

## 相关实体

- [[entities/aws-bedrock-ops-alert]]
- [[entities/agent-harness-architecture-deep-dive-aksahy]]
- [[entities/netflix-druid-interval-aware-caching]]
- [[entities/agent-production-harness-engineering]]

## 八、相关概念

- [[concepts/multi-agent-systems]] — Multi-Agent 系统基础
- [[concepts/agent-orchestration-patterns]] — Agent 编排模式
- [[concepts/agentic-workflow-patterns]] — Agentic Workflow 设计模式
- [[entities/aws-quicksight-dataset-qa-natural-language]] — AWS QuickSight Q 案例
- [[entities/strands-agents-sdk-build-analytics-layer-vqr-amazon-bedrock-practice]] — Strands on Bedrock 案例
- [[queries/ai-agent-platforms-selection-criteria]] — Agent 平台选型维度

## 新增关联实体
- [[entities/企业级多-agent-规模化落地怎么做群虾智能-ai-沙龙-ppt-限时领取]]

## 关联实体

**上游依赖**:
- [[entities/aws-bedrock-ops-alert]] — 提供基础理论/方法
- [[entities/agent-harness-architecture-deep-dive-aksahy]] — 提供基础理论/方法

**下游应用**:
- [[entities/netflix-druid-interval-aware-caching]] — 具体应用场景
- [[entities/agent-production-harness-engineering]] — 具体应用场景

**平行协作**:
- [[entities/aws-quicksight-dataset-qa-natural-language]] — 替代/补充方案
- [[entities/strands-agents-sdk-build-analytics-layer-vqr-amazon-bedrock-practice]] — 替代/补充方案
- [[entities/企业级多-agent-规模化落地怎么做群虾智能-ai-沙龙-ppt-限时领取]] — 替代/补充方案

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
