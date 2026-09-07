---
title: "Real-Time Personalisation at Scale: How Zepto Understands What You Want, Right Now"
created: 2026-06-25
updated: 2026-09-07
type: entity
tags: ["recommendation-system", "real-time", "personalization", "engineering", "ranking"]
provenance_state: inferred
source: "[[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker]]"
confidence: 0.8
review_value: 8
review_confidence: 8
review_recommendation: strong
review_stars: 4
sources:
  - raw/articles/zepto-real-time-personalisation-dual-sequence-ranker
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Real-Time Personalisation at Scale: How Zepto Understands What You Want, Right Now

Zepto 的实时个性化系统：双序列排序器（Dual Sequence Ranker）在高并发场景下的工程实践，涵盖特征工程、模型服务和实时推理链路。^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


## 核心内容

[![Image 1: Zepto Tech](https://miro.medium.com/v2/da:true/resize:fill:64:64/0*Zdo4al9KE5LuqNxm)](https://medium.com/@tech.culture?source=post_page---byline--d743b13367c8---------------------------------------)^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


9 min read

2 days ago

Press enter or click to view image in full size^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


![Image 2](https://miro.medium.com/v2/resize:fit:700/1*qxtdmR1I8cptGyc-nkzm9Q.jpeg)^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


Zepto started as a quick-commerce platform for groceries. Today it’s where people shop for electronics, home essentials, gifts and festive needs. The catalogue has grown dramatically, but the app is still the same screen. That same real estate now needs to serve a far wider range of intents, in real-time, to millions of users a day.^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


This is what makes in-session personalisation critical. A user browsing at 8 AM on a Tuesday (rushing to add staples before leaving for work) is operating very differently from the same user at 11:30 PM on a Friday, exploring snacks and beverages after a long week. While the historical user profile is identical, the immediate intent has completely shifted. To keep the shopping experience seamless, our ranking systems must instantly adapt to these context switches.^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


In this post, we describe the system we built to do exactly that: a real-time dual sequence ranker that combines what a user is doing right now with what they have done before and serves scores within strict milliseconds latency budgets.^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


## The Problem: One Screen, Many Intents

Personalisation at Zepto is never a one-size-fits-all _“Recommended for You”_ widget. Our app must seamlessly adapt to a wide spectrum of user intents in real-time from habit driven repurchases to deeply contextual cross sells.^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


Press enter or click to view image in full size^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


![Image 3](https://miro.medium.com/v2/resize:fit:700/0*-mfaXffJbzpLGxWB.png)^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


> **_The core issue:_**_User behaviour varies significantly across assets. A model that treats every surface the same, or compresses all interactions into a single static user vector, will overlook these differences — and those mistakes directly translate into lost conversions._

## What Came Before: And Why It Was Not Enough

Before this system, recommendation surfaces at Zepto relied on a combination of Collaborative Filtering (CF), popularity weighted ranking and cohort level models. These approaches had real strengths: they were fast to train, highly interpretable and worked well for head items and well understood user segments.^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


But as our catalogue expanded and user journeys became increasingly complex, three structural limitations became glaringly visible:^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


*   **No session awareness:** Prior systems had no visibility into what a user was doing _in the current visit_. A user who opened the app specifically to restock party supplies got the exact same ranking as a user doing a routine weekly shop.
*   **Time blindness:** CF treats interactions from three months ago the same as interactions from three days ago. For fast moving categories such as snacks, beverages and seasonal items, recency matters.
*   **Static user representation:** Compressing a user’s entire history into a single vector is a heavily lossy operation. It works for stable preferences, but completely misses the contextual signals that drive in-session decisions.

> **_The Goal:_**_Build a model that understands both who you are (long term habits) and what you are doing right now (short term session intent) — producing a deeply personalised score for every candidate Stock Keeping Unit (SKU) at inference time._

## System Overview: Architecting the Dual Sequence ReRanker

To address these historical gaps, we built a custom **Dual Sequence ReRanker** — inspired by Alibaba’s [_Deep Interest Network (DIN)_](https://arxiv.org/abs/1706.06978) and [_Behaviour Sequence Transformer (BST)_](https://arxiv.org/abs/1905.12249). Instead of relying on a static snapshot, the new architecture dynamically balances what a user _usually_ does with what they are doing _right now_.^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


Press enter or click to view image in full size^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


![Image 4](https://miro.medium.com/v2/resize:fit:700/0*vGgi1GzZnGjweRnY.png)^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


> **_The Architectural Edge_**_Separate encoders ensure neither time horizon drowns the other out. The model leans on history for new sessions, but instantly adapts when in-session intent is strong. Coupled with_**_target aware pooling_**_— which rebuilds the user profile per candidate item — and a_**_hybrid loss function_**_, the model captures a highly precise, context aware signal._

In production, this architecture delivered measurable bottom line improvements across our recommendation surfaces: **Conversion Lift, Tail Item Discovery and Lightning Fast Serving (P99 in low single digit ms).**^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


## Building Intuition: The Full Architecture

Press enter or click to view image in full size^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


![Image 5](https://miro.medium.com/v2/resize:fit:700/0*0htSkTuAbbgUJl25.png)^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


Ranker end-to-end architecture: Token enrichment (item + action + temporal), dual encoders, target aware pooling, fusion gate and ranking head.^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


Ranker end-to-end architecture: Token enrichment (item + action + temporal), dual encoders, target aware pooling, fusion gate and ranking head.^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


> **_End-to-End Execution_**_At inference, the model ingests a user’s history alongside their live in-session interactions to score a set of candidate SKUs. It encodes both timelines through separate transformer stacks, computes a candidate specific user representation via target aware pooling, dynamically blends these signals through a learned gate and outputs a final conversion probability — enriched by real-time popularity trends and calendar context._

## 1. Token Enrichment: Richer Than a Product ID

A sequence item isn’t just an ID; it’s a structured token combining three core signals:^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]


*   **Item Embedding (128 d):** A dense SKU vector, initialised from pre-trained embeddings and fine tuned during training.
*   **Action Embedding:** Differentiates interaction types. An _Add-to-Cart (ATC)_ carries heavier 

## 深度分析

### 双序列架构解决"静态用户画像"的根本性缺陷

Zepto 的双序列排序器（Dual Sequence Ranker）直接回应了推荐系统中的一个结构性问题：将用户历史压缩为单一静态向量是极度有损的操作。^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md] 双序列设计（长期历史 + 实时会话）通过独立编码器保持两个时间维度的信号完整性，避免短期意图被长期偏好淹没。这一架构灵感来自阿里巴巴的 DIN 和 BST，但 Zepto 将其应用于快商务场景（10 分钟配送），其中会话意图的时效性远比传统电商更强。

### Target-Aware Pooling 的候选级个性化

传统推荐模型计算一次用户表示后对所有候选商品使用同一向量打分。Zepto 的 target-aware pooling 机制为每个候选 SKU 重新构建用户表示——这意味着用户对不同商品的偏好被动态计算，而非用一个通用向量近似。^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md] 这种"候选级个性化"显著提升了转化率，尤其是在长尾商品发现场景中。

### Token Enrichment 超越商品 ID

Zepto 的序列 token 不仅是商品 ID，而是融合了商品嵌入（128 维）、动作类型嵌入（加入购物车 vs 浏览 vs 搜索）和时间信号的结构化 token。^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md] 这种丰富的 token 表示使得模型能够区分"浏览了某商品"和"将某商品加入购物车"——两者对购买意图的预测力完全不同。

### 从协作过滤到深度排序的演化路径

Zepto 的推荐系统演化路径（CF → 人口加权排序 → 队列级模型 → 双序列排序器）反映了推荐系统的经典演化模式。每个阶段解决了前一阶段的结构性限制：CF 无会话感知，队列级模型无实时性，双序列排序器同时解决了两者。^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md]

### 混合损失函数的训练策略

双序列排序器使用混合损失函数同时优化多个目标（转化率、长尾发现、多样性），这比单一损失函数更能平衡推荐系统的多个商业指标。^[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker.md] 这种多目标优化在快商务场景中尤为重要——不仅要推荐用户可能购买的商品，还要引导用户发现新品类和长尾商品。

## 实践启示

1. **推荐系统应分离长期和短期信号**：使用独立编码器处理用户历史和实时会话，避免短期意图被长期偏好淹没，尤其是在会话时效性强的场景（快商务、内容消费）。
2. **Target-aware pooling 是转化率提升的关键**：为每个候选商品重新计算用户表示，而非使用通用用户向量——这在商品品类丰富、用户意图多样的场景中效果尤为显著。
3. **动作类型是被低估的信号源**：将用户行为（浏览、加入购物车、搜索、购买）编码为独立嵌入，使模型能区分不同行为的预测力差异。
4. **推荐系统演化应遵循渐进式路径**：从简单模型（CF）开始，逐步引入更复杂的架构，每个阶段解决前一阶段的结构性限制，而非直接跳到最复杂的方案。
5. **多目标损失函数平衡商业指标**：单一损失函数（如仅优化 CTR）会导致推荐同质化，混合损失函数可以同时优化转化率、多样性和长尾发现。

→ [[raw/articles/zepto-real-time-personalisation-dual-sequence-ranker|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

