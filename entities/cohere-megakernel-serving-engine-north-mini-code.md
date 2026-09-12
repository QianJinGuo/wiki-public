---
title: "Cohere Megakernel Decode Serving Engine (North Mini Code)"
created: 2026-09-10
updated: 2026-09-10
type: entity
tags: [inference-optimization, kernel-engineering, megakernel, serving, vllm, cuda]
sources: [raw/articles/cohere-megakernel-serving-engine-north-mini-code]
confidence: 0.75
---

# Cohere Megakernel Decode Serving Engine

Cohere（2026-09-08，Xiaochun Tong/Conway Zhu/Donglu Wang）发布围绕 decode megakernel 构建的完整 serving engine，服务其 North Mini Code（30B 模型、每 token 3.3B 活跃参数、BF16 下每 decode step 需流式搬运 6.6GB 权重）。核心论点：低 batch decode 本质是 memory-bound，正确的问题不是 flops 而是 memory bandwidth 利用率。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md]

## 性能数据

H100（3.35 TB/s HBM）上 Speed-of-Light ≈ 470 tok/s；vLLM 服务该模型为 185 tok/s（仅 39% SoL）。Megakernel 在 batch=1 达 292 tok/s（62% SoL），比 vLLM 快 1.58×，端到端快 1.25-1.41×，优势跨 batch size 与 256K context 保持，无精度损失。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md]

## 架构设计

传统 serving 把一次 forward pass 拆成上百个小 kernel，GPU 在 launch 间隙空等。Megakernel = 单个 persistent kernel 跑完整 forward pass：每 SM 一个 threadblock 常驻整个 decode step；数据依赖不靠 kernel 边界编码，而用 global memory 中的显式计数器（task 完成时递增、需要输入时自旋等待）；block 从 host 准备的 task list 取活。这是首个支持 continuous batching、paged attention、ragged sequence length 与 OpenAI-compatible endpoint（含 tool calling）的 megakernel serving 系统。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md]

## 可迁移性

作者强调 megakernel 比名声上更容易写：单个 CUDA 文件，无编译器、无新编程范式——普通 tiled GEMM + 普通 paged attention 重构进单一 calling convention 即可，并附 kernel 移植 recipe。代码开源在 GitHub。该工程模式（memory-bound 分析 → task-list 调度 → persistent kernel）可迁移到其他 decode 场景，与 vLLM 等框架的 kernel-launch 模型形成对照。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md]

## 关联

- 推理优化对比基线：[[entities/vllm|vLLM]]、[[entities/ai-infra-llm-efficient-inference-vllm|AI Infra LLM 高效推理]]
- 效率叙事对照：[[entities/magic-10x-more-efficient-pretraining|Magic 10x 高效预训练]]

→ [[raw/articles/cohere-megakernel-serving-engine-north-mini-code|原文存档]]
