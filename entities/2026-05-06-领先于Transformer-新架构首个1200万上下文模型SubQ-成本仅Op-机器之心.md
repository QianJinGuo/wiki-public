---
title: "领先于Transformer 新架构首个1200万上下文模型SubQ 成本仅Op 机器之心"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-06-领先于Transformer-新架构首个1200万上下文模型SubQ-成本仅Op-机器之心]
provenance_state: extracted
---

> -> [[raw/articles/2026-05-06-领先于Transformer-新架构首个1200万上下文模型SubQ-成本仅Op-机器之心.md|原文存档]]

sha256: 4c85f0eda05fcae3a1252ceeee84cc621696e7e8b547b20ff5ac70550d93f266 ^[raw/articles/2026-05-06-领先于Transformer-新架构首个1200万上下文模型SubQ-成本仅Op-机器之心.md]

## 摘要

机器之心介绍了 Subquadratic 公司发布的 SubQ 模型——首个基于完全亚二次稀疏注意力架构（SSA）构建、首个拥有 1200 万 token 上下文窗口的前沿模型，联合创始人 Alexander Whedon 称其为"LLM 智能的一次重大突破"：100 万 token 场景下比 FlashAttention 快 52 倍，成本不到 Opus 的 5%，有望将计算量降低近 1000 倍。文章先论证了稠密注意力的困境：all-pairs 计算随序列长度二次增长（上下文翻倍成本变四倍），且训练好的模型中绝大多数注意力权重接近于零——"稠密注意力不仅是二次复杂度，而且是浪费性的二次复杂度"；FlashAttention 只是执行得更高效，RAG、上下文压缩、Agent 编排等系统层补救则是在绕开而非消除二次成本这条边界，且 RAG 丢失位置信息与引用关系、Agent 工作流的错误在步骤间累积。企业真正的难题本质都是长上下文的多跳推理问题（代码库跨模块调用、合同跨页引用、科研证据整合）。^[raw/articles/2026-05-06-领先于Transformer-新架构首个1200万上下文模型SubQ-成本仅Op-机器之心.md]

SSA 的核心是"基于内容的选择"：对每个 query 先判断序列中哪些位置值得关注，只在这些位置精确计算注意力，从而同时具备三个特性——计算与内存线性扩展（成本取决于被选中位置数而非序列长度）、基于内容的路由能力（按语义决定"去哪里看"）、从任意位置进行稀疏检索（保留恢复任意远位置具体信息的能力）。实测 wall-clock 加速随上下文长度指数级放大：128K 时 7.2 倍、256K 时 13.2 倍、512K 时 23.0 倍、1M 时 52.2 倍（B200 GPU 上对比 FlashAttention-2，后者在 B200 上已无额外收益）；100 万 token 规模注意力 FLOPs 降低 62.5 倍。训练采用预训练、监督微调、强化学习三阶段，RL 阶段针对"看起来合理的失效"（基于邻近上下文作答、局部正确的补丁违反他处接口定义）设计，用高信息密度、跨引用结构的长文本迫使选择机制学会大跨度路由。评估强调"功能上下文"而非"名义上下文"：MRCR v2 上 SubQ 得分 65.9%，处于 Claude Opus 4.6 的 78 分区间内，领先 GPT-5.4 的 39 分和 Gemini 3.1 Pro 的 23 分。^[raw/articles/2026-05-06-领先于Transformer-新架构首个1200万上下文模型SubQ-成本仅Op-机器之心.md]

## 关键要点

- RULER 基准覆盖多跳检索、信息聚合、变量跟踪、选择性过滤，检验多跳任务的"连锁放大效应"——链条早期遗漏一个关键引用会污染后续每步推理。
- 上下文翻倍时 SSA 计算成本仅翻倍（线性），而传统二次复杂度增长四倍——这是"吞吐反转"：上下文越长，稠密注意力相对 SSA 越慢。
- SSA 全称有两处表述：亚二次稀疏注意力（Subquadratic Sparse Attention）与亚二次选择性注意力（Subquadratic Selective Attention），文章注明二者所指机制相同但命名不同。
- 技术博客标题为 How SSA Makes Long Context Practical，地址 subq.ai/how-ssa-makes-long-context-practical。
- 评估维度有二：部署可行性（计算量削减与 wall-clock 速度）与检索能力（RULER 与 MRCR v2），另提及 SWE-Bench Verified 用于评估真实 GitHub issue 的端到端工程能力。

## 来源

- 原文: [[raw/articles/2026-05-06-领先于Transformer-新架构首个1200万上下文模型SubQ-成本仅Op-机器之心.md|领先于Transformer 新架构首个1200万上下文模型SubQ 成本仅Op 机器之心]]
