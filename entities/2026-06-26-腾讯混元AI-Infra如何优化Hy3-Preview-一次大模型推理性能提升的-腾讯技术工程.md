---
title: "腾讯混元AI Infra如何优化Hy3 Preview 一次大模型推理性能提升的 腾讯技术工程"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md|原文存档]]

sha256: d5481a4c017610659bcb55ba77bc8938598fcba990e15bd055b7329195930259 ^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md]

## 摘要

腾讯混元 AI Infra 推理团队剖析了 Hy3 preview 模型（GQA+MoE 混合架构、原生 256K 上下文）在算力更低、显存更紧凑的 NVIDIA Hopper 卡上的推理全栈优化实践，从算子优化与融合、并行策略、多级缓存、MTP 和异步调度、量化与稀疏五大维度展开。算子层面：动态调度的 Attention 负载均衡在单 batch 长文本场景单算子最高加速 2.95x；Router GEMM 用双 BF16 重构 FP32 计算，相比 cuBLAS FP32 实现加速 2.86x~3.22x；FusedMoE 流水线重构相比 vLLM/SGLang 加速 1.5x-1.6x；采样算子将十余个 Kernel 融合为 2 个，全词表只加载 1 次，相比 vLLM 提升约 5.5x。^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md]

并行与系统层面：Prefill 采用 TPSP 并行策略使 32k 上下文 TTFT 降低 24.5%；Decode 采用 Attention DP + MoE EP 跨节点混合并行，端到端吞吐提升 15.7%~44.7%；构建 GPU→CPU→KVStore 三级缓存体系扩大有效缓存容量；MTP 场景下解除 CPU 对真实接收长度的同步依赖，消除 decode 间 5~10ms CPU 气泡、端到端提升 10%~20%。量化与稀疏层面：AngelSlim 框架通过"GPTQ 权重重建 + 激活平滑与旋转变换 + QAT 轻量微调"实现 W4A8 + Attn FP8 精度无损（与 BF16 差距 <1%）、吞吐提升 28%+；自研 Stem 稀疏注意力算法配合 HPC-BSA 算子，仅用 25% 计算预算达到接近稠密注意力精度，128K 上下文 Prefill 延迟降低 3.6 倍。测试基于 5000 条真实数据（最大输入 192k、平均 68k），约束 50ms TPOT、4s TTFT。^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md]

## 关键要点

- Fused Rope+Norm+[Hadamard]+Quant+Store KV 将 5 个低算力强度算子融合为单 Kernel，融合算子加速约 5x。
- Fused AllReduce+Norm+Add 与腾讯网络平台部联合实现，基于 NVSwitch 多播与 Lamport P2P 两种版本，相比 NCCL/FlashInfer 最高加速 1.68x。
- GEMM 与 ReduceScatter 细粒度通算融合将 SM 显式划分为计算/通信两类角色，端到端加速比 1.68x~1.81x。
- Stem 稀疏注意力的两个关键技术：Token Position-Decay（Top-k 预算从头部向尾部线性衰减）和 Output-Aware Metric（把 Value 模长引入 token 选择标准）。
- 后续方向：C4 与 W4 量化优化、并行投机解码、PD 高效传输、多级缓存中心及其他硬件平台适配。

## 来源

- 原文: [[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md|腾讯混元AI Infra如何优化Hy3 Preview 一次大模型推理性能提升的 腾讯技术工程]]
