---
title: "Guardoc Health 医疗文档AI处理 — Amazon Nova 多模态 RAG 管线"
created: 2026-07-28
updated: 2026-09-07
type: entity
tags: [ai, rag, amazon-nova, amazon-bedrock, document-processing, medical, multimodal, healthcare]
sources: [raw/articles/how-guardoc-transforms-medical-document-processing-with-amazon-nova-models]
confidence: 0.65
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Guardoc Health 医疗文档AI处理 — Amazon Nova 多模态 RAG 管线

> **Background**: Guardoc Health 使用 Amazon Nova 系列模型在 Amazon Bedrock 上构建多阶段医疗文档处理管线，涵盖 RAG 医疗条件分类、混合 OCR 药物提取等功能。本文是 AWS ML Blog 发布的官方案例研究。

## 管线架构

Guardoc 的核心管线是**成本分层多阶段流水线**，每阶段使用最适合该任务的组件：

1. **Amazon Textract**（OCR 层）：从 PDF/扫描件提取文本、结构元数据和布局信息——低成本高通量
2. **文档分块**：沿临床语义边界切分（药物列表、诊断节、医生笔记），非任意字符分块
3. **Amazon Titan Text Embeddings V2**（嵌入层）：每个块向量化后存入 **Amazon DynamoDB**（按患者分区，避免跨患者检索）
4. **自定义预过滤器**：按文档类型、时效性、患者上下文信号缩小候选集——降低下游检索成本
5. **k-NN 搜索**：内存内检索最相关的块，返回页面引用（非完整内容）
6. **Amazon Nova 2 Lite**（粗筛）：轻量文本模型快速排除明显不匹配的页面
7. **Amazon Nova Pro**（多模态推理）：接收 PDF 原始字节，推理布局、笔迹、签名、印章等视觉上下文，输出最终分类

每个分类结果都追溯到原始 PDF 的特定源页面——这在临床场景中是不可妥协的设计约束。^[raw/articles/how-guardoc-transforms-medical-document-processing-with-amazon-nova-models.md]

## 关键技术挑战与方案

### 医疗条件 RAG 分类

使用 **Retrieval Augmented Generation (RAG)** 从患者文档中识别医疗条件。管线先检索患者自身文档中的相关证据，再跨证据推理产生最终分类。与传统的"先 OCR 全文再 LLM 分类"不同，Guardoc 的分级架构让 Textract 负责低价 OCR、Nova Pro 仅在最后一步做昂贵的多模态推理——实现了 **成本-精度权衡**。^[raw/articles/how-guardoc-transforms-medical-document-processing-with-amazon-nova-models.md]

### 混合 OCR 药物提取

药物信息（药名、剂量、用法、频率）从多种格式中提取：
- **结构化表格** → Amazon Textract 处理
- **非结构化医生笔记 / 手写增改 / 传真扫描** → Amazon Nova Pro 补充推理

这种混合架构各取所长：Textract 处理高容量结构化提取，Nova 系列模型处理复杂边缘案例（表格跨列、手写增加、非常规格式）。^[raw/articles/how-guardoc-transforms-medical-document-processing-with-amazon-nova-models.md]

## 业务影响

Guardoc 的部署效果量化案例：

| 指标 | 改进 |
|------|------|
| 文档错误率 | 下降 46% |
| 审计罚款 | 减少 70% |
| 单设施年度 ROI | 超 $400K |
| 单季度（200 患者，2 设施） | 847 文档修正，74% 住院转送减少 |

这些结果说明：在真实生产环境中，**AI 文档处理的 ROI 不只是操作效率的提升，更是临床风险与合规成本的直接降低**。^[raw/articles/how-guardoc-transforms-medical-document-processing-with-amazon-nova-models.md]

## 与现有知识的关系

- [[entities/rag-chunk-embedding-rerank-pipeline|RAG 分块/嵌入/重排管线]] — Guardoc 的 k-NN + 分级筛序方案是对标准 RAG 的医疗场景适配
- [[entities/rag-for-documents|文档 RAG 处理]] — Guardoc 处理的是医疗文档这一极复杂文档类型
- [[entities/amazon-bedrock|Amazon Bedrock]] — Guardoc 的 AI 基础设施底座
- [[entities/amazon-nova-forge-hyperparameter-tuning-art-science|Amazon Nova Forge]] — Nova 模型系列的调优平台
- [[concepts/rag-retrieval-augmented-generation|RAG（检索增强生成）]] — 分类管线的基础范式
- 医疗 AI 应用 — Guardoc 是 AI 在医疗文档处理中的典型案例
- Agentic RAG 模式 — Guardoc 的端到端多阶段检索架构

→ [[raw/articles/how-guardoc-transforms-medical-document-processing-with-amazon-nova-models|原文存档]]
