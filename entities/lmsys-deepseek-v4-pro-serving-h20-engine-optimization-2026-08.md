---
title: "LMSYS 在 H20 上服务 DeepSeek-V4-Pro 的场景化 Serving 优化"
created: 2026-08-20
updated: 2026-09-07
type: entity
tags: [ml, inference, serving, moe, lmsys, speculative-decoding, optimization]
sources: [raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LMSYS 在 H20 上服务 DeepSeek-V4-Pro 的场景化 Serving 优化

## 概述

LMSYS 团队发表了一篇关于在 **NVIDIA H20 GPU**（而非 Blackwell）上服务 1.6T 参数 MoE 模型 **DeepSeek-V4-Pro** 的系统优化长文。核心论点是「一个模型需要多个 serving profile」：没有一种通用配置能同时满足长上下文 prefill、交互式 decode 的 TPOT、以及高并发吞吐的 KV 容量约束，必须按工作负载、SLO 与实测硬件行为分别选择部署拓扑与执行路径。^[raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08.md]

## 核心方法论：场景化 Serving Profile

文章提出「场景驱动」的 serving 方法论：prefill 依据实测 context-length 在 **PP2 与 PP4** 间切换，decode 则按低延迟/高吞吐/KV 容量分别优化。关键设计决策包括：^[raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08.md]

- **Prefill 用 MoE-TP 而非 MoE-EP**：MoE-EP 虽只交换路由 token，但真实 prefill 流量存在显著 expert 倾斜，hot expert 所在 rank 成为 straggler；MoE-TP 的全序列 all-gather/reduce-scatter 走 NVLink 高带宽、成本可预测，「可预测的通信比不可预测的不均衡更便宜」。
- **容量取舍**：用 **Humming MXFP4AFP8**（MXFP4 expert 权重 + 在线 FP8 激活）缩减权重足迹，叠加 **Online C128** 压缩 KV 辅助状态，组合容量提升至 baseline 的 3.88×（DP32-EP32）/ 10.14×（PP2-TP8）。
- **低延迟 decode 用 PP2-TP8，高吞吐用 DP32-EP32**：单节点 TP8 最快但 KV 容量受限；PP2-TP8 跨节点分摊权重换取 KV 空间；DP32-EP32 扩展 EP 组释放 HBM 提升并发容量（256K/512K/1M 并发请求 2× 扩展）。

## 关键量化结果

论文给出了 H20 与 B300 的对比与多场景实测：^[raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08.md]

- batch size 1 时单节点 H20-141GB 达 **271 tokens/s**，对比 B300 的 383.7 tokens/s，优化后观测差距缩至 **1.42×**（尽管 B300 峰值 Tensor Core 算力为 H20 的 45.6×）。
- 优化 prefill 达 **8.45k input tokens/s/node**，1M-token prompt 处理耗时 **43.7s**；高吞吐 decode（DP16-EP16）达 **4.67k output tokens/s/node**，平均 TPOT 27.4ms。
- DSpark 投机解码优化后 peak TPOT 在 batch size 1 下降 **74.8%–78.0%**，大 batch 仍降 52.2%–60.0%；Humming decode 热路径融合 SwiGLU 量化后 per-GPU 吞吐提升 **44.0%**。

## 与既有知识的关联

这篇文章是 LMSYS serving 优化系列的重要延续，与 [[entities/lmsys-dflash-speculative-decoding-2026-06|DFlash 投机解码]]、[[entities/agent-assisted-sglang-development-lmsys-2026-07|LMSYS Agent 辅助 SGLang 开发]] 同源同团队；其 DSpark 投机解码路径与 [[entities/deepseek-dspark-v4-speculative-decoding-deepspec|DeepSeek DSpark V4]] 一脉相承。方法论层面可对照 [[entities/ai-infra-llm-efficient-inference-vllm|LLM 高效推理]] 与 [[entities/native-speed-vllm-transformers-modeling-backend|vLLM Native Speed]]，其「可预测通信优于不可预测不均衡」的 MoE-TP 论证是通用可迁移结论。

→ [[raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08|原文存档]]
