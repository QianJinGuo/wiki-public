---
title: "ACM MM 2026｜高德2篇论文被收录，覆盖自回归图像生成、强化学习、视频美学评估方向"
created: 2026-07-16
updated: 2026-09-07
type: entity
tags: ['auto-harvested']
sources: [raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md|原文存档]]

- 论文：https://arxiv.org/abs/2604.06966 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md]

## 摘要

高德技术宣布 2 篇论文被 ACM MM 2026 收录。论文一 MAR-GRPO 解决端到端 GRPO 应用于 MAR（Masked Autoregressive，AR Transformer + 轻量级扩散解码器）图像生成时的训练不稳定问题——根因是扩散头在联合训练中引入极大随机性和梯度噪声；其四项关键技术为：冻结扩散头参数仅优化 AR 主干、多轨迹期望估计（MTE，相同 AR 潜变量采样多条扩散轨迹计算期望以降低梯度噪声）、基于不确定性的局部优化（仅对不确定性最高的前 k% token 应用多轨迹优化）、一致性感知 token 选择（过滤优化方向与生成内容不一致的 token），在 HPSv2 人类偏好评分、提示词遵循、空间结构理解上超越端到端 GRPO 基线 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md]

论文二 Peak-End-Net 把心理学的峰终定律（人们对一段体验的总体评价取决于显著时刻和结束阶段）引入视频美学评估：用预训练图像美学评价（IAA）头生成帧级美学先验识别关键时刻，美学节奏编码器建模审美质量随时间的连续演化，动态门控融合机制增强分布鲁棒性；骨干网络为冻结的 ViT 仅训练少量参数，在 VADB（域内）和 DIVIDE-3K（跨域）上取得 SOTA。两篇论文代码均开源在 github.com/AMAP-ML ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md]

## 关键要点

- MAR-GRPO（arXiv:2604.06966）：MAR 模型 = 大型 AR Transformer + 轻量级扩散解码器；端到端 GRPO 训练不稳定的根因是扩散头的随机性和梯度噪声。
- MTE 多轨迹期望估计：对相同 AR 潜变量采样多条扩散轨迹求期望，引导优化方向。
- Peak-End-Net（arXiv:2607.13941）：峰终定律启发的视频美学评估框架，IAA 先验识别关键时刻 + 美学节奏编码器 + 动态门控融合。
- Peak-End-Net 骨干为冻结 ViT，仅训练少量参数，VADB 和 DIVIDE-3K 双 SOTA。
- 代码开源：github.com/AMAP-ML/mar-grpo 与 github.com/AMAP-ML/Peak-End-Net。

## 来源

- 原文: [[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md|ACM MM 2026｜高德2篇论文被收录，覆盖自回归图像生成、强化学习、视频美学评估方向]]
- 原始链接: : https://mp.weixin.qq.com/s/I4g2n7JeM7rmT-HEv0ByNg
