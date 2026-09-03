---
title: "推测解码加速 LLM 推理"
created: 2026-06-30
updated: 2026-08-01
type: concept
tags: [concept, inference, speculative-decoding, llm, latency, optimization]
sources: []
---

## 定义

推测解码（Speculative Decoding）是一种 LLM 推理加速技术：用一个小而快的"草稿模型"（draft model）先快速生成多个候选 token，再用大模型并行验证这些 token 的正确性。被接受的 token 直接采用，被拒绝的 token 从拒绝位置重新采样。在不改变模型输出分布的前提下，推测解码可以将推理速度提升 2-3 倍。

## 核心范式

- **草稿-验证两阶段**：draft model 快速生成 K 个候选 token → target model 并行验证 K 个 token
- **无损加速**：通过修正拒绝采样（rejection sampling），保证输出分布与直接用 target model 生成完全一致
- **投机并行**：draft model 的"猜测"将串行生成转化为并行验证，充分利用 GPU 并行能力
- **Draft model 选择**：同系列小模型（如 Llama-7B 做 Llama-70B 的 draft）、n-gram 模型、Medusa 多头猜测
- **自适应长度**：根据 draft model 的接受率动态调整每次猜测的 token 数量

## 背景与提出

推测解码由 Leviathan et al.（2023）和 Chen et al.（2023）独立提出。核心洞察是 LLM 推理的瓶颈在于自回归解码的串行性——每个 token 都依赖前一个 token 的完整前向传播。而验证多个 token 只需要一次前向传播（通过 causal attention 的特性），所以"猜测-验证"比"逐个生成"更高效。

2024-2025 年推测解码在工业界广泛部署：vLLM、TensorRT-LLM 等主流推理框架内置支持；Medusa（Cai et al., 2024）提出多头架构替代独立 draft model；Eagle（Li et al., 2024）用特征级别的草稿进一步提升接受率。

## 局限与反对声音

- **接受率依赖**：draft model 与 target model 的分布差异越大，接受率越低，加速效果越差
- **额外内存**：需要同时加载 draft model 和 target model，内存占用增加
- **不适用于 batch 场景**：推测解码主要优化单请求延迟，batch 推理时 GPU 已充分利用，加速效果有限
- **Draft model 维护**：需要为每个 target model 单独准备或训练 draft model，增加工程复杂度

## 实践启示

1. **首选同系列小模型**：如用 Llama-3.1-8B 做 Llama-3.1-70B 的 draft model，分布接近，接受率高
2. **Medusa/Eagle 更通用**：不需要独立 draft model，在 target model 上加额外预测头，部署更简单
3. **K 值调优**：每次猜测的 token 数 K 通常 3-5，太大会浪费 draft 计算，太小无法充分利用并行
4. **与量化互补**：推测解码减少延迟，量化减少内存和吞吐成本，两者可叠加
5. **评估真实收益**：在你的实际 prompt 长度和 batch size 下 benchmark，不要只看论文数据

## 相关实体

- [[concepts/moe-mixture-of-experts-2025]] — MoE 模型的稀疏激活特性与推测解码互补
- [[concepts/transformer-architecture-2025]] — Transformer 的 causal attention 特性是推测解码的理论基础
- [[concepts/rag-retrieval-augmented-generation]] — RAG 场景的生成阶段可通过推测解码加速

## 所属 MOC

- [[moc/llm-core-technology|Llm Core Technology]]
