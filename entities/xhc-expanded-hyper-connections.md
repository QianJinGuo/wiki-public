---
title: "xHC: Expanded Hyper-Connections — 16-Way Residual Stream Architecture"
created: 2026-07-22
updated: 2026-09-17
type: entity
tags: [hyper-connections, residual-stream, architecture, transformer, deepseek, moe]
sources: [raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构]
confidence: 0.6
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# xHC: Expanded Hyper-Connections

xHC (Expanded Hyper-Connections) is a neural architecture that extends [[deepseek-v3-moe-architecture|DeepSeek]]'s mHC (Multi-path Hyper-Connections) to support **16 residual streams** with **sparse updates and multi-scale temporal feature enhancement**. Developed by researchers from Shanghai Jiao Tong University and Xiaohongshu, xHC addresses the saturation problem that arises when scaling mHC beyond 4 residual paths. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md]

## Motivation

The original Hyper-Connection (HC) expanded single residual streams to multiple paths; mHC added constrained cross-stream mixing via Sinkhorn normalization for training stability. However, scaling from 4 to 16 paths in mHC incurred a **32% increase in training FLOPs** for only **0.006 loss reduction** — indicating that additional streams lacked sufficiently diverse write-back signals while bearing growing dynamic mixing costs. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md]

## Key Design

xHC introduces three innovations over mHC:

### Dense Read, Sparse Update
xHC maintains 16 full residual states. Each sublayer reads all 16 streams but writes to only **k=4** of them (2 permanently active + 2 dynamically selected by a sigmoid-gated router). This reduces the projection cost for the residual mixing matrix from O(N²) to O(N·k), bringing FLOPs overhead down to ~4% while keeping access to the full state space. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md]

### Multi-Scale Temporal Feature Enhancement
After the MLP output, xHC applies r=3 causal 1D convolution branches with kernel sizes covering different local context ranges. These convolved outputs are concatenated with the original MLP output, producing 4 write-back components. A modified Gram–Schmidt process removes collinear components. This mechanism enriches the diversity of write-back signals without re-executing the MLP per stream. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md]

### Extensibility Beyond 4 Streams
While mHC saturates at 4 paths, xHC's dense-read/sparse-update architecture scales efficiently: loss reduction from 4→16 paths jumps from 0.006 (mHC) to 0.012 (xHC), with parameter count per layer at ~1/7 of dense mHC for the same 16-stream configuration. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md]

## Results

On DeepSeekMoE-style backbones (18B/28B, ~1.7B/2.7B activated):
- **18B avg score**: mHC 44.8 → xHC 48.8
- **28B avg score**: mHC 50.5 → xHC 53.6
- Gains are distributed across knowledge, reasoning, math, code, and Chinese language tasks
- Additional training FLOPs relative to standard residual baseline: only **3.3%** (vs mHC's ~20% for 16-path dense)

Scaling law analysis confirms that xHC reaches the same loss target with significantly less compute than both standard residual and mHC baselines. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md]

→ [[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构|原文存档]]

---
## 深度分析

### Why mHC saturates past four paths

mHC's saturation past four paths is a signal-diversity ceiling, not a capacity ceiling: the quantity written into stream i is still one shared write-back vector whose only per-stream freedom is a scalar weight, so streams weight an identical vector instead of carrying different content, and their accumulated histories converge. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:124-137] Cost moves the other way: predicting N coefficients from an N-dimensional state makes the mixing projection grow with N, so each added stream carries a growing dynamic-mixing bill for a shrinking return — on a 2.5B [[concepts/moe-mixture-of-experts-2025|MoE]] model, 4 to 16 paths cost +32% training FLOPs and bought 0.006 of loss. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:109-111] xHC makes the same jump with 0.012 of loss at roughly 4% extra FLOPs, which locates the binding constraint in write-back signal supply rather than in path count. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:49]

### Dense read, sparse update: splitting breadth from update width

xHC separates stream count from per-sublayer update width. All 16 states are read, but only k=4 are mixed and written: two permanently active streams plus two chosen per sublayer by a sigmoid-scored router. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:211-224] Generating the mixing matrix over the active set alone drops the dominant projection cost from N to k, while routing and pre-mapping still address the whole state, so the full state space stays reachable through a narrow write set; unselected streams simply pass through the sublayer untouched. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:234] Both halves earn their place in the ablations — removing the dense read or freezing stream selection each raises validation loss, and k=4 was adopted as the loss-versus-overhead sweet spot. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:247]

### Multi-scale temporal write-back and the decorrelation step

