---
title: "Multi-Vector (Late Interaction) Embedding Models with Sentence Transformers"
created: 2026-08-19
updated: 2026-09-07
type: entity
tags: [embedding, retrieval, colbert, late-interaction, sentence-transformers, rag, multimodal]
sources: [raw/articles/multi-vector-late-interaction-embedding-models-with-sentence]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Multi-Vector (Late Interaction) Embedding Models with Sentence Transformers

## 核心思想：把「压缩损失」留到打分阶段

普通 dense embedding 把一段文本压成一个定长向量，罕见实体、精确 ID、长段落里的关键从句都要挤进同一个向量里竞争空间；多条件查询（如"绿色沙发+木腿+圆坐垫"）也会被折成一个点。多向量模型（late-interaction / ColBERT 家族）避开这层压缩：同样跑 transformer，但把每个 token embedding 投影到小维度（经典 128）并全部保留，一篇 9-token 文档变成 9×128 矩阵，而非 1×128 向量。^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md]

cross-encoder 交互最早（query 与 doc 一起过模型，最准但 doc 无法离线预计算）；bi-encoder 交互最少（一个点积，可离线索引、查询快）；late-interaction 介于两者之间——文档仍独立编码、可离线建索引，但打分时让每个 query token 与每个 document token 交互，保留 token 级匹配信息。^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md]

## MaxSim 算子

打分用 MaxSim：对每个 query token 取它与任一 document token 的最高相似度，再对 query 求和。token embedding 经 L2 归一化，每个点积都是 [-1,1] 的余弦相似度，总和落在 [-num_query_tokens, num_query_tokens]。它把单向量模型必须平均掉的 token 级匹配信息保留下来，通常换来更强的召回，代价是更大的索引。^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md]

## 多向量模型的形态与 SOTA 场景

多向量模型是**视觉文档检索**的当前 SOTA：文本 query 直接与页面图像匹配、无需中间 OCR 步骤。同一套 `sentence-transformers`（`pip install -U sentence-transformers`）即可加载各类 checkpoint、编码打分、接入检索栈、在页面图像上运行并控制索引成本；也覆盖音频与视频检索、token pooling、推理加速与评估。^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md]

## 互补定位

- 与 [[entities/instemb-instruction-following-embeddings-2026|InstEmB]]、[[entities/douyin-dme-multimodal-embedding-multimodal-retrieval|Douyin DME]] 等单向量/多模态 embedding 不同，本文聚焦 late-interaction 的工程使用（加载、编码、MaxSim 打分、视觉无 OCR 检索、索引降本）。
- 适用：RAG 检索、视觉文档检索、需要 token 级精确匹配的场景；代价是索引体积随 token 数线性增长。

→ [[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence|原文存档]]
