---
title: "Netflix MAPS: Multimodal Asset Personalization at Scale"
created: 2026-08-29
updated: 2026-09-07
type: entity
tags: [multimodal, embedding, cold-start, personalization, recommendation, clp, mediafm, netflix, ml-systems]
sources: [raw/articles/maps-netflix-multimodal-asset-personalization-2026]
confidence: 0.85
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Netflix MAPS: Multimodal Asset Personalization at Scale

> **Background**：本文档基于 Netflix Tech Blog 2026-08-28 发布的技术文章 "MAPS: Netflix's Multimodal Asset Personalization at Scale" 的系统分析建立。文章详细介绍了 Netflix 如何使用 CLIP 和 MediaFM 多模态嵌入解决内容冷启动问题，并将 5 个画布模型合并为 1 个统一模型。

## Overview

Netflix 的 MAPS（Multimodal Asset Personalization at Scale）系统展示了如何用 **CLIP 多模态嵌入** 解决内容冷启动问题，将 5 个独立的画布模型合并为 1 个统一模型，并通过 **MediaFM** 多模态基础模型（视觉+音频+文本）进一步提升视频预览个性化效果。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

### 核心架构模式

**CLIP Embedding 解决冷启动**：用 CLIP 预训练模型为每个素材生成 768 维嵌入向量，与学习到的 ID 嵌入拼接后通过 MLP 层。新素材在创建时即获得嵌入，不需要积累交互历史即可个性化。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

**模型合并（5→1）**：CLIP 嵌入对裁剪、缩放、宽高比具有不变性，使得不同画布的渲染结果映射到相近向量空间，从而可以用单一模型统一所有画布的信号。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

**Reward-based Training Data Mixing**：用长期奖励分数对训练样本加权，按交互类型的长期价值而非数量来混合不同画布的数据。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

### 三版本消融实验

| 版本 | 描述 | 离线 IPS | 在线 A/B |
|------|------|---------|---------|
| V1 | 仅加图像嵌入，保持 5 模型 | 部分画布提升 | 不显著 |
| V2 | 仅合并为 1 模型，无图像嵌入 | 部分画布提升 | 不显著 |
| V3 | 合并 + 图像嵌入 | 短面板 +5.691% | 统计显著 |

关键发现：V1 和 V2 各自单独都不显著，只有 V3（两者结合）产生统计显著提升。效果是复合而非相加。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

### Query-Aware 搜索个性化

利用 CLIP 将文本和图像投影到共享嵌入空间的特性，在搜索时通过查询文本嵌入与素材图像嵌入的余弦相似度实现查询感知的个性化排序。无需额外建模，CLIP 嵌入已存在于素材表示中。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

### MediaFM 多模态基础模型

Netflix 首个自研多模态基础模型，基于 8000 万个镜头训练，融合三个信号：
- **视觉**：SeqCLIP
- **音频**：预训练语音和音频嵌入模型
- **文本**：大规模文本模型编码的字幕

MediaFM > SeqCLIP > ID-only，每增加一个模态都有收益，音频和文本信号在电视端收益最大。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

### Embedding Store 架构

**解耦基础模型更新与下游模型部署**：所有嵌入（CLIP、SeqCLIP、MediaFM）存储在统一的 Embedding Store 中，基础模型版本更新只需重新注册和回填嵌入，无需修改下游模型的训练或服务代码。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

### Linear Probe 代理任务

用线性探测器从嵌入预测素材是否为非个性化策略下的赢家，作为新嵌入的廉价预筛选门控。线性探测准确率、离线 IPS 提升和在线 A/B 结果三者一致，验证了嵌入质量与实际效果的相关性。^[raw/articles/maps-netflix-multimodal-asset-personalization-2026.md]

## 相关实体

- [[entities/democratizing-machine-learning-at-netflix-building-the-model]] — Netflix ML 平台
- [[entities/data-projects-managing-data-assets-at-netflix-scale]] — Netflix 数据资产管理
- [[entities/evaluating-netflix-show-synopses-with-llm-as-a-judge]] — Netflix LLM 评估

→ [[raw/articles/maps-netflix-multimodal-asset-personalization-2026|原文存档]]
