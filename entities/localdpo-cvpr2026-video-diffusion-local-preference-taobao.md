---
title: "LocalDPO — 面向视频扩散模型的局部细节偏好优化方法 (CVPR 2026)"
type: entity
created: 2026-07-05
updated: 2026-09-05
tags: [diffusion, video-generation, dpo, preference-optimization, cvpr2026, multimodal, generative-ai, fine-tuning]
rating: v8c8
sources:
  - raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao
---

# LocalDPO — 面向视频扩散模型的局部细节偏好优化方法 (CVPR 2026)

LocalDPO 是淘天音视频技术团队联合外部合作伙伴提出的面向视频扩散模型的细粒度偏好优化方法，入选 CVPR 2026。该方法以高质量真实视频为正样本，通过局部时空退化自动构造负样本，结合区域感知 DPO 损失，在无需外部打分模型或人工标注的情况下，显著提升了视频生成模型的视觉质量、时序一致性和人类主观偏好。^[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao.md]

## 核心动机

现有视频 DPO 方法存在三大问题：一是依赖多次采样和人工标注，成本高昂；二是基于全局打分的监督信号容易产生歧义；三是忽略了人物五官、手部结构、局部纹理等对主观体验影响更大的局部偏好信号。^[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao.md]

## 方法设计

LocalDPO 的核心创新在于偏好对构造方式和优化目标设计两个层面：^[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao.md]


### 局部偏好对自动构造

- **正样本**：直接使用真实高质量视频（63K 高质量视频片段，由 VLM 生成结构化文本描述）
- **负样本**：通过对真实视频的局部时空区域施加可控退化自动构造。采用随机贝塞尔曲线生成 3D 时空掩码，基于冻结预训练 VDM 进行局部重绘式退化，仅在掩码区域执行恢复，非掩码区域保持原始内容不变^[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao.md]

### 区域感知偏好优化目标

提出区域感知 DPO 损失（Region-aware DPO Loss），仅在局部退化区域上计算偏好误差。同时结合标准 DPO 损失和 SFT 损失构建混合训练目标，在提升局部细节修复能力的同时保持全局生成稳定性。^[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao.md]

## 实验验证

在 CogVideoX-2B、CogVideoX-5B 和 Wan2.1-1.3B 等多个主流视频扩散模型上进行了系统实验，与 SFT、Vanilla DPO、DenseDPO 等方法比较：^[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao.md]


- **定量评估**：在 VBench、VideoJAM 等基准上多项指标显著领先，尤其在视觉质量相关指标上提升突出
- **主观评测**：20 位评测者在视觉质量、运动质量、文本对齐和综合质量四个维度上均显著更优
- **定性效果**：局部纹理更丰富、画面更清晰、伪影更少、时序更稳定、语义对齐更好^[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao.md]

## 意义

LocalDPO 为视频生成模型的偏好对齐提供了一种高效、稳定且细粒度的新思路，无需外部打分模型或人工标注，即可实现对视频局部细节的高效偏好对齐。论文代码已开源。^[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao.md]


> [!note] 领域特化说明
> 与现有 RLHF/DPO 对齐中通用的偏好优化方法不同，LocalDPO 专注于视频扩散模型局部细节的优化，是一种领域特化的 DPO 变体。其"真实视频做正样本+局部退化构造负样本"的策略在图像/文本等模态中不可直接复用。

## 参考

- 论文：https://arxiv.org/pdf/2601.04068
- 代码：https://github.com/1170300714/Local-DPO
- → [[raw/articles/localdpo-cvpr2026-video-diffusion-local-preference-taobao|原文存档]]

## 相关实体

- "扩散模型架构"
- "视频生成模型"
- "RLHF/DPO/GRPO 对齐"
- [[entities/diffusion-model-consistency-framework-2026-survey]]
- [[entities/a2rd-agentic-autoregressive-diffusion-long-video]]
