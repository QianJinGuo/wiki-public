---
title: "TPU Megakernels：Inferact 在 Kimi K3 上跑出 700 TPS"
created: 2026-09-26
updated: 2026-09-26
type: entity
tags: [inference-optimization, tpu, megakernel, kimi-k3, speculative-decoding, llm-inference, vmem]
sources: [raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026]
confidence: 0.75
provenance_state: extracted
---

# TPU Megakernels：Inferact 在 Kimi K3 上跑出 700 TPS

Inferact 开源 tpu-megakernels（TPU v7 Ironwood）：Kimi K3 speculative decoding 下 700+ tokens/s（GB200 基线 452）；无 spec decoding 时 batch 1-8 解码吞吐 1.4-2× GB200。核心设计：TPU 大容量片上 VMEM + 顺序编程模型 + 跨层异步权重预取，把端到端解码推向 HBM 带宽理论峰值。 ^[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026.md]

## 相关链接

- [[concepts/inference-optimization|推理优化]]
- [[entities/cohere-megakernel-serving-engine-north-mini-code|Cohere North megakernel serving]]

→ [[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026|原文存档]]
