---
title: "Nunchaku — 4-bit Diffusion 推理加速"
created: 2026-08-14
updated: 2026-08-14
type: entity
tags: [nunchaku, quantization, 4-bit, diffusion, inference, svdquant, mit-han-lab]
sources: [raw/articles/nunchaku-4bit-diffusion-inference-diffusers]
confidence: 0.75
provenance_state: extracted
---

# Nunchaku — 4-bit Diffusion 推理加速

MIT Han Lab 的 Nunchaku 量化引擎接入 diffusers 生态，实现 **4-bit diffusion 模型推理**——把 SVDQuant 等低比特量化方案从实验原型变为 diffusers 可开箱使用的后端。^[raw/articles/nunchaku-4bit-diffusion-inference-diffusers.md]

## 关键点

- **4-bit 量化**：扩散模型权重压到 4-bit，显著降低显存占用
- **diffusers 集成**：作为推理后端直接可用，降低使用门槛
- **开源生态**：MIT Han Lab 出品，与 [[entities/quantization-techniques|量化技术]] 家族一致

## 定位

属于 [[concepts/inference-optimization|推理优化]] 在生成式视觉模型上的延伸——与 LLM 侧 4-bit 量化（GGUF/GPTQ 等）对应，扩散模型的低比特推理是显存受限部署（本地/边缘）的关键路径。

→ [[raw/articles/nunchaku-4bit-diffusion-inference-diffusers|原文存档]]
