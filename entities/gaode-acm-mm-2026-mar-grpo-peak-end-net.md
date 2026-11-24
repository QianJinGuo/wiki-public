---
title: "ACM MM 2026｜高德2篇论文被收录，覆盖自回归图像生成、强化学习、视频美学评估方向"
created: 2026-07-16
updated: 2026-11-24
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

## 深度分析

### 一、训练不稳定的根因定位：把噪声源精确隔离到扩散头

MAR-GRPO 的起点不是提出一个新 RL 算法，而是一次根因诊断。MAR（Masked Autoregressive）模型由大型 AR Transformer 与轻量级扩散解码器组成，端到端 [[concepts/grpo-policy-optimization-2026|GRPO]] 直接作用于整个模型时训练不稳定，论文将根因精确定位为扩散头在联合训练中引入的极大随机性与梯度噪声 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:11-17]。这个诊断决定了后续全部技术选型的方向：既然噪声来自扩散头，稳定化就不必改动 RL 算法本身，而是围绕"如何让 AR 主干在一个可控、低噪声的优化目标上更新"来设计。相比把强化学习细调整体套在混合架构上再调参，这种先隔离噪声源、再设计优化方案的路径明显更工程化。

### 二、四项关键技术的分层降噪链：从冻结到选择性多轨迹

四项关键技术不是并列清单，而是逐层收紧的降噪结构 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:19-23]：第一层冻结扩散头参数、仅优化 AR 主干，直接移除噪声源并提供稳定优化目标；第二层多轨迹期望估计（MTE）对相同 AR 潜变量采样多条扩散轨迹、以期望值引导优化，进一步压低梯度噪声；第三层基于不确定性的局部优化，估算每个 token 的方差，仅对不确定性最高的前 k% 应用多轨迹期望；第四层一致性感知 token 选择，过滤掉优化方向与生成内容不一致的 token。其中 MTE 与局部优化的组合最值得注意：多轨迹采样本身有额外开销，把它限制在前 k% 高不确定性 token 上，等于在降噪收益与计算成本之间做了显式权衡，而不是全量铺开。最终效果是在 HPSv2 人类偏好评分、提示词遵循、空间结构理解等方面超越端到端 GRPO 基线 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:25]。这一 AR 主干加扩散解码头的混合路线，与 [[entities/a2rd-agentic-autoregressive-diffusion-long-video|A2RD 的 AR+扩散混合架构]] 属于同一技术谱系。

### 三、峰终定律作为归纳偏置：把心理学结论转译成评估架构

Peak-End-Net 的核心动作，是把心理学的峰终定律——人们对一段体验的总体评价取决于显著时刻和结束阶段——从行为科学结论转译为视频美学评估的架构先验 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:29-33]。方法设计与定律形成一一对应：预训练图像美学评价（IAA）头生成帧级美学先验、识别关键时刻，对应定律中的"峰"；美学节奏编码器建模审美质量随时间的连续演化，把离散帧分数串成时间曲线；动态门控融合机制优化整体评价结果并增强分布鲁棒性，对应"终"与整体评分的合成 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:35-39]。这背后的设计判断是：视频美学是随时间演化的量，评估架构必须内建时间结构，而时间结构的形状不必完全从打分数据里学——成熟的行为科学结论可以直接充当归纳偏置。

### 四、两篇论文共享的参数策略：冻结主干、小量更新

两条论文在参数效率上做出了同构选择。MAR-GRPO 冻结扩散头、仅优化 AR 主干 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:20]；Peak-End-Net 骨干网络为冻结的 ViT，仅训练少量参数，仍在 VADB（域内）与 DIVIDE-3K（跨域）双数据集取得 SOTA ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:41]。这指向一个共同判断：在混合架构或跨任务迁移场景中，预训练部分的质量应当被保护而非被联合训练冲刷，可训练容量由"哪里引入不确定性、哪里需要新知识"决定，而不是由参数总量决定。Peak-End-Net 的跨域 SOTA 进一步说明，少量可训练参数配合鲁棒的融合设计（动态门控）同样能换来泛化性。这一"冻结主干、局部更新"的做法在 [[entities/appo-agentic-procedural-policy-optimization-amap-ml-2026|AMAP-ML 同团队的 APPO 工作]] 中也能看到延续，属于该团队的稳定方法论。

## 实践启示

1. **在混合生成架构上做 RL 细调前，先做噪声源隔离**：MAR-GRPO 的训练不稳定根因是扩散头引入的随机性与梯度噪声 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:17]，先冻结随机模块、只优化确定性主干（配合 [[concepts/reinforcement-fine-tuning-rft|强化学习细调]] 的常规流程），比直接调 RL 超参更对症。
2. **用不确定性决定算力分配**：MTE 只对不确定性最高的前 k% token 应用多轨迹优化 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:22]，这种方差驱动的选择性计算可以迁移到任何采样昂贵的训练环节（多轨迹、多候选、多次前向）。
3. **跨粒度迁移用预训练模型当先验，而不是从零训练**：Peak-End-Net 把图像美学评价（IAA）知识迁移到视频美学，用预训练 IAA 头产出帧级先验、再在时间维度上建模演化 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:36-38]。
4. **评估指标设计可以从成熟的行为科学结论借归纳偏置**：峰终定律直接给出了"显著时刻 + 结束阶段"的架构骨架 ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:33]，比让模型从打分数据里隐式学习时间结构更省数据、也更可解释。
5. **冻结主干 + 小量参数 + 鲁棒融合，是兼顾域内与跨域性能的路线**：Peak-End-Net 冻结 ViT 仅训练少量参数仍拿下 VADB 与 DIVIDE-3K 双 SOTA ^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:41]，说明泛化瓶颈常在融合机制而非模型容量。
6. **两个工作均开源了配套代码**（github.com/AMAP-ML 下的 mar-grpo 与 Peak-End-Net）^[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md:15,31]，技术细节可复现验证后再引入自己的管线。

## 来源

- 原文: [[raw/articles/gaode-acm-mm-2026-mar-grpo-peak-end-net.md|ACM MM 2026｜高德2篇论文被收录，覆盖自回归图像生成、强化学习、视频美学评估方向]]
- 原始链接: : https://mp.weixin.qq.com/s/I4g2n7JeM7rmT-HEv0ByNg
