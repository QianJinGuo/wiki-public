---
title: "Lost in the Middle 长上下文注意力衰减"
created: 2026-06-30
updated: 2026-08-01
type: concept
tags: [concept, llm, attention, context-window, rag, long-context]
sources: []
---

## 定义

Lost in the Middle 是 LLM 在处理长上下文时的一种系统性偏差：模型对上下文开头和结尾的信息利用率显著高于中间部分。当关键信息被放置在长 prompt 的中间位置时，模型回答准确率会大幅下降。这一现象由 Liu et al.（2023）系统性地实验证实，对 RAG、长文档摘要、多轮对话等依赖长上下文的应用场景有直接工程影响。

## 核心范式

- **U 形注意力曲线**：模型对 prompt 首尾的 token 注意力权重高，中间区域注意力衰减
- **位置敏感**：同一段关键信息放在 prompt 开头 vs 中间 vs 结尾，回答准确率可差 20-30%
- **任务依赖**：检索类任务（"找到相关信息"）受影响最大；生成类任务（"总结全文"）受影响较小
- **模型差异**：不同模型的衰减程度不同，GPT-4、Claude 3.5 等新模型有所改善但未完全消除
- **上下文长度正相关**：上下文越长，中间区域的衰减越严重

## 背景与提出

Stanford 和 UC Berkeley 的 Nelson F. Liu 等人在 2023 年论文 "Lost in the Middle: How Language Models Use Long Contexts" 中首次系统性地量化了这一现象。实验设计简洁有力：在不同位置插入正确答案，测量模型的准确率变化。结果发现几乎所有测试模型都呈现 U 形曲线——对首尾信息敏感，对中间信息"视而不见"。

这一发现对 RAG 架构有直接冲击：如果检索返回 10 个片段拼接到 prompt 中，排在第 3-7 位的片段可能被模型"忽略"。这促使工程实践中采用 reranking（将最相关的片段放在首尾）、减少检索数量、分块摘要等缓解策略。

## 局限与反对声音

- **正在改善**：2025-2026 年的长上下文模型（Gemini 1.5、Claude 3.5 Sonnet）在 128K 窗口下 U 形曲线已明显减弱
- **任务类型影响**：并非所有任务都受影响，简单的信息提取任务对位置不敏感
- **评估方法争议**：部分研究者认为原始实验的 needle-in-a-haystack 设计过于极端，不代表真实使用场景
- **注意力机制改进**：YaRN、RoPE 缩放等技术在改善长上下文处理能力

## 实践启示

1. **RAG 检索结果排序**：将最相关的片段放在 prompt 首尾，而不是按相关度分数线性排列
2. **控制上下文长度**：宁可少检索几个高相关片段，也不要塞满整个上下文窗口
3. **分段处理**：超长文档先分段摘要，再将摘要拼接——避免直接将全文塞入 prompt
4. **位置感知评估**：在评估 RAG 系统时，测试不同位置的信息召回率，不要只看整体准确率
5. **Reranking 策略**：采用 Lost-in-the-Middle-aware 的排序策略，将次优结果放中间、最优结果放首尾

## 相关实体

- [[concepts/rag-retrieval-augmented-generation]] — RAG 架构直接受 Lost in the Middle 影响
- [[concepts/transformer-architecture-2025]] — Transformer 注意力机制是 U 形曲线的技术根因
- [[concepts/speculative-decoding]] — 推测解码优化长上下文推理效率

## 所属 MOC

- [[moc/memory-context-systems|Memory Context Systems]]
