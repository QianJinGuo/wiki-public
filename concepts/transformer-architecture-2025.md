---
title: "Transformer 架构"
created: 2026-06-30
updated: 2026-08-01
type: concept
tags: [concept, architecture, transformer, attention, deep-learning, llm, foundation]
sources: []
---

## 定义

Transformer 是一种基于自注意力（self-attention）机制的神经网络架构，由 Vaswani et al.（2017）在 "Attention Is All You Need" 中提出。它彻底取代了 RNN/LSTM 成为 NLP 的主流架构，并扩展到视觉（ViT）、音频（Whisper）、多模态（GPT-4V）等领域。当前所有主流 LLM（GPT-4、Claude、Llama、DeepSeek、Gemini）均基于 Transformer 架构或其变体。

## 核心范式

- **自注意力机制**：每个 token 与序列中所有其他 token 计算注意力权重，实现全局信息交互
- **并行计算**：与 RNN 的串行处理不同，Transformer 的所有位置可以并行计算，充分利用 GPU
- **位置编码**：自注意力本身不含位置信息，需要额外的位置编码（sinusoidal、RoPE、ALiBi）注入序列顺序
- **Encoder-Decoder 到 Decoder-only**：原始 Transformer 是 encoder-decoder，GPT 系列将其简化为 decoder-only（因果注意力），成为 LLM 的标准架构
- **Scaling Law**：Transformer 的性能随参数量、数据量、计算量的增加呈幂律提升（Kaplan et al., 2020）

## 背景与提出

2017 年 Google Brain 团队提出 Transformer 时，核心动机是解决 RNN 的两个瓶颈：串行计算导致训练慢，长距离依赖难以捕获。Transformer 的 self-attention 机制一次计算就能建立任意两个位置之间的直接连接，且完全并行化。原始论文标题 "Attention Is All You Need" 暗示了其极简主义——用纯注意力机制替代所有循环和卷积。

Transformer 的影响远超 NLP 领域：Vision Transformer（ViT, 2020）证明纯注意力架构在图像分类上可以匹敌 CNN；AlphaFold 2（2020）用 Transformer 预测蛋白质结构；Whisper（2022）用 Transformer 做语音识别。到 2025 年，Transformer 已成为深度学习的"通用计算基底"。

## 局限与反对声音

- **二次复杂度**：标准 self-attention 的计算和内存复杂度是 O(n²)，长序列（>32K tokens）计算成本极高
- **位置编码外推**：模型在训练长度外的序列上表现下降，需要 RoPE 缩放、YaRN 等技术扩展
- **推理效率**：自回归解码是串行的，每生成一个 token 都需要完整前向传播
- **Mamba/SSM 挑战**：状态空间模型（Mamba, 2023）以 O(n) 线性复杂度挑战 Transformer 的主导地位，但在大规模场景下 Transformer 仍然胜出
- **能效问题**：Transformer 的密集计算模式能耗巨大，推动了稀疏注意力、MoE 等优化方向

## 实践启示

1. **理解注意力是基础**：所有 LLM 工程优化（prompt engineering、RAG、微调）都建立在对注意力机制的理解之上
2. **上下文窗口决定应用边界**：选择模型时关注有效上下文长度（不是标称长度），RAG 场景尤其重要
3. **Decoder-only 是默认选择**：除非有特殊需求（如需要双向编码的分类任务），默认使用 decoder-only 架构
4. **RoPE 是主流位置编码**：Llama、Qwen、DeepSeek 等主流模型均使用 RoPE，支持 YaRN/NTK 扩展
5. **关注架构变体**：GQA（Grouped Query Attention）、FlashAttention、Ring Attention 等改进持续推动效率提升

## 相关实体

- [[concepts/moe-mixture-of-experts-2025]] — MoE 是 Transformer 的稀疏化扩展
- [[concepts/speculative-decoding]] — 推测解码利用 Transformer 的因果注意力特性加速推理
- [[concepts/lost-in-the-middle]] — Transformer 注意力的 U 形分布导致长上下文信息丢失
- [[concepts/rag-retrieval-augmented-generation]] — RAG 架构建立在 Transformer 的上下文学习能力之上
- [[entities/deepseek-v3-moe-architecture]] — DeepSeek V3 是 Transformer + MoE 的代表性实现

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
