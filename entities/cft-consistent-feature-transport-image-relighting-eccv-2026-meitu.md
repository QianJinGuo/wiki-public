---
title: "CFT：一致特征传输的人像重打光（美图影像研究院 ECCV 2026）"
created: 2026-08-22
updated: 2026-09-07
type: entity
tags: [vision, diffusion, image-relighting, eccv, rectified-flow, image-generation, multimodal, portrait]
sources: [raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# CFT：一致特征传输的人像重打光（美图影像研究院 ECCV 2026）

## 概述

美图影像研究院（MT Lab）提出 Consistent Feature Transport（CFT，一致特征传输），将人像重打光重新表述为**光照一致的特征传输问题**，在 Rectified Flow 框架上显式学习源图与目标图分布之间的光照变换，而非泛泛的图像翻译。成果被计算机视觉顶级会议 ECCV 2026 录用，论文题为《Consistent Feature Transport for Image Relighting》。^[raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu.md]

## 背景：三类主流方法的短板

现有基于扩散模型的人像重打光主流方法各有不足：^[raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu.md]

- **控制信号驱动（Control-based）** — 可控性有所提升，但结果高度依赖控制信号的准确性与完整性
- **本征分解驱动（Decomposition-based）** — 复杂光照下容易出错，误差传导至最终图像，导致阴影不一致、颜色偏移或细节丢失
- **直接图到图翻译（Image-to-image）** — 未显式建模「光照专属」的特征位移，光照编辑精度与表达力不足

此外，公开人像重打光数据集规模有限、多在受控环境采集，缺乏多色温、空间变化阴影、非均匀照明等复杂光效，难以支撑真实场景下的可控重打光学习。^[raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu.md]

## 方法：CFT 三部分损失

CFT 在噪声、源图、目标图三个分布之间建立联合建模，训练目标由三部分组成：^[raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu.md]

- **L₁ 噪声→目标图** — 给定源图 (x_src)、重打光目标图 (x_tgt) 及文本条件 (c_tgt)，学习从先验噪声到目标 latent 的速度场，完成条件生成
- **L₂ 噪声→源图** — 在同一先验噪声下，以中性条件 (c_src) 重建源图分布，强化对非光照内容（身份、几何、场景结构）的保持
- **L₃ 源图→目标图：光照一致的特征传输（CFT 核心）** — 借鉴无反演编辑方法，利用 Rectified Flow 的线性性质，通过平行四边形法则构造源到目标的直接传输路径：`z_t_direct = z_src + z_t_tgt − z_t_src`，对应速度场 `v_t_direct = v_θ(z_t_tgt,t,x_src,c_tgt) − v_θ(z_t_src,t,x_src,c_src)`

L₃ 的监督使用**身份与场景内容不同、但光照变换相同**的图像对作为真值。若监督来自同一对图像，模型易拟合样本特有的内容差异，把光照变化与非光照变化混在一起；换用「同光照变换、不同内容」的配对，可让模型学习可跨实例复用的光照特征传输，在生成稳定性与光照传输监督间取得平衡。^[raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu.md]

## 数据集与实验结果

团队构建了覆盖室内、室外等主流场景的大规模人像重打光数据集，覆盖 14 类光照效果类别，强调空间结构化照明、多色温与复杂阴影，弥补现有 portrait relighting 数据在复杂光效上的不足。^[raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu.md]

实验显示 CFT 在像素保真、感知相似度、分布真实感与光照专用指标上均优于现有方案；定性结果在多光源交互、空间结构化照明、色温大幅偏移等困难场景下的一致性与稳定性有明显优势。30 名用户对 40 组随机样本打分显示，Flux-Kontext + CFT 在光照质量（IQ）、内容一致性（CC）、物理合理性（PP）、图像美学（IA）上表现最佳。消融实验表明，单独增加 L₂ 不能提升重打光质量，但与 L₃ 联用时提供额外结构保持，完整 CFT 效果最优。^[raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu.md]

## 相关实体

- 扩散模型架构
- 视觉语言模型
- [[entities/a2rd-agentic-autoregressive-diffusion-long-video|A2RD 扩散长视频]]
- [[entities/cvpr-2026-dgaf-vsr-video-super-resolution-diffusion-taobao|CVPR 2026 DGAF-VSR 视频超分]]

→ [[raw/articles/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu|原文存档]]
