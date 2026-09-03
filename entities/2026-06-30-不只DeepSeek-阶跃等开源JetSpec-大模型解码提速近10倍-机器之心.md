---
title: "不只DeepSeek 阶跃等开源JetSpec 大模型解码提速近10倍 机器之心"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-30-不只DeepSeek-阶跃等开源JetSpec-大模型解码提速近10倍-机器之心]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-30-不只DeepSeek-阶跃等开源JetSpec-大模型解码提速近10倍-机器之心.md|原文存档]]

sha256: a838ada2aa8f6bb5ee2d049707683d04bb2c791b0c74d4ae1debcbd86e7b2f28 ^[raw/articles/2026-06-30-不只DeepSeek-阶跃等开源JetSpec-大模型解码提速近10倍-机器之心.md]

## 摘要

机器之心报道阶跃星辰开源的投机解码框架 JetSpec：当 Agent 高频调用大模型时，单次推理延迟被连续放大，推理效率成为规模化落地的基础变量。JetSpec 从 Draft 生成本身入手，将因果性直接融入并行草稿头，生成路径条件化的草稿树，提高一次验证能接受的 token 数。在 Qwen3-8B 上，JetSpec 相比标准自回归解码最高实现 9.64× 端到端解码加速，MATH-500 上一次验证平均可接受 10.76 个 token；HumanEval、LiveCodeBench、MT-Bench 分别加速 7.12×、7.67× 和 4.58× ^[raw/articles/2026-06-30-不只DeepSeek-阶跃等开源JetSpec-大模型解码提速近10倍-机器之心.md]

文章将 JetSpec 与同期 DeepSeek 的 DSpark 对比：DSpark 面向高并发、预算受限的服务场景，用置信度调度验证把平均接受长度从 4.07 提到 5.01；JetSpec 面向低并发低延迟场景，把平均接受长度从预算 16 时的 7.23 提升到预算 128 时的 9.82，超过 DFlash 的 7.34 和 DDTree 的 8.66。两者共同指向投机解码的核心瓶颈——草稿变便宜后，因果一致性与并行效率的两难困境。JetSpec 一作为 UCSD 的 Lanxiang Hu（阶跃实习期间完成），作者含阶跃 CEO 姜大昕、CTO 朱亦博；阶跃与 UCSD 此前还合作过 PD 分离开山论文 DistServe ^[raw/articles/2026-06-30-不只DeepSeek-阶跃等开源JetSpec-大模型解码提速近10倍-机器之心.md]

## 关键要点

- JetSpec 在 Qwen3-8B / MATH-500 上最高 9.64× 端到端解码加速，一次验证平均接受 10.76 个 token；AIME25 上深度 1 处逐位置接受率约 99%，深度 8 处仍保持约 50%。
- 核心机制：因果并行草稿头——草稿树更深层节点依赖同一分支上更早生成的 token，解决块并行草稿（DFlash）「局部合理、整体不一致」导致的接受率稀释问题。
- DSpark（DeepSeek，作者含梁文锋）走互补路线：高并发吞吐导向，用 Markov 风格修正头 + 置信度调度控制验证预算。
- JetSpec 开源在 github.com/hao-ai-lab/JetSpec，论文 arXiv:2606.18394；草稿树预算 256 token。
- 阶跃与 UCSD 此前合作 DistServe（Prefill-Decode 分离），该架构已被 NVIDIA TensorRT-LLM、SGLang、vLLM 采用。

## 来源

- 原文: [[raw/articles/2026-06-30-不只DeepSeek-阶跃等开源JetSpec-大模型解码提速近10倍-机器之心.md|不只DeepSeek 阶跃等开源JetSpec 大模型解码提速近10倍 机器之心]]
