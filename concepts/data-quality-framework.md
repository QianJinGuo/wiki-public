---
title: "数据质量框架"
created: 2026-07-02
updated: 2026-08-01
type: concept
tags: [data-quality, rag, knowledge-base, framework]
provenance_state: inferred
confidence: 0.7
---

# 数据质量框架

> 本页填补知识库在数据质量维度的盲区。

## 知识库的数据质量维度

一个 AI 知识库的质量不是由内容数量决定，而是由以下维度决定：

1. **准确性**：内容是否与原始来源一致（溯源完整性）
2. **完整性**：关键信息是否有遗漏
3. **一致性**：不同页面间是否存在矛盾
4. **时效性**：信息是否过时
5. **可溯源**：每条声明能否追溯到原始来源
6. **可发现**：内容能否被有效检索

## 质量评估指标

- 溯源覆盖率 = 有引用段落 / 总段落
- 索引完整率 = 已登记页面 / 磁盘页面
- 断链率 = 断裂链接 / 总链接
- 推断比例 = 无源推断段落 / 总段落

## 关联

- [[entities/knowledge-base-construction|知识库构建方法论]]
- [[entities/llm-wiki-knowledge-management|LLM Wiki 知识管理]]
- [[entities/ai-knowledge-base-llm-wiki-practice-alicloud|阿里云 AI 知识库实践]]

## 所属 MOC

- [[moc/rag-knowledge-retrieval|Rag Knowledge Retrieval]]
