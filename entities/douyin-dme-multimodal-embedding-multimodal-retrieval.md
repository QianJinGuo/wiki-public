---
title: "抖音 DME — Douyin Multimodal Embedding 多模态表征模型"
created: 2026-08-19
updated: 2026-09-07
type: entity
tags: [douyin, multimodal, embedding, retrieval, representation-learning, mme-b, vector-search, contrastive-learning, generation, ai-search]
sources: [raw/articles/douyin-dme-multimodal-embedding-multimodal-retrieval]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 抖音 DME — Douyin Multimodal Embedding 多模态表征模型

## 概述

抖音搜索多模态团队联合中国人民大学高瓴人工智能学院发布多模态表征模型 **DME（Douyin Multimodal Embedding）**，采用「对比 + 生成」混合范式。在 MMEB-v2（78 个数据集，覆盖图像、视频、视觉文档三大域）上 2B/9B 两个参数量级均达对应规模 SOTA（74.8 / 78.4），视频与视觉文档检索优势突出。^[raw/articles/douyin-dme-multimodal-embedding-multimodal-retrieval.md]

DME 已部署进抖音线上系统：离线评测集整体相对提升 2.92%，线上 A/B 验证核心业务指标 0.1% LT（Lifetime）收益，应用于生成式搜索、视觉搜索、AI 搜索等场景。^[raw/articles/douyin-dme-multimodal-embedding-multimodal-retrieval.md]

## 核心命题：效率与细粒度不必二选一

DME 想验证的核心是检索表征的「效率」与「细粒度」不必二选一——将「该看哪里」（细粒度定位）与「必须记住什么」（高层语义记忆）两大任务前置至训练阶段，保障在线推理高效性。该能力适配百亿级工业检索场景，还可支撑 AI 搜索、Agent 等需依托检索结果做深度推理的新兴场景——向量需承载非对称语义，而非仅作排序相似度信号。^[raw/articles/douyin-dme-multimodal-embedding-multimodal-retrieval.md]

## 技术架构

- 对比学习 + 生成任务联合训练，平衡多模态表征的判别力与生成力^[raw/articles/douyin-dme-multimodal-embedding-multimodal-retrieval.md]
- MMEB-v2 三域（图像/视频/视觉文档）全量评估体系^[raw/articles/douyin-dme-multimodal-embedding-multimodal-retrieval.md]
- 论文：arxiv 2608.02148^[raw/articles/douyin-dme-multimodal-embedding-multimodal-retrieval.md]

## 意义

DME 代表多模态表征从「纯检索排序信号」向「承载非对称语义、支撑 Agent 深度推理」的范式演进，与 [[entities/智源悟界robobrain-orca多模态表征世界模型|智源悟界 Orca]] 等属于同一多模态表征前沿家族。

→ [[raw/articles/douyin-dme-multimodal-embedding-multimodal-retrieval|原文存档]]
