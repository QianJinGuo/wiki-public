---
title: "MiniMax-H3 on 8×H200: SGLang Diffusion 1.95× Lossless加速"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: [minimax, h200, sglang, diffusion, video-generation, benchmark, inference-optimization]
sources: [raw/articles/minimax-h3-h200-sglang-benchmark-2026]
confidence: 0.8
---

# MiniMax-H3 on 8×H200: SGLang Diffusion 1.95× Lossless加速

> **Background**：SGLang Diffusion团队联合NVIDIA和蚂蚁集团发布的MiniMax-H3视频生成模型在8×H200上的加速benchmark报告。涵盖fused kernels、Cache-DiT step reuse、SubBlock sparse attention三层加速技术的组合效果。

## 核心发现

**无损路径**：SGLang Diffusion的dense无损路径比Diffusers快1.85–1.95×，相同的denoising计算在更快的运行时上执行。^[raw/articles/minimax-h3-h200-sglang-benchmark-2026.md]

**有损加速**：Stacking step reuse + sparse attention可达6.24×加速，SSIM保持0.76–0.91。

**质量-速度权衡矩阵**：

| 模式 | 加速比 | SSIM | 适用场景 |
|------|--------|------|----------|
| 质量优先 (Cache-DiT alone) | 最高2.99× | 0.90–0.92 | 默认推荐 |
| 平衡 (SubBlock 0.75 + Cache-DiT) | 4.90–5.93× | 0.79–0.90 | 生产环境 |
| 速度优先 (SubBlock 0.80 + Cache-DiT) | 5.06–6.24× | 0.76–0.91 | 延迟敏感 |

## 三层加速技术

- **Fused kernels**：降低每步固定开销，不改变数学——单个non-GEMM站点微基准测试加速2.00–12.16×（非端到端可加）
- **Cache-DiT**：复用denoising步间结果，部分step跳过不运行
- **SubBlock sparse attention**：NVIDIA block-sparse forward，跳过贡献低于阈值的attention blocks

三层可组合：fused kernels是无损的，后两者在质量-速度间权衡。

## 关键参数

- 硬件：8× NVIDIA H200 (141 GB)
- 工作负载：MiniMax-H3 · 1344×768 · 24 FPS · 50 denoising steps
- 版本：SGLang v0.5.18
- 测量日期：2026-08-18

## 实践启示

对于视频生成推理优化，三层加速的组合策略提供了从无损到有损的完整权衡空间。Cache-DiT作为质量优先默认选择（SSIM 0.90+），SubBlock sparse attention在速度优先场景下提供额外2×加速但质量下降。^[raw/articles/minimax-h3-h200-sglang-benchmark-2026.md]

→ [[raw/articles/minimax-h3-h200-sglang-benchmark-2026|原文存档]]
