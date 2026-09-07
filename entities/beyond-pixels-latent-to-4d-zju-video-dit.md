---
title: "Beyond Pixels / Latent-to-4D：从视频 latent 直接走向 4D 世界（浙大）"
created: 2026-08-20
updated: 2026-09-07
type: entity
tags: [video-generation, 4d, latent, diT, world-model, multimodal]
sources: [raw/articles/beyond-pixels-latent-to-4d-zju-video-dit]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Beyond Pixels / Latent-to-4D：从视频 latent 直接走向 4D 世界（浙大）

> **背景**：本文基于新智元对 ReLER、CCAI、浙江大学团队论文《Beyond Pixels》的解读（arxiv 2608.10744）。核心洞察：视频生成模型在 latent 空间已学到外观、运动与空间线索，这些线索能否不经过像素、直接转化为动态三维几何？答案是 Latent-to-4D——绕过"生成 latent → RGB 视频 → 重建特征 → 4D 几何"的多余翻译往返。

## 摘要

Beyond Pixels 提出 Latent-to-4D 框架，把视频 DiT 最终去噪的 VAE latent 直接作为显式 4D 预测的输入，通过 Latent-to-4D Alignment and Refinement（L4AR）网络把视频 latent 翻译成结构化 4D latent，再由预训练 4D 解码器输出相机与动态世界空间点图。传统路线是"先生成再重建"（latent→RGB→4D），每次跨过表示边界都可能损失信息，且生成器与重建器训练数据分布不同，生成伪影会被重建模型固化为几何孔洞。Latent-to-4D 的关键洞察是：不同视频 DiT 只要共用同一套 VAE 规范，其最终 latent 就处于同一种表示空间中，从而"DiT 可以不同，但说着同一种 latent 语言"。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]

## 核心要点

- **为什么绕 RGB 是多余的**：视频 DiT 在 latent 空间去噪，VAE Decoder 把 latent 解码成 RGB，再由独立 4D 重建模型读像素恢复几何。这条"翻译往返"（生成 latent→RGB→重建特征→4D）每跨一次表示边界都损失信息，且生成器与重建器见过不同数据，生成视频的伪影会被重建模型"固化"为几何孔洞、破碎表面和不稳定运动。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]
- **shared VAE = 统一 latent 语言**：不同视频 DiT 未必有相同架构/参数量/条件输入，但可能共享同一 VAE。只要 VAE checkpoint、latent 归一化、张量布局、压缩规范和 latent shape 一致，这些 DiT 最终生成的 latent 就处于同一种表示空间——而这个最终 latent 已包含画面内容、主体运动、镜头变化，且位于 RGB 解码之前。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]
- **L4AR 三步跨表示翻译**：(1) 三线性重采样把视频 latent 调整到 4D 解码器时空分辨率 + 可学习 3D 卷积投影到 4D token 特征维度（初始对齐聚合局部运动与外观证据）；(2) 交替使用 Frame Attention（单帧内整理空间结构）与 Global Attention（跨时空 token 交换信息，连接跨帧运动/镜头变化/长距离对应）；(3) 4D Decoder 预测每帧相机、深度与世界空间射线，组合成统一坐标系中的动态点图。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]

## 训练与推理：一次"无缝换源"

训练阶段不需要运行视频 DiT——用冻结的 Wan VAE 编码已有重建视频得到 observed-video latent，用已有相机/几何标注训练 latent-to-4D 下游路径。视频生成器、VAE、预训练 4D 模型原始权重全部冻结，主要更新 latent 对齐模块、基于 LoRA 的轻量时空细化参数、相机与几何预测头。推理阶段只做一次替换：把真实视频编码得到的 latent 换成兼容视频 DiT 生成的最终去噪 latent，后续 L4AR 与 4D Decoder 完全不变，不做测试时适配。文本/图像/姿态/轨迹条件共用同一条路径，因为条件已被视频模型"实现"在最终 latent 中。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]

## 关键数据

- **一个 checkpoint 横跨三种 DiT**：最终训练阶段只用约 1K 条已有重建视频，同一 checkpoint 原样接入 Wan2.1-T2V-14B、Wan2.1-T2V-1.3B、Wan2.2-I2V-A14B（不同规模 Text-to-Video 与 Image-to-Video 模型，共享同一 Wan VAE），切换 DiT 不重新训练、不加条件分支、不做测试时微调。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]
- **same-latent 严格对比**：与 Wan+4RC 基线（latent→RGB→4D 重建）使用完全相同的生成 latent。在 Text4D-200（200 个文本条件案例）与 I4D-200（200 个图像条件案例）评测中，Image-to-4D 测试全部指标排名第一。14B 与 1.3B 两个 Text-to-Video DiT 接入同一 checkpoint 后 DINO-F1 分别为 57.01 与 57.09，结果非常接近——为"共享 VAE latent 可成为统一接口"提供直接证据。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]

## 实践启示

1. **latent 空间可作为跨模型统一接口**：与其在像素层做可组合但误差传播的重建，不如在 VAE latent 层找一个所有生成模型共享的接口。shared-VAE 前提让"训练一次、多 DiT 通用"成为可能。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]
2. **冻结 + 轻量适配的规模化路径**：训练阶段不跑视频 DiT、只更新 LoRA 时空细化 + 预测头，把昂贵的 4D 几何监督需求压到 1K 条数据，是数据稀缺场景下的高效范式。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]
3. **生成与理解的解耦**：视频模型负责"想象内容和运动"，Latent-to-4D 提供统一 4D 出口——这种关注点分离让生成能力（多 DiT）与几何理解（单一 4D 解码）各自演进。^[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit.md]

## 相关实体

- 视频生成模型
- [[entities/4d-wam-world-action-model-3d-trajectory-alignment-2026|4D WAM 世界动作模型]]
- [[entities/eccv26-为视频虚拟试衣解锁自由视角tryoncrafter玩转4d试衣代理|TryOnCrafter 4D 试衣]]
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0 世界模型]]

→ [[raw/articles/beyond-pixels-latent-to-4d-zju-video-dit|原文存档]]