The second lever is write-back diversity. After the MLP output, xHC appends r causal depthwise 1-D convolution branches at different kernel sizes and concatenates them with the original output, giving four write-back components from a single MLP execution that is never re-run per stream. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:148-159] A modified Gram–Schmidt pass then removes collinear components so streams are not fed near-duplicates; the mechanism stays MLP-only because [[concepts/attention-mechanism|attention]] already mixes across tokens. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:164] The ablation ladder is monotone: one branch moves validation loss from 1.998 to 1.989 and three scales reach 1.984, evidence that different time ranges carry complementary information. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:179] Decorrelation behaves as a scaling-time safety property rather than a loss optimizer — nearly free at 10B, destabilizing when removed at 18B. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:184] Note the trade-off: temporal enhancement repairs the information supply but not the cost, since dense 16-path mHC still pays 20.1% extra FLOPs with the module installed. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:194]

### Efficiency accounting: where the 3.3% comes from

Sparse update plus temporal enhancement lands xHC at validation loss 1.983 with 3.3% extra training FLOPs over the standard residual baseline — roughly a sixth of dense mHC's 20.1% — and per-layer extra parameters of about one seventh of dense mHC at the same 16 streams, because the mixing projections scale with k rather than with N. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:252] Those 3.3% cover arithmetic, not state traffic: routing and pre-mapping still read all 16 states, so the memory-access profile stays that of a 16-stream design — precisely the gap the Flash variant targets, which is why I/O estimates are reported alongside loss. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:224]

### Evidence base, scaling behaviour, and what xHC-Flash adds

The pretraining numbers come from [[entities/deepseek-v3-moe-architecture|DeepSeekMoE-style]] 18B and 28B backbones with roughly 1.7B and 2.7B activated parameters, not released DeepSeek checkpoints, with mHC re-implemented under the same recipe. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:266] Average scores rise from 44.8 to 48.8 and from 50.5 to 53.6, with gains distributed across knowledge, reasoning, math, code and Chinese rather than driven by a few items. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:282] Optimization is hybrid — Muon on the 2-D backbone weights, AdamW for xHC-specific parameters — and the same run also dropped Gram–Schmidt, making the result a joint recipe rather than an isolated architectural delta. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:287] At fixed E=0.72, the fitted [[concepts/scaling-laws|scaling law]] puts the standard-residual baseline at about 1.50x and mHC at about 1.19x the compute needed to reach xHC's loss. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:300-305] xHC-Flash trades a bounded approximation for I/O: one routing decision is reused across a short shared window, per-sublayer pre-mappings and the base readout come from the window-entry state, and later sublayers need only an exact correction instead of a fresh full-state scan — estimated I/O falls from 73.5C to 51C, and to 40C for the 4-sublayer variant, at about +11% training time versus a re-implemented 4-path mHC and roughly +1.3% prefill latency at 2K tokens. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:340-346] What remains untested is structural, not cosmetic: only N=16 with k=4 and DeepSeekMoE-style backbones were validated, leaving larger expansion rates and other architectures open. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:370]

## 实践启示

The takeaways generalize the design moves rather than the configuration; they assume an architecture already using multi-path residual streams and ask what to change when scaling stalls. ^[raw/articles/deepseek-mhc之后xhc重新设计16路残差流架构.md:360-365]

1. **Diagnose saturation as a signal-supply problem before adding capacity.** If every write-back is still one shared output vector dressed up with per-path weights, diversify the write-back content before widening the stream count.
2. **Decouple state breadth from update breadth.** Keep many readable states but restrict mixing and write-back to a small active set (here k=4: two fixed streams, two routed), then confirm by ablation that both the dense read and the dynamic selection earn their keep.
3. **Buy diversity from one expensive computation, not N of them.** Reuse a single MLP pass and derive multiple write-back components from cheap causal convolution branches at different scales, then decorrelate them, instead of re-executing the block per stream.
4. **Do not ship stability-critical steps on small-scale evidence.** Gram–Schmidt removal was nearly free at 10B and destabilizing at 18B, so decorrelation and regularization components should be validated at the largest scale you intend to train.
5. **Account for memory traffic, not just FLOPs.** Sparse updates can be arithmetic-cheap and still state-read-expensive; when latency is the goal, optimize the read path (routing reuse, deferred mixing, correction from a cached entry state) and report I/O alongside loss.
6. **State the tested envelope when transferring a result.** The evidence covers N=16/k=4 on DeepSeekMoE-style backbones, so re-fit the scaling law and re-run the ablations before assuming the trend holds elsewhere.

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

