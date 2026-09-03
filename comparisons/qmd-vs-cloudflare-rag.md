---
title: "QMD vs Cloudflare RAG：wiki-book 搜索方案对比"
created: 2026-07-02
updated: 2026-07-02
type: comparison
tags: [comparison, qmd, rag, search, architecture, wiki-book]
sources: [concepts/qmd-hybrid-search, concepts/wiki-book-rag-architecture]
---

## 背景

wiki-book（《AI 工程》电子书，63,013 篇文档，200MB 源文件）的 AI Chat 功能需要基于知识库内容回答用户问题。经历了从完全服务端到客户端优先的架构迁移后，当前三路降级 RAG 已上线运行。QMD（Query Markdown Documents）作为一个全本地混合检索引擎，被评估为绕开 Cloudflare Free 计划限制的候选升级方案。

本文站在工程哲学层面比较两套方案的异同。

---

## 方案总览

### 当前 RAG 方案（v1.3.3）

三层降级，目标是在 Cloudflare Free 计划内实现最大 RAG 能力：

```
Tier 1（客户端） → Phase 2（服务端） → [Phase 3（语义）]
```

| 层 | 实现 | 语义深度 | 代价 | 状态 |
|----|------|---------|------|------|
| **Tier 1 客户端** | 关键词词频 + TF-IDF 近邻图 | 词袋级 | 21MB 索引 + 30MB 近邻图 (IndexedDB) | ✅ 三环境线上运行 |
| **Phase 2 Reranker** | CF Workers AI `bge-reranker-base` | 跨句语义理解 | ~200ms/次，Free 间歇 503 | ⚠️ CF Pages 专属 |
| **Phase 3 语义** | `bge-m3` → Vectorize 查询 | 向量语义 | 需 $5/月 Workers Paid | ❌ Free 10ms CPU 锁死 |

客户端优先设计：`ragClient.search()` → 失败降级到 `/rag-query` → 再失败给空结果兜底[^RAG-DESIGN.md]。

### QMD 方案（评估中）

一个自包含的 Markdown 混合搜索引擎，跑在你自己服务器上：

```
Query Expansion → BM25/FTS5 + Embedding → RRF 融合 → LLM Reranker → 排序输出
```

| 组件 | 模型 | 大小 | 用途 |
|------|------|------|------|
| Query Expansion | `qmd-query-expansion-1.7B` (q4) | ~1.1GB | 生成 2 个查询变体，原查询 2x 权重 |
| Embedding | `embeddinggemma-300M` (Q8) | ~300MB | 文档向量化，语义匹配 |
| Reranker | `qwen3-reranker-0.6b` (Q8) | ~640MB | 交叉编码器重排序 |
| 全文检索 | SQLite FTS5 (BM25) | — | 关键词精确匹配 |

模型自动下载到 `~/.cache/qmd/models/`，共约 2GB。暴露 HTTP（`POST /mcp`）和 MCP Server 两种接口[^QMD-GitHub]。

---

## 核心能力对比

### 关键词搜索

| 维度 | 当前方案 | QMD |
|------|---------|-----|
| 算法 | 自写 `tokenize()` + 词频打分（标题权重 3x，正文权重 1x） | BM25 (SQLite FTS5) — 信息检索 50 年沉淀的排名函数 |
| 词项处理 | 去停用词、中日英单字符保留 | FTS5 内置分词器，支持 prefix/unicode61 |
| 排名质量 | 固定权重，无词频饱和度 | BM25 的 k1/b 参数调节 TF 饱和度和文档长度归一化 |
| 客户端部署 | 浏览器内纯 JS 执行 | 服务器端 SQLite 索引，稳定一致 |

> 差距：你的自写方案≈2005 年 Lucene 之前的水平，BM25 是 2026 年的基线标准。不过对于精确名称匹配（"MCP 协议"、"Agent 记忆"），两者差距不大，TF-IDF 词袋级在这类场景下够用。

### 语义搜索

| 维度 | 当前 Phase 3 | QMD |
|------|-------------|-----|
| Embedding 模型 | `@cf/baai/bge-m3` (1024维) | `embeddinggemma-300M` (Q8) |
| 掩码语言支持 | ✅ 多语言 | ✅ 多语言 |
| 运行位置 | CF Workers AI（外部 API） | 本地 node-llama-cpp（本地进程） |
| 可用性 | ❌ Free 计划 10ms CPU 锁死 | ✅ 常驻进程，零故障 |
| 每次查询成本 | Workers AI 调用（Free 额度内） | 自有 CPU/RAM，电费 |

> 当前 RAG 方案最痛的短板：Phase 3 所有基础设施已就绪（Vectorize 索引 37,204 条向量、代码分支、wrangler 绑定），但 Free 计划的 10ms CPU 时间限制使任何 embedding 调用都超限。不是实现问题，是 Cloudflare 定价策略的硬切割——Free→Paid 的 $5/月门槛。
>
> QMD 的语义搜索在你自己服务器上跑，不存在这个问题。代价是每查询多了网络延迟（HP 局域网 ~1ms，Tunnel 转发 ~50ms），但模型加载后的推理延迟 <100ms（300M 参数的 embedding 模型）。

