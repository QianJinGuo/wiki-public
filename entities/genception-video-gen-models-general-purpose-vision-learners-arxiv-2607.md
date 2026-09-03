---
title: "GenCeption — 视频生成模型作为通用视觉学习器"
created: 2026-07-14
updated: 2026-07-14
type: entity
tags: [vision, multimodal, video-generation, foundation-model, diffusion, generalist, arxiv]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607]
---

# GenCeption — 视频生成模型作为通用视觉学习器

**GenCeption** 是一篇 2026 年 7 月的 arXiv 论文（2607.09024），提出以文生视频（text-to-video）生成为视觉基础模型的预训练范式。论文将预训练的视频生成扩散模型转化为统一的、前馈的通用视觉感知模型——GenCeption，在深度估计、表面法线预测、相机姿态估计、分割和 3D 关键点等任务上达到或超越专门模型（SOTA）。Kaiming He 为合作者之一。^[raw/articles/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607.md]

## 核心论点：视频生成是视觉的"next-token prediction"时刻

正如 NLP 中的 next-token prediction 将任务特定模型进化为通用语言基础模型（如 GPT 系列），论文主张 **大规模文生视频生成可以作为计算机视觉的等价催化剂**——提供视觉通用智能所需的时空先验、视觉-语言对齐和可扩展性。^[raw/articles/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607.md]

这与 [[entities/elf-embedded-language-flows-hekaiming|ELF (Embedded Language Flows)]] 共享 Kaiming He 在通用表征学习方面的研究脉络——ELF 提出语言流作为通用接口，GenCeption 则探索视频生成作为视觉预训练的接口。

## 技术架构

GenCeption 的架构包含两个阶段：^[raw/articles/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607.md]

1. **预训练阶段**：利用视频生成扩散模型捕获丰富的时空世界先验和原生视觉-语言对齐能力
2. **后训练阶段**：通过多任务后训练将多步生成骨干转化为单步前馈模型，主要在合成数据上微调

稠密视觉任务（深度、法线、相机姿态）在 RGB 环境空间中统一，可在潜在空间中高效施加监督。稀疏视觉任务（关键点、分割）通过向 Diffusion Transformer (DiT) 添加可学习 token 作为额外输入来实现。

## 关键发现

### SOTA 性能
GenCeption 在以下任务上达到或超越专门模型：^[raw/articles/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607.md]
- 深度估计 → 超越 DepthAnything3
- 表面法线预测 → 超越 Sapiens, David
- 相机姿态估计 → 超越 D4RT, VGGT-Omega
- 指代表达分割 → 超越 SAM3, Genmo, Lotus-2
- 2D/3D 关键点预测 → 超越 D4RT

### 极端数据效率
视频生成预训练骨干在同等微调数据下胜过替代预训练范式（V-JEPA, VideoMAE V2）。最引人注目的：仅用 **D4RT 和 VGGT-Omega 的 1/7 到 1/500 训练数据**即可达到可比性能。^[raw/articles/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607.md]

### 涌现行为
仅在合成人体视频上训练的模型能泛化到真实世界 footage 和分布外物体类别（动物、机器人）——展现出初级的 sim-to-real 迁移能力和模型缩放特性。^[raw/articles/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607.md]

## 与现有生态的关系

- [[entities/diffusion-model-consistency-framework-2026-survey|Diffusion Model 一致性框架]] — GenCeption 将多步扩散转化为单步前馈，是 consistency 方向的重要应用
- [[entities/elf-embedded-language-flows-hekaiming|ELF: Embedded Language Flows]] — 同一位研究者（Kaiming He）在通用接口方面的不同探索：语言流 vs 视频生成
- [[entities/ard-agentic-autoregressive-diffusion-for-long-video-consistency|ARD: Agentic Autoregressive Diffusion]] — 视频生成的另一个扩散方向
- [[entities/video-agent-paradigm-compute-talent-flywheel-ethan-he-20260606|Video Agent 范式]] — 视频作为 AI 基础设施的不同视角
- [[entities/nvidia-cosmos-fine-tuning-robot-video-generation|NVIDIA Cosmos]] — 视频生成作为机器人 foundation model 训练数据的另一路径

## 引用

- arXiv: [2607.09024](https://arxiv.org/abs/2607.09024)
- Project page: [genception.github.io](https://genception.github.io/)

→ [[raw/articles/genception-video-gen-models-general-purpose-vision-learners-arxiv-2607|原文存档]]
