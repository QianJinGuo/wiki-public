---
title: "HySparse2：MiMo-V3 新架构预览（两级 KV 共享 + token 级稀疏选择）"
created: 2026-09-25
updated: 2026-09-25
type: entity
tags: [mimo, xiaomi, attention, sparse-attention, kvcache, yoco, long-context, agent, inference-optimization, model-architecture]
sources: [raw/articles/hysparse2xiaomi-mimo-v3-新架构预览]
confidence: 0.75
provenance_state: extracted
vxc: 56
---

# HySparse2：MiMo-V3 新架构预览（两级 KV 共享 + token 级稀疏选择）

小米大模型官方（2026-09-24）公开的 MiMo-V3 核心架构 HySparse2，面向**长程多轮 Agent** 场景的注意力架构升级：更少的 Prefill 计算、更小的 KV Cache、更精准的长上下文检索。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## 背景与问题定义

从 MiMo-V2 系列的 Hybrid SWA（Full Attention + Sliding Window Attention 混合）演进而来；第一代 HySparse 用少量 Full Attention 层提供 KV Cache 和重要位置选择结果，供后续 Sparse Attention 层复用，但 Prefill 仍需执行所有层，块级选择在多轮长距离检索上也有精度提升空间。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

Agent 长程多轮任务对注意力架构的三个要求：①高效读入（减少长输入 Prefill 计算）；②节省显存（降低 KV Cache 占用）；③精准检索（从长历史中找出相关证据、跨轮次信息整合）。一次简短的工具调用可能带回整页网页/文件/执行日志，每步都要处理新增输入。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## 两级 KV 共享（借鉴 YOCO）

HySparse2 将模型分成前后两部分：前半 Self-Decoder 采用 Full Attention + SWA 混合；后半 Cross-Decoder 采用 Full Attention + Sparse Attention 混合。KV 共享分两个层次：^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

- **KV Bridging（跨前后两部分）**：后半部分每个 Full Attention 层从前半部分对应 Full Attention 层的输入隐藏状态生成自己的 KV Cache——每个目标层保留独立 K/V 投影，同一份源隐藏状态可构建不同的 KV。后半部分 Full Attention 层的 KV 不必等输入逐层经过它们后才就绪，"KV 更早就绪"。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]
- **KV Reuse（同一 Hybrid Block 内）**：每个 Hybrid Block = 1 层 Full Attention + 随后多层 Sparse Attention。Full Attention 计算时按注意力分数选出重要位置，后续稀疏层直接复用其 KV Cache 和选择索引——保留 HySparse 核心设计（少量全注意力层提供全局信息与选择结果）。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## token 级稀疏选择 + 统一局部访问

HySparse2 在块级选择基础上引入 **token 级稀疏选择**，提升多轮长距离信息检索的精度；并将局部信息访问（SWA 滑窗）统一进同一机制。三个设计目标直指 Agent 场景的本质约束：历史随任务推进不断增长，而关键证据分散在早期轮次的工具返回中。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## 与已有架构的关系

- 相对 **YOCO**：借其"前半 Self-Decoder / 后半 Cross-Decoder"分层思想，但 Cross-Decoder 内部改为 Full+Sparse 混合而非全 Full；
- 相对第一代 **HySparse**：块级选择 → token 级选择；Prefill 不再全层执行；
- 与 DeepSeek V4.1 的 KV Cache 压缩路线（CSA2 三维联合利用）同属 2026 下半年"长程 Agent 注意力架构"竞赛，但各家切分维度不同（见 [[deepseek-v4-flash-pro-通往百万级上下文与万亿参数推理的新纪元-v2|DeepSeek V4 Flash Pro]]）。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## 定位

这是继 [[mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo-V2.5 推理系统全链路优化（Hybrid SWA）]] 与 MiMo-V2.6 RL 训练之后，MiMo 系列公开的第三代架构路线信号：V2.5 解决"推理系统"效率，V3 的 HySparse2 解决"注意力架构"效率——从系统层下沉到模型结构层服务长程 Agent。

## 相关

- [[mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo-V2.5 推理系统全链路优化]] — 前代 Hybrid SWA 架构与推理系统
- [[concepts/attention-mechanism|注意力机制]] — 稀疏/滑动窗口注意力的基础概念
- [[entities/agent-memory-architecture|Agent 记忆架构]] — 长程 Agent 的另一个互补维度（KV 检索 vs 记忆层）

→ [[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览|原文存档]]
