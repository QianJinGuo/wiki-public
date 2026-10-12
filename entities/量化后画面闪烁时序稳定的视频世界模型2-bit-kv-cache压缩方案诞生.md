---
title: "QuantWM：视频世界模型 2-bit KV Cache 量化"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [world-model, quantization, kv-cache, video-generation]
sources: [raw/articles/量化后画面闪烁时序稳定的视频世界模型2-bit-kv-cache压缩方案诞生]
confidence: 0.75
---

# QuantWM：视频世界模型 2-bit KV Cache 量化

## 深度分析

哈工大（深圳）iLearn-Lab + NUS LV-Lab 的 QuantWM（arXiv:2609.26425）：免训练 2-bit KV Cache 量化框架，五种模型最高 6.20 倍压缩。核心发现：VBench 分数几乎不变 ≠ 生成质量不退化——HY-World 1.5 的 Temporal flickering BF16 95.62% vs QVG 95.85%，但画面闪烁严重，连闪烁相关指标都未捕捉退化。机制归因：Key 是检索「索引」、Value 是「内容」，Key 量化误差更小却造成更大输出误差——Key 量化破坏注意力排序导致闪烁，需从注意力保护角度设计量化。 ^[raw/articles/量化后画面闪烁时序稳定的视频世界模型2-bit-kv-cache压缩方案诞生.md]

## 关联

- [[concepts/world-models|世界模型]]
- [[entities/a2rd-agentic-autoregressive-diffusion-long-video|A2RD 自回归扩散]]

→ [[raw/articles/量化后画面闪烁时序稳定的视频世界模型2-bit-kv-cache压缩方案诞生|原文存档]]
