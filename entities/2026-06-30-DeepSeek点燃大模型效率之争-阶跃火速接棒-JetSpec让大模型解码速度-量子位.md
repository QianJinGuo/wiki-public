---
title: "DeepSeek点燃大模型效率之争 阶跃火速接棒 JetSpec让大模型解码速度 量子位"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md|原文存档]]

sha256: 5154ffcbc975958fb9699291e32702d75b4b0b6cdcf727c08e566478d1ee539c ^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md]

## 摘要

文章对比了几乎同期发布的两篇投机解码论文：DeepSeek 的 DSpark 与阶跃星辰参与发表的《JetSpec：Breaking the Scaling Ceiling of Speculative Decoding with Parallel Tree Drafting》。DSpark 关注高并发推理服务中的验证效率（展示生产系统仍有 Flash 模型 60%-85%、Pro 模型 57%-78% 的提速空间，Qwen3-8B/AIME25 上把平均接受长度从 DFlash 的 4.07 提到 5.01）；JetSpec 则从 Draft 生成本身入手，把因果性直接融入并行草稿头生成路径条件化的草稿树：在 Qwen3-8B 上相比标准自回归解码最高实现 9.64 倍端到端解码加速，MATH-500 上一次验证平均可接受 10.76 个 token，HumanEval、LiveCodeBench、MT-Bench 分别加速 7.12 倍、7.67 倍、4.58 倍。JetSpec 作者含阶跃 CEO 姜大昕与 CTO 朱亦博，一作 Lanxiang Hu 为 UCSD 博士生（阶跃实习期间完成），团队此前还合作过 PD 分离开山论文 DistServe；文章认为两者共同说明推理效率正成为 Agent 规模化落地的基础变量。^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md]

## 关键要点

- JetSpec 核心机制：因果并行草稿头生成路径条件化草稿树，深层节点依赖同一分支上更早生成的 token，把更大草稿预算转化为更长可接受前缀
- 关键数据：接受长度从投机预算 16 时的 7.23 提升到预算 128 时的 9.82（同预算下 DFlash 为 7.34、DDTree 为 8.66）；AIME25 上草稿深度 1 接受率约 99%、深度 8 仍保持约 50%，有效逐 token 接受率 alpha_eff 约 93%
- 理论背景：草稿成本下降后瓶颈转移到接受率——逐 token 接受率从 0.85 提到 0.95 即可显著提高最大理论加速比，这决定了 DSpark（吞吐导向、置信度调度验证）与 JetSpec（延迟导向、追求接受长度）的互补定位
- 团队背景：JetSpec 作者含阶跃 CEO 姜大昕、CTO 朱亦博（AI Infra 专家）；一作 Lanxiang Hu 就读 UCSD；阶跃与 UCSD 此前合作过 PD 分离代表作 DistServe，该架构已被 NVIDIA TensorRT-LLM、SGLang、vLLM 采用
- JetSpec 是阶跃 Flash 模型叙事（Step 3.5 Flash → Step 3.7 Flash，面向 Agent 场景的高效智能）在推理算法层面的拼图；项目开源在 GitHub（hao-ai-lab/JetSpec），论文编号 arXiv 2606.18394

## 来源

- 原文: [[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md|DeepSeek点燃大模型效率之争 阶跃火速接棒 JetSpec让大模型解码速度 量子位]]
