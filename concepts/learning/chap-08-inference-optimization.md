---
title: 第 8 章：推理优化
created: 2026-06-24
updated: 2026-06-26
type: concept
tags: [learning-path, chapter-08, layer-1]
estimated_minutes: 50
prerequisites: [chap-04, chap-05, chap-07]
---

# 第 8 章：推理优化

> 📍 [学习路径](../../moc/learning-path.md) · [第 1 层](../../moc/layer-1-llm-principles.md) · 上一章：[第 7 章](chap-07-scaling-laws.md) · 下一层：[第 2 层](../../moc/layer-2-interaction.md)

## 🍅 番茄钟规划

50min，2 番茄钟：番茄1（KV-Cache+PagedAttention）→ 番茄2（量化+蒸馏+复习题+关卡）

## 📋 前置回顾

- 第 4 章：attention 的 Q/K/V 里，K 和 V 是什么？
- 第 5 章：KV-Cache 显存随什么线性涨？
- 第 7 章：Scaling Law 说模型越大越好，但部署成本怎么变？

## 🔍 预习

第 7 章你学了「模型越大效果越好」。但大模型部署成本爆炸——一个 70B 模型光权重就要 140GB 显存。这章讲工程师怎么把「理论上的大模型」变成「能用的服务」。

## 📖 正文

### 1.1 KV-Cache：推理的核心优化

第 4 章说过，attention 要算 `Q·K`。生成第 N 个 token 时，前面 N-1 个 token 的 K 和 V **不会变**。**KV-Cache** 就是把这些 K/V 缓存起来，避免每生成一个 token 就重算前面所有 token 的 K/V。没有 KV-Cache，生成长文本的成本是 O(n³)；有 KV-Cache，降到 O(n²)。

### 1.2 PagedAttention：vLLM 的核心

但 KV-Cache 有问题：**显存浪费**。传统做法要预分配「最大上下文长度」的连续显存。[[concepts/inference-optimization|推理系统优化]] 介绍 vLLM 的 **PagedAttention**：把 KV-Cache 切成固定大小的「页」，按需分配，多个请求可共享相同前缀的 page。效果：显存利用率从 ~30% 提到 ~90%。

### 1.3 PD 分离：Prefill/Decode 拆开

生成有两个阶段：**Prefill**（处理输入 prompt，算所有 token 的 KV，计算密集）和 **Decode**（逐个生成输出 token，显存带宽密集）。**PD 分离**：把 prefill 和 decode 放不同机器。SGLang、DeepSeek 都用这个思路。

### 1.4 量化：权重压缩

[[concepts/model-distillation-compression|模型蒸馏与压缩]] 介绍量化：

| 精度 | 显存（70B） | 精度损失 |
|---|---|---|
| FP16 | 140GB | 基准 |
| INT8 | 70GB | 极小 |
| INT4 | 35GB | 可接受 |
| 1-bit | ~9GB | 显著但可用 |

**QAT**（量化感知训练）：训练时就模拟低精度。**PTQ**（后训练量化）：训练后直接压。

### 1.5 蒸馏：小模型学大模型

**知识蒸馏**让小模型（学生）学大模型（教师）的输出分布。学生不仅学正确答案，还学教师对其他选项的概率——这携带了类别间相似性的「暗知识」。

## 🎯 重点回顾

1. **KV-Cache** 缓存历史 K/V，把生成复杂度从 O(n³) 降到 O(n²)
2. **PagedAttention** 分页管理显存，利用率 30%→90%
3. **PD 分离** 把 prefill 和 decode 拆到不同机器
4. **量化** 把 FP16 压成 INT8/4/1-bit
5. **蒸馏** 让小模型学大模型的暗知识

## 🧠 费曼练习

> 向 12 岁孩子解释「为什么 AI 回答问题需要等一会儿」。

提示：AI 像厨师，prefill 是看菜谱，decode 是一道道做菜。

## ✅ 复习题

1. **[选择题]** KV-Cache 缓存的是什么？ A. 模型权重 B. 输入 prompt C. 历史 token 的 Key 和 Value D. 输出结果
2. **[问答题]** PagedAttention 相比传统 KV-Cache 管理有什么改进？
3. **[场景题]** 单张 24GB 显卡上跑 70B 模型。用哪些技术组合？
4. **[费曼题]** 用 3 句话向新手解释「量化为什么能省显存但会损精度」。
5. **[关联题]** 回顾第 5 章上下文窗口。KV-Cache 显存和上下文长度是什么关系？

??? answer "参考答案"
    1. **C**
    2. 传统预留连续大块显存，利用率低（~30%）。PagedAttention 把 KV 切成页，按需分配，利用率提到 ~90%。
    3. 组合：① INT4 量化；② 张量并行拆多卡；③ KV-Cache 用 PagedAttention；④ PD 分离。或用蒸馏出的小模型。
    4. 显存存的是数值精度。FP16 用 16 位，INT4 只用 4 位，省 4 倍空间。但 4 位能表示的值有限，丢失精度。
    5. KV-Cache 显存随上下文长度线性增长（每个 token 都要存 K/V）。

## 📚 拓展阅读

- [[concepts/inference-optimization|推理系统优化]] — 本章主源
- [[concepts/model-distillation-compression|模型蒸馏与压缩]]
- [[entities/vllm-v0-to-v1-correctness-before-corrections|vLLM V0→V1]]
- [[entities/sglang|SGLang]]
- [[entities/bonsai-image-4b-1-bit-ternary|1-bit 三值量化]]
- [[raw/articles/vllm-v0-to-v1-correctness-before-corrections|vLLM 论文]]
- [[raw/articles/glm5-scaling-pain-inference|GLM5 推理痛点]]

## 🚪 第 1 层关卡

恭喜完成第 1 层！回答 [第 1 层 MOC](../../moc/layer-1-llm-principles.md) 的 5 道关卡题。

## ⏭️ 下一层预告

第 2 层讲 **交互实践**——Prompt 工程、上下文工程、RAG。