### Reranker 重排序

| 维度 | 当前 Phase 2 | QMD |
|------|-------------|-----|
| 模型 | `@cf/baai/bge-reranker-base` | `qwen3-reranker-0.6b` (Q8) |
| 参数规模 | ~270M | ~600M |
| 运行位置 | CF Workers AI | 本地 node-llama-cpp |
| 稳定性 | 间歇 503（连续 3+ 请求触发） | ✅ 零外部依赖 |
| 排序精度 | 足够（30 选 5） | 更强（更大模型 + 位置感知融合） |

> Phase 2 的 Reranker 本身不差——bge-reranker-base 是一个合格的交叉编码器。问题是 Cloudflare Free 计划的 CPU 时间限制（10ms/请求）使连续查询触发 503。这不是模型问题，是部署环境问题。QMD 将 Reranker 移到本地后，这个 503 风险自然消失。

### Query Expansion（查询扩展）

当前 RAG 方案没有查询扩展——用户怎么问就怎么搜。如果用户说"怎么做 AI 应用"而文档写的是"构建智能体应用"，精确关键词匹配会漏掉。

QMD 用 1.7B 模型生成 2 个查询变体（"构建 AI 应用的方法"、"开发智能体应用的步骤"），原查询保持 2x 权重，再 RRF 融合。这在**用户表达方式与文档措辞不一致时**显著提升召回。

> 这个能力对 wiki-book 特别有价值——你的文档以中文为主夹杂英文术语，用户提问时可能用英文、口语、或不同专有名词。

---

## 近邻图 vs 语义向量

这是两套方案最本质的技术哲学差异。

### 当前方案：近邻图（TF-IDF 余弦）

```
离线计算：
  search_index.json (63K 文档)
  → 分词 → 418K 唯一词 → CSR 稀疏矩阵
  → A @ A.T → 63K × 63K 余弦矩阵
  → 每文档 top-20 近邻 → neighbor_graph.json (30MB, 57,380 节点)

查询时：
  top-10 关键词种子 × 20 近邻 → 融合评分
```

**本质**：TF-IDF 词袋级的共现关系。两篇文档近邻，不是因为它们概念相似，而是因为它们用了相似的词。这是词袋模型的"伪语义"——"Agent 记忆"和"记忆存储管理"可能在同一个 TF-IDF 向量空间里距离很近，不是因为它们都讨论记忆系统，而是因为它们都高频出现"记忆"这个词。

### QMD 方案：语义向量

```
文档预处理：
  embeddinggemma-300M → 每个文档向量化
  → SQLite FTS5 索引（BM25）+ 向量索引

查询时：
  Query Expansion → BM25 检索 + 向量检索
  → RRF 融合 → Reranker 重排序 → 输出
```

**本质**：向量空间中的**真正语义匹配**。"hallucination"的向量和"幻觉"在语义空间中是邻居，因为它们的上下文分布相似，而不是因为它们共享词汇。这是词袋模型永远无法做到的。

### 差异可视化

| 查询 "hallucination" | 近邻图方案 | QMD 语义方案 |
|---------------------|-----------|-------------|
| 关键词匹配 | ❌ "hallucination" 无命中 | ✅ BM25 不一定匹配，但 embedding 匹配 |
| 近邻扩展 | ❌ 种子为空，无法扩展 | ✅ 直接从向量索引召回 |
| 最终结果 | ❌ 漏召回 | ✅ 找到"幻觉"相关文章 |

> RAG-RETROSPECTIVE.md 的测试确认了这一差距：`"hallucination"` 在 Phase 1+2 中**无任何命中**，因为 Phase 2（reranker）只能重排序 Phase 1（关键词）返回的候选集，"无命中→无候选→reranker 无能为力"。
>
> QMD 的双路检索（BM25 + 向量）在这个案例中会正常工作：向量路直接找到英文"hallucination"与中文"幻觉"文章的语义关联。

---

## 架构哲学对比

### 当前 RAG：分层降级，网络依赖

```
浏览器                服务器 (CF)               你的服务器
ragClient.search()    
  ├→ IndexedDB 搜索 ← R2 (search_index.json + neighbor_graph.json)
  └→ /rag-query     ← CF Pages Function → Workers AI (Reranker)
                                          → [Vectorize — Paid]
```

- **优势**：三环境共享，浏览器本地执行，零服务器 CPU
- **代价**：首次加载 51MB 数据到 IndexedDB，近邻图 30MB
- **瓶颈**：Phase 3 被 Free 计划锁死，Reranker 间歇 503
- **复杂度**：多环境部署 + R2 资产同步 + 版本化缓存

### QMD：单点部署，统一语义

```
HP 服务器                         浏览器
qmd mcp --transport http         
  ├→ BM25 FTS5                   
  ├→ Embedding semantic          
  ├→ Reranker cross-encoder      
  └→ HTTP POST /mcp (统一查询)  ← ai-chat.js
```

