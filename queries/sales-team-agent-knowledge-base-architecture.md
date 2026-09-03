---
created: 2026-06-10
uid: sales-team-agent-knowledge-base-architecture
title: "销售团队的 Agent 知识库架构应该如何设计？"
type: query
tags: [agent, knowledge-base, rag, architecture, enterprise, wecom, sales, multi-tenant]
related:
  - "[[agent-harness-architecture]]"
  - "[[agent-harness-engineering-survey-etcvlovg-taxonomy]]"
  - "[[harness-engineering]]"
  - "[[anthropic-12-mcp-production-patterns]]"
  - "[[routa-harness-visualization]]"
updated: 2026-06-10
sources:
  - raw/articles/sales-team-agent-kb-architecture-interview
---

# 销售团队的 Agent 知识库架构应该如何设计？

> 为 200 人销售团队搭建企业级 Agent 知识库问答系统，支持多源文档检索、企微机器人接入、多产品线复用，核心是在 3 天内交付 MVP

## 问题定义

**面试题原文**：假设你需要在 3 天内为一个 200 人的销售团队搭建一个企业级 Agent 知识库问答系统，支持：
1. 产品手册、FAQ、历史工单等多源文档检索
2. 销售在企微机器人里用自然语言提问
3. 能复用到其他产品线

请你设计这个系统架构，并说明核心组件的选择理由。

## 完整架构图

```
企微机器人 (WeChat Work Bot)
           │
           │ 自然语言提问
           ▼
      API Gateway / 路由层
      - Query Understanding (意图分类 + 实体抽取)
      - Tenant Routing (产品线识别)
      - Rate Limit / Auth
           │
     ┌─────┴─────┐
     ▼           ▼
  知识检索管道    生成管线 (Generation)
     │              │
     │  Top-K       │
     ├─────────────►│
     │              │
  Hybrid Search   LLM (Claude/GPT-4o)
  - 向量检索 (Chroma/Qdrant)   + Reranker
  - BM25 全文 (Elasticsearch)  + Source Attribution
     │              │
     │              │ RAG Context
     │              ▼
     │        格式化回答（含引用来源）
     ▼              │
  文档处理管道 (Ingestion)     │
     │                         │
     ▼                         │
PDF/Word/Markdown/Excel ──► Unstructured/MarkItDown
                                │
                                ▼
                          Chunking (512 tokens, 50 overlap)
                                │
                                ▼
                          Embedding (bge-m3) → Vector DB
                          + 原文 → PostgreSQL / ES (全文)
```

## 核心组件选择理由

### 文档处理：`Unstructured` 或 `MarkItDown`

3 天没有时间写 PDF/Word 解析器。Unstructured 支持 8+ 格式自动提取，API 或自托管均可。历史工单走 `pandas` 直接读 Excel/CSV。

### Chunking 策略

- **固定 chunk：512 tokens，50 overlap**（适合 QA 问答，超过 1024 的内容用子查询切分）
- 对 FAQ 用「标题+内容」作为独立 chunk 保留结构
- 对工单按「问题描述→解决方案」配对 chunk

### 向量库选择

| 方案 | 适用场景 | 选型理由 |
|------|---------|---------|
| **Chroma** | 3 天 MVP / 轻量级 | 零基础设施，pip install 即可跑 |
| **Qdrant** | 生产级复用 | 向量索引性能更强，支持多租户 |

**Embedding 模型：** `bge-m3`（中文效果好，支持多语言）或 `text-embedding-3-small`（OpenAI，延迟低）

### 检索策略：Hybrid Search（向量 + BM25）

单一向量检索召回率约 70%，混合检索能拉到 90%+。因为销售提问常有具体产品型号（精确匹配 > 语义相似）。

- 实现：`Jina Search API` 或自建 `Elasticsearch + Chroma`
- 检索后 **Reranker**（BGE-Reranker）重排，Top-5 送 LLM

### LLM 选择

| 场景 | 推荐 | 理由 |
|------|------|------|
| 内部知识库 | Claude 3.7 Sonnet | 中文理解好，长上下文直接读 chunk |
| 降本 | GPT-4o | OpenAI API 稳定 |
| 国产/合规 | DeepSeek-V3 | 自主部署，数据不出境 |

**Prompt 策略：** RAG context 放在 System Prompt，用「产品线+文档来源+时间」标注归属，回答必须带来源索引。

### 企微机器人接入

企微支持**企业机器人**（Webhook 模式），接收用户消息 → 调 API → 返回图文/文本。**纯响应式够用**，不需要主动推送。

```python
# 极简流程
user_msg → API Gateway → Query Understanding
  → Hybrid Search → Reranker → LLM
  → 格式化回答（含引用来源）→ 企微 Bot 回复
```

## 3 天交付计划

| Day | 任务 | 产出 |
|------|------|------|
| **Day 1** | 文档接入（3 个格式）、Chroma 部署、检索 demo | 可搜索的 MVP |
| **Day 2** | LLM 集成、企微 Bot 接入、Reranker | 端到端可用的 Bot |
| **Day 3** | 评测调优（bad case）、多产品线配置化、部署文档 | 上线交付 + 复用手册 |

## 跨产品线复用关键设计

```yaml
# products.yaml — 配置驱动，非代码复制
products:
  - name: 销售知识库
    document_sources: [产品手册/, FAQ/, 历史工单/]
    embedding_model: bge-m3
    prompt_template: 你是一个专业的销售助手...
    wecom_bot_url: https://qyapi.weixin.qq.com/...
  - name: 客服知识库
    document_sources: [客服手册/, 故障库/]
    embedding_model: bge-m3
    prompt_template: 你是一个专业的客服助手...
    wecom_bot_url: https://qyapi.weixin.qq.com/...
```

**核心原则：** 新产品线接入只需加配置，不改代码。

## 核心风险与应对

| 风险 | 缓解方案 |
|------|---------|
| 文档更新滞后 | 增量更新 cron（每天凌晨重新索引） |
| LLM 编造答案（幻觉） | 强制要求回答带「来源」引用，无引用不返回 |
| 企微 Bot 响应超时 | 异步处理 + 草稿回复「正在查询中，请稍候」 |
| 200 人并发 | LLM API 限流 + 本地缓存热门 query 结果 |

## 相关实体


## See Also

- [[agent-harness-architecture]] — Agent 架构通用模式
- [[agent-harness-engineering-survey-etcvlovg-taxonomy]] — ETCLOVG 中的 V 层（Verification & Evaluation）涉及评测体系设计
- [[harness-engineering]] — Harness 工程化实践
- [[anthropic-12-mcp-production-patterns]] — MCP 协议在生产中的模式
- [[queries/agent-memory-system-design|Agent Memory System 设计指南]]
- [[concepts/agent-backend-unification|Agent 与后端统一架构]]
