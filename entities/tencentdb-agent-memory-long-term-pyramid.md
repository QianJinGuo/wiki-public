---

title: "TencentDB Agent Memory：L0-L3 语义金字塔长期记忆"
created: 2026-07-02
updated: 2026-08-29
type: entity
tags: [tencent, agent, memory, long-term, hierarchical, persona, semantic-pyramid]
sources: [raw/articles/tencentdb-agent-memory-hierarchical]
review_value: 8
review_confidence: 8
confidence: 0.8
provenance_state: extracted
related:
  - entities/tencentdb-agent-memory-short-term-compression
  - entities/tencentdb-agent-memory-context-offloading
---

# TencentDB Agent Memory：L0-L3 语义金字塔长期记忆

## 摘要

腾讯开源的 TencentDB Agent Memory 的长期记忆子系统：L0-L3 语义金字塔（Conversation→Atom→Scenario→Persona），实现真正可用的跨会话用户理解。PersonaMem 准确率 48% → **76%**。 ^[raw/articles/tencentdb-agent-memory-hierarchical.md]

## 现有方案的不足

传统 Agent Memory 把历史对话切片丢进向量库靠相似度召回，能跑 Demo 但任务一长就出问题。工具调用日志和搜索结果持续累积，上下文窗口塞满过程垃圾。 ^[raw/articles/tencentdb-agent-memory-hierarchical.md]

## L0-L3 语义金字塔

| 层级 | 名称 | 内容 | 用途 |
|------|------|------|------|
| L3 | **Persona**（用户画像） | 长期偏好、工作方式、习惯 | 理解用户是谁 |
| L2 | **Scenario**（场景块） | 场景级上下文与目标 | 理解用户在干什么 |
| L1 | **Atom**（结构化事实） | 关键事实原子，可检索 | 需要时精确查找 |
| L0 | **Conversation**（原始对话） | 完整原始记录 | 追证与审计 |

**设计理念**：不是把历史平铺成向量碎片，而是建成语义金字塔。平时用 L3 理解用户长期偏好，需要具体事实时回溯到 L1 甚至 L0。比"召回几条最相似历史"更像真正可用的记忆系统。 ^[raw/articles/tencentdb-agent-memory-hierarchical.md]

## 短期记忆的压缩索引

短期记忆采用**压缩索引**而非简单摘要：
- 厚重工具日志卸载到外部文件
- 中间层保留步骤摘要（JSONL）
- 最高层只给 Agent 轻量任务图
- 高层保留结构，底层保留证据，中间靠 ID 打通 ^[raw/articles/tencentdb-agent-memory-hierarchical.md]

**关键创新**：不是简单摘要（省 Token 但也丢证据），而是压缩索引——需要时能追回原始证据。 ^[raw/articles/tencentdb-agent-memory-hierarchical.md]

## Benchmark

| Benchmark | 成功率 | Token 消耗 |
|-----------|--------|-----------|
| WideSearch | 33% → **50%** (+17pp) | 221.31M → **85.64M** (-61%) |
| SWE-bench | 58.4% → **64.2%** (+5.8pp) | 3474.1M → **2375.4M** (-32%) |
| AA-LCR | 44.0% → **47.5%** (+3.5pp) | 112.0M → **77.3M** (-31%) |
| PersonaMem | 48% → **76%** (+28pp) | — |

PersonaMem（人物画像记忆准确率）提升最显著（+28pp），说明语义金字塔在建模用户长期记忆方面特别有效。 ^[raw/articles/tencentdb-agent-memory-hierarchical.md]

## 相关实体

- [[entities/tencentdb-agent-memory-short-term-compression|TencentDB Agent Memory 短期记忆压缩方案]] — 短期记忆压缩的另一种实现角度

## 来源

→ [[raw/articles/tencentdb-agent-memory-hierarchical|原文存档]] ^[raw/articles/tencentdb-agent-memory-hierarchical.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