- **优势**：全检索能力，本地执行，零外部故障点
- **代价**：首次需 ~2s 模型加载，后续 ~100ms/查询
- **瓶颈**：HP 关机或网络断时 QMD 不可用，降级到客户端方案
- **复杂度**：单 Docker 容器，`qmd update` 一步重索引

### 关键差异

| 哲学维度 | 当前方案 | QMD |
|---------|---------|-----|
| 计算位置 | 边缘（CF） + 本地（浏览器） | 中心（你自己的服务器） |
| 故障分布 | 3 个降级层各自独立故障 | 1 个单点，降级到客户端兜底 |
| 语义深度 | 词袋级（Tier 1）→ 语义（Phase 3 — 未实现） | 全链路语义（扩写→embedding→reranker） |
| 维护负担 | 多组件（R2 + Pages Function + Workers AI + Vectorize） | 单进程（qmd + node-llama-cpp） |
| 升级路径 | $5/月 Workers Paid → Phase 3 零代码启用 | 无需升级，全能力恒定 |

---

## 部署评估

### QMD 对 HP 服务器的要求

| 需求项 | 规格 | 说明 |
|--------|------|------|
| 架构 | x86_64 | `node-llama-cpp` 需要 SIMD 支持 |
| 内存 | ~2-4GB | 模型加载 + SQLite 索引 |
| 磁盘 | ~2GB | 模型缓存 + 索引文件 |
| 运行模式 | 常驻 HTTP 服务 | `qmd mcp --transport http` |
| 暴露方式 | Cloudflare Tunnel → `qmd.jinguo.tech` 或局域网直连 | |

### HP 服务器现状

| 资源 | 状态 |
|------|------|
| x86_64 架构 | ✅ |
| Docker 环境 | ✅ 已有 OpenMAIC 运行 |
| 可用内存 | 待确认（OpenMAIC 占用 ~512MB，剩余需查） |
| 可用磁盘 | 待确认 |
| Cloudflare Tunnel | ✅ 已有 `openmaic.jinguo.tech` 通过 tunnel 暴露 |
| Node.js | 待检查（QMD 需要 Node 18+） |

> QMD 当前状态：**评估完成，未部署**。下一步是检查 HP 实际资源后用 `qmd init` 创建索引。

---

## 最终判断

### 两套方案不是零和博弈

最佳架构是**QMD 优先，当前方案降级**：

```
doRagSearch("问题")
  │
  ├─ ① QMD (HP)
  │   BM25 + 语义搜索 + Reranker → top 5 ✅
  │
  └─ ② QMD 不可用 → 客户端降级
       关键词 + 近邻图（v1.3.3 方案）
       → IndexedDB 或 /rag-query 兜底
```

### 什么应该保留

| 组件 | 保留原因 |
|------|---------|
| **rag-client.js** | QMD 不可用时的降级路径 |
| **search_index.json** | 降级搜索需要 |
| **neighbor_graph.json** | 降级语义扩展（QMD 在线时闲置） |
| **nginx /rag-query fallback** | Docker 兜底 |
| **R2 资产同步** | CF Pages 降级路径 |

### 什么可以被 QMD 替代

| 组件 | 替代原因 |
|------|---------|
| **Phase 2 Reranker (Workers AI)** | QMD 本地 Reranker 更稳定、更强 |
| **Phase 3 语义搜索 (Vectorize)** | QMD 语义搜索绕开 Free 限制 |
| **近邻图** | QMD 语义向量是真正的语义匹配，近邻图只是 TF-IDF 词袋的"伪语义" |

### 一句话

> **QMD 不是一个平行方案，而是当前 RAG 架构在语义深度上的自然延伸**——用你自己的服务器，实现你被 Cloudflare Free 计划剥夺的那层能力。它不替代客户端关键词搜索，而是在它前面加一层更强的语义入口，原来的三层降级保留为兜底。

---

## 参考

- **wiki-book/RAG-DESIGN.md** — 当前 RAG 的完整架构文档
- **wiki-book/RAG-RETROSPECTIVE.md** — 从 Phase 3 锁死到客户端迁移的全过程记录
- [QMD GitHub](https://github.com/tobi/qmd) — 项目官网
- [[comparisons/ai-knowledge-tools-comparison|AI 知识管理工具横向对比]] — 同目录对比系列

[^RAG-DESIGN.md]: 详见 **wiki-book/RAG-DESIGN.md**："ragClient.search() ← 客户端优先 → 失败降级 fetch(/rag-query) → 再失败空结果"
[^RAG-RETROSPECTIVE.md]: 详见 **wiki-book/RAG-RETROSPECTIVE.md**："选择方案 B（客户端迁移）的理由：个人项目工程实验场价值 > $60/年"
[^QMD-GitHub]: QMD 文档："An on-device, all-local hybrid search engine for personal knowledge bases, optimized for AI agent workflows."
