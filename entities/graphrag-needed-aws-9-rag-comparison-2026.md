---
title: "GraphRAG 实测：朴素 RAG 调优可胜复杂图谱方案"
created: 2026-06-29
updated: 2026-06-29
type: entity
tags: [rag, graphrag, agentic-rag, context-engineering, retrieval-generation-gap, benchmark, lost-in-the-middle]
sources:
  - raw/articles/graphrag-needed-aws-9-rag-comparison-2026
confidence: 0.85
provenance_state: extracted
---

# GraphRAG 实测：朴素 RAG 调优可胜复杂图谱方案

AWS 生成式 AI 创新中心与 Cisco 联合研究，在 STaRK-Prime 数据集上统一评测 9 种 RAG 方案，发现简单方案调优后可胜过复杂 GraphRAG/Agentic RAG。^[raw/articles/graphrag-needed-aws-9-rag-comparison-2026.md]

## 9 种方案三大类

| 类别 | 场景 | 核心思路 |
|------|------|----------|
| Regular RAG | 1-2 | 纯文档检索 / 文档+1-hop 关系 |
| GraphRAG | 3-5 | 纯图谱 / 自动建图 / 混合遍历 |
| Modular & Agentic | 6-9 | 固定流水线 / 多工具智能体 / 最少工具智能体 |

## 三个反直觉结果

**1. 文档+1-hop 干翻复杂 GraphRAG**：场景 2 Hit@1=0.6972 > 场景 5（混合 GraphRAG）≈0.6514。原因：按类型分组 vs 冗长三元组导致 lost-in-the-middle。^[raw/articles/graphrag-needed-aws-9-rag-comparison-2026.md]

**2. 纯图谱搜索几乎废了**：场景 3 Hit@1=0.1376。光有结构没有语义不行。自动建图（场景 4）质量不稳定。

**3. 工具最少的智能体最强**：场景 8（仅一个检索工具）Hit@1=0.6881, MRR=0.7549，全场最高。加图谱工具反而变差。^[raw/articles/graphrag-needed-aws-9-rag-comparison-2026.md]

## 上下文工程（省 token 关键）

- **关系分组的图表示**：entity1-(rel1 rel2 rel3)-entity2，token O(n)→O(1)
- **图检索+文档去重**：统一子图，亚线性增长
- **Hybrid ReAct-ReWOO**：批量子问题打包一次检索

场景 5 省 53% token；场景 9 省 24% token + 指标反而提升。^[raw/articles/graphrag-needed-aws-9-rag-comparison-2026.md]

## 最核心洞察：检索-生成鸿沟

检索覆盖率 83.5%，模型实际利用率仅 47.9%——资料捞到了模型却"视而不见"。

三个原因：
1. **位置注意力衰减**：前 10% 命中率 85.5%，30-40% 暴跌到 26.3%，70%+ 归零
2. **模型偏爱标准答案**：21 个正确答案只挑 4 个
3. **问题措辞暗示数量**：单数问法让模型只吐一个答案

**启示**：Hit@k/MRR 等检索指标高估了高级策略的真实收益，应分开评测检索覆盖率和生成利用率。^[raw/articles/graphrag-needed-aws-9-rag-comparison-2026.md]

## 与现有知识库的关联

- [[entities/rag-full-pipeline-taobao|RAG 全链路]]：淘宝 RAG 实践侧重工程落地，本文侧重方案对比和评测方法论
- [[concepts/context-engineering|上下文工程]]：本文的 token 去重和 Hybrid ReAct-ReWOO 是上下文工程的具体实现
- [[entities/dream-dense-retrieval-autoregressive-modeling-challengehub-2026|DREAM 检索器]]：DREAM 改进检索质量，本文揭示检索后还有生成鸿沟
- [[concepts/lost-in-the-middle|Lost in the Middle]]：本文用实证量化了位置衰减对 RAG 生成的影响

→ [[raw/articles/graphrag-needed-aws-9-rag-comparison-2026|原文存档]]
