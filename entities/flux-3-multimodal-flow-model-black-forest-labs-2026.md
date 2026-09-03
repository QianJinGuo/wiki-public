---
title: "FLUX 3 — Black Forest Labs 多模态流模型"
created: 2026-07-24
updated: 2026-08-29
type: entity
tags: [diffusion, multimodal, video-generation, image-generation, flow-model, world-model, black-forest-labs]
sources: [raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026]
confidence: 0.7
---

# FLUX 3 — Black Forest Labs 多模态流模型

> **Background**：本文档基于 Black Forest Labs 官方博客和早期评测数据建立。FLUX 3 是 2026 年 7 月发布的多模态基础模型，统一学习图像、视频和音频。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

## 概述

FLUX 3 是 Black Forest Labs 发布的第三代多模态基础模型，统一学习图像、视频和音频。基于 Self-Flow 方法构建，高效对齐多模态生成与理解。核心理念是"从所有模态同时学习"，通过模态间的相互约束来理解底层现实——"图像捕捉空间结构，视频恢复时间维度，音频揭示因果关联，语言连接目标与抽象"。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

## 架构创新

### Self-Flow 对齐方法

FLUX 3 基于 Self-Flow 方法构建，这是一种高效的对齐多模态生成与理解的方法。在此方法上，团队显著扩展了计算和训练数据规模，同时训练视频、图像和音频。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

### 统一多模态骨干网络

FLUX 3 的架构将多个模态整合在同一个骨干网络中。设计原则是"没有任何单一模态能提供完整描述"，每个模态都是同一底层现实的投影，通过联合学习它们的相互约束来获得更丰富的表示。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

## 核心能力

### 视频生成（含原生音频）

- **Text-to-video**：文本直接生成视频+音频，最长 20 秒
- **Image-to-video**：从起始帧继续或作为视觉参考
- **Video-to-video**：保留核心元素转换场景
- **Keyframe-to-video**：关键帧之间可控过渡
- **Agentic chaining**：多镜头序列链接
- 多语言对白、广泛视觉风格、排版生成

早期评测中最强胜率对比 Runway Gen-4.5 达 77%、Luma Ray 3.2 达 93%。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

### 图像生成

相比前代 FLUX 1/2（参见 [[entities/meta-agent-image-generation-model|图像生成模型]]），在复杂 prompt 处理、多语言文字渲染方面显著提升。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

### Action 预测

FLUX 3 将动作预测直接集成到模型中，使用预训练的视频骨干网络作为动力感知基础，微调专用动作模型。与 mimic robotics 合作开发 FLUX-mimic 视频动作模型，正在奥迪工厂进行灵巧操作测试。这使其进入了 [[entities/loopwm-looped-world-models|world model]] 和具身智能领域。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

## 与同类工作的对比

FLUX 3 的定位是统一视频/图像/音频/动作的多模态基础模型，与 [[entities/nvidia-cosmos-fine-tuning-robot-video-generation|NVIDIA Cosmos]]（侧重于机器人视频生成和物理仿真）和 [[entities/gemini-embedding-2-multimodal-unified-vector-hyman|Google 的多模态 embedding]] 不同，前者更侧重生成质量和创意表达，同时保留了物理世界建模能力（通过 action 预测分支）。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

## 发布计划

- **FLUX 3 Video**：API 和私有权重访问（首批开放）
- **FLUX-mimic / FLUX 3 Action**：研究和商业合作伙伴
- **FLUX 3 Image**：API 和私有权重访问
- **FLUX 3 Dev**：开源权重多模态骨干网络

## 意义与评估

FLUX 3 代表了视频生成领域向**统一多模态基础模型**方向的重要进展。其"感知→预测→行动"的路线图指向 real-world visual intelligence。Self-Flow 方法让多模态生成和理解在同一架构中对齐，是一个值得关注的技术方向。^[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026.md]

→ [[raw/articles/flux-3-multimodal-flow-model-black-forest-labs-2026|原文存档]]
