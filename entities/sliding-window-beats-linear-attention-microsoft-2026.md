---
title: "Sliding-window beats linear attention (微软 SWA vs 线性注意力)"
created: 2026-09-02
updated: 2026-09-07
type: entity
tags: [attention, linear-attention, sliding-window, inference, kv-cache, microsoft, transformer, arxiv]
sources: [raw/articles/sliding-window-beats-linear-attention-microsoft-2026]
confidence: 0.85
arxiv: "https://arxiv.org/abs/2608.28444"
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Sliding-window beats linear attention (微软 SWA vs 线性注意力)

微软团队发表论文证明：带 attention sinks 的滑动窗口注意力 (SWA) 在零后训练投入下，性能接近甚至超越需要数亿 token 后训练的线性化方法。

## 核心发现

### 0 Token vs. 千亿 Token

线性注意力方案（LoLCATs、Liger-GLA、QRWKV6、SUPRA 等）通过蒸馏或后训练将现成 Transformer 改造为线性化模型，训练量从 20M 到 100B token 不等。

微软提出的 **SWA(64,4)** — 总窗口 64、保留 4 个 attention sinks — **零 token、零后训练阶段**，在六项知识与推理任务上平均恢复原模型 **99.0%** 性能，与需要 350M–700M token 后训练的 QRWKV6（99.1%）几乎持平。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

MMLU 上 SWA 恢复率 **93.2%**，略高于 QRWKV6 的 92.4%。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

### 实验覆盖

覆盖 Phi、Mistral、Llama、Qwen、QwQ 等多个模型系列，参数规模从 1.3B 到数十 B。11 组比较中 SWA 有 9 组拿到除全注意力外的最高平均成绩。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

Qwen3-8B 上 SWA 平均 71.6，而 Gated DeltaNet、GLA、QRWKV6 经约 100M token 两阶段蒸馏后仍停留在 50 多分。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

### 长上下文优势

SWA 有效感受野通过网络深度逐层扩展（l 层后约 lw）。在 Llama 3.1 8B 的 S-NIAH 测试中：

- 窗口 128 + 上下文 4K：SWA 准确率 17.2 vs LoLCATs 1.6（**10.8 倍差距**）^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]
- BABILong 4K：SWA 15 vs LoLCATs 3^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

### 工程优势

- 窗口 < 512 时，SWA 状态内存与线性注意力相当或更低，吞吐更快^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]
- KV Cache 达窗口上限后保持固定，解码吞吐稳定^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]
- 窗口 64 时速度最快、内存最低^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

### 关键洞察

论文指出过去线性化工作很少直接和带 sinks 的 SWA 比较——普通 SWA（不带 sinks）在窗口滑动后 sink token 丢失会导致灾难性下降，容易低估 SWA 能力。SWA + attention sinks 已成为后续线性化工作难以绕开的 baseline。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

## 与 [[entities/mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo v2.5 混合 SWA]] 的关系

MiMo 的混合 SWA 架构也采用滑动窗口注意力作为推理优化手段。本文从理论上验证了 SWA 作为线性注意力替代方案的有效性，为 MiMo 等采用 SWA 的模型提供了理论支撑。

## → [[raw/articles/sliding-window-beats-linear-attention-microsoft-2026|原文存档]]
