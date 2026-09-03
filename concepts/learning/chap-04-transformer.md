---
title: 第 4 章：Transformer 架构
created: 2026-06-24
updated: 2026-08-29
type: concept
tags: [learning-path, chapter-04, layer-1]
estimated_minutes: 50
prerequisites: [chap-01, chap-02, chap-03]
---

# 第 4 章：Transformer 架构

> 📍 [学习路径](../../moc/learning-path.md) · [第 1 层](../../moc/layer-1-llm-principles.md) · 上一章：[第 3 章](chap-03-knowledge-management.md) · 下一章：[第 5 章](chap-05-token-context.md)

## 🍅 番茄钟规划

50min，2 番茄钟：番茄1（Embedding+Attention）→ 番茄2（Multi-Head+工程意义+复习题）

## 📋 前置回顾

- 第 2 章：Software 2.0 的「程序」是？
- 第 3 章：LLM Wiki 中层由谁维护？

## 🔍 预习

你天天用 ChatGPT/Claude，但知道它们底层是什么架构吗？答案就一个词：**Transformer**。2017 年 Google 的《Attention Is All You Need》提出，从此改变了 NLP。

## 📖 正文

### 1.1 一句话理解 Transformer

> Transformer 是一种让序列中**任意两个位置直接建立依赖**的神经网络架构。

对比 RNN：RNN 要逐 token 顺序处理，前面的信息靠「隐藏状态」往后传，远了就忘了。Transformer 用 **attention** 让 token 之间直接「看」彼此，没有距离衰减。

### 1.2 Embedding：Token 变成向量

LLM 不认识文字，只认识数字。流程：`文字 → 分词 → token → Embedding → 向量`。[[concepts/transformer-architecture|Transformer Architecture]]：Embedding 矩阵把每个 token 映射为稠密向量（通常 768-4096 维）。

### 1.3 Self-Attention：核心创新

[[concepts/attention-mechanism|Attention Mechanism]]：**让每个 token 决定「关注」序列中哪些其他 token**。每个 token 被投影成三个角色：Query（查询）/ Key（键）/ Value（值）。计算：`分数 = Query · Key` → `权重 = softmax(分数)` → `输出 = 权重 × Value`。

**直觉**：读「The cat sat on the mat because it was tired」时，`it` 的 Query 去匹配 `cat` 的 Key，把 `cat` 的 Value 加进来。

### 1.4 Multi-Head Attention：多视角

一个 attention 只学一种「关注模式」。Multi-Head 用多个 attention 头并行，每个学不同模式：语法/指代/长程依赖。最后把所有头的输出拼起来。

### 1.5 工程意义

- **为什么有上下文窗口限制**？→ attention 是 O(n²)（第 5 章）
- **为什么需要 KV-Cache**？→ 缓存 Key/Value（第 8 章）
- **为什么 Prompt 工程有效**？→ Context 里的 token 互相 attention（第 2 层）
- **为什么模型越大越强**？→ 更多参数（第 7 章 Scaling Law）

## 🎯 重点回顾

1. **Transformer** = 让序列任意两位置直接建立依赖
2. **Embedding** 把 token 映射为稠密向量
3. **Self-Attention** 用 Q/K/V 让 token 互相「关注」
4. **Multi-Head** 是多视角并行
5. **理解它**是理解上下文窗口、KV-Cache、Prompt 工程的前提

## 🧠 费曼练习

> 向 12 岁孩子解释「self-attention 在做什么」。不能用 Q/K/V。

提示：一群人在房间里，每个人问「谁有我需要的信息」，每个人也报「我能提供什么信息」，然后大家根据匹配度互相交换信息。

## ✅ 复习题

1. **[选择题]** Self-Attention 的核心是？ A. RNN 顺序传递 B. CNN 局部感知 C. 任意两 token 直接建立依赖 D. 全连接层映射
2. **[问答题]** 用自己的话解释 Q/K/V 三个角色的作用。
3. **[场景题]** 给句子「The cat sat on the mat because it was tired」，`it` 的 attention 应该主要聚合哪个 token 的 Value？为什么？
4. **[费曼题]** 用 3 句话向新手解释「为什么 Multi-Head 比单头好」。
5. **[关联题]** 本章说 attention 是 O(n²) 复杂度。这对「上下文窗口」意味着什么？

??? answer "参考答案"
    1. **C**
    2. Query 是「我在找什么」，Key 是「我能提供什么」，Value 是「我实际信息」。Q·K 算相关性，softmax 归一化，乘 V 加权聚合。
    3. `cat`。因为 `it` 指代 `cat`（猫累了）。
    4. 单头只学一种关注模式；多头并行学多种（语法、指代、长程等），最后拼接。
    5. O(n²) 意味着序列长度翻倍，计算量和显存翻 4 倍。这就是为什么上下文窗口不能无限大。

## 📚 拓展阅读

- [[concepts/transformer-architecture|Transformer Architecture]] — 本章主源
- [[concepts/attention-mechanism|Attention Mechanism]] — Q/K/V 数学
- Reasoning Models
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes]]
- [[raw/articles/kimi-attention-residuals-preNorm-dilution-block-attnres|Kimi AttnRes 论文]]

## ⏭️ 下一章预告

第 5 章讲 **Token 与上下文窗口**——Transformer 吃的是什么、能吃多少。
