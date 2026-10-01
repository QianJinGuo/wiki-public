---
title: 淘宝闪购爆品团精排 Scaling Up 迭代实践
created: 2026-07-16
updated: 2026-10-01
type: entity
tags: [recommendation, ranking, scaling-law, rankmixer, deep-learning, ctr, moe, alibaba]
status: verified
confidence: 0.9
provenance_state: extracted
sources: [raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> 阿里云开发者团队李伟康撰写的淘宝闪购爆品团频道排序模型升级实践，核心是将 DLRM 架构的碎片化模块堆叠重构为 Token-Based RankMixer 统一主干，经过三期迭代从 85M 参数扩展到 243M 参数，取得稳定线上收益。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

## 背景与动机

淘宝闪购爆品团业务的旧排序模型长期采用"堆模块"模式：Embedding 和 Sequence 层之后串联 EPNet、多层 PLE、Task Specific Net、Task Bias Net、ResFlow Tower 等模块。每个模块各自解决过局部问题，但整体呈现 **结构碎片化、计算冗余、维护成本高、扩展性差** 四大问题。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

重构的核心思路不是继续加模块，而是将主干从传统 DLRM 范式切换为 **Token-Based RankMixer** 架构，用统一的 Token-Mixing 主干自动学习高阶组合。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

## 架构设计

新模型保留 Embedding 和 Sequence 层，将序列和非序列特征聚合后 concat，切割为指定维度的 Token，进入多层 RankMixer，最后接 MMCN Task Tower 分别预测 CTR、Item CTCVR 和 Shop CTCVR。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

每个 RankMixer Block 由两个核心组件构成：^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

- **Token Mixer** — 跨 Token 信息混合
- **PerToken FFN** — Token 内部非线性变换

## 结构消融关键结论

### 负采样

随机负采样不适合当前样本分布，**保留全量负样本 + Loss 加权** 更稳，AUC +0.1%。随机采样会丢失 Hard Negatives、破坏概率校准。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

### 多任务学习

ESMM 约束虽带来 CTCVR AUC +0.2% 但损失 CTR AUC -0.4%。**GradNorm** 通过动态 Loss 权重平衡多任务学习节奏，CTR AUC +0.7%，不改变模型结构，但训练 GFLOPs 约翻倍。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

### 序列层

HSTU (AUC -0.44%) 和 STCA (AUC -0.32%) 均负收益，**Gated Attention** 在 MHA/ETA 后增加 Gating 机制 AUC +0.06%，是唯一正收益方案。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

### Tokenization

四种方案对比：Auto-Split 最差，**Pad-Split**（zero padding 后切割）持平 Group-wise 但实现更简单，差异在万分位。Token 粒度：16×320 优于 32×160，AUC +0.14%。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

### RankMixer 层数

2 层 baseline → 4 层 AUC +0.21%（显著）→ 6 层 +0.12%（边际递减）。当前 PostNorm 下 4 层是最优选择。有趣发现：丢失 Mixup 后 Add & Norm 反而 AUC +0.2%，补回后下降 — 残差对齐与 Mixup 语义重组不兼容。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

### Dense FFN vs Sparse MoE

- **SwiGLU D→4/3D** 零成本替换（同 76M/1.7GFLOPs），AUC +0.07%
- **AFFN v2** 性价比最高，额外 19M/0.34GFLOPs，AUC +0.13%
- **Sparse MoE** 在 16 Token 配置下全线负收益，未超过 Dense，Dense 仍是当前主线^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

### Task Tower

输入方式：Mean Pooling < TSAP < **Flat Concat**（保留完整 Token 表达）。反直觉发现：TSAP 参数量更小但 RT 增加 5ms，Flat 更适合 GPU 计算。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

Tower 结构：MLP baseline → ResFlow MLP (AUC +0.07%) → **MMCN** (AUC +0.32%)，4-head 交叉结构是收益最明显的方案。MMCN 维度从 [1024,512,256,128] 扩到 [4096,2048,1024,512] 时 AUC +0.21%，继续扩到 5 层训练 NaN。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

## 三期迭代实验

| 阶段 | 时间 | 配置 | 参数量 | GFLOPs | 核心收益 |
|------|------|------|--------|--------|----------|
| 一期 | 2026-04-09 | 4 层 RankMixer + 3×MMCN, Token 32×160 | 85M→107M | 2.82→2.26 | CTCVR AUC +0.6%, 页面导购率 +0.97%, 人均G +1.14% |
| 二期 | 2026-05-07 | 2 层 RankMixer, Token 16×320, GradNorm | ~107M | 2.26→4.78训练 | CTR AUC +0.7%, 吞吐量 +12%, 人均订单 +0.32% |
| 三期 | 2026-05-21 | 4 层 RankMixer, Tower 扩维 | 107M→243M | 2.26→5.18 | CTCVR AUC +0.37%, 导购率 +0.44% |

三期模型迁移至超抢手业务：CTCVR AUC +0.66%, CTR AUC +1.6%, 页面导购率 +1.09%, 人均G +1.16%。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

## 关键经验

- Scaling Up 不是简单堆参数，而是让主干、计算形态、Task Tower 都适合放大
- Sparse MoE 不是加上就一定涨 — 推荐场景下 Token 数量、位置语义、路由稳定性都影响效果
- FLOPs/参数量与线上 RT/吞吐量不总是强正相关（TSAP 现象）
- 旧模型的问题通常是**整体结构碎片化**而非单个模块失效

## 后续方向

短期：探索更优 PerToken FFN、推理侧优化（算子融合/量化）。中长期：跟进 TokenMixer-Large/UniMixer/TokenFormer，在更大规模下重探 Sparse MoE。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

## 深度分析

**重构决策的本质：范式替换优于局部修补。** 旧模型的 EPNet + PLE + Task Specific Net + Task Bias Net + ResFlow Tower 是典型的"问题驱动补丁链"——每个模块在加入时都有局部收益，但组合后互相干扰、联合调优困难、无法通过加层加维继续放大。RankMixer 的价值不在于单点 AUC 更高，而在于提供了一个结构上可扩展（scalable）的主干：Token-Mixing 范式让"增加层数/扩大维度"重新成为可用的升级手段，这正是 DLRM 类异构堆叠结构最缺的性质。NLP 领域 Scaling Law 的前提（大规模参数 + 稠密计算架构）被有意识地移植到了推荐排序场景。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

**三期迭代呈现清晰的节奏控制：先验证主干、再优化效率、最后放大规模。** 一期（85M→107M）参数增加但推理 GFLOPs 从 2.82 降至 2.26，证明新主干"更贵但更稠密"——收益与算力不是简单换算关系；二期刻意缩小到 2 层、Token 调整为 16×320 并引入 GradNorm，用训练算力（GFLOPs 翻倍）换取 CTR AUC +0.7% 和吞吐量 +12%；三期才回到 Scaling 主线扩到 243M。这种"验证—效率—放大"的交替节奏避免了直接 all-in 大模型的常见陷阱：在主干未验证前盲目扩参数。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

**Scaling 的收益来源可以拆解到具体结构，而非笼统的"参数变多"。** 消融数据表明收益高度依赖结构选择：Token 粒度从 32×160 改为 16×320 的 AUC +0.14% 大概率来自 PerToken-FFN 的参数/计算量提升；MMCN Tower 扩维到接近输入维度 5120 时仍有 +0.21%，说明 Flat Concat 保留完整 Token 表达后 Tower 仍有扩维空间——Flat 输入不只是"效果更好"，它是 Tower Scaling 的前置条件。相反，Mean Pooling 在最后一层 Token 未自然趋同的情况下直接平均会损失信息，堵死了后续放大路径。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

**Sparse MoE 的失败是一个有诊断价值的负结果。** 四种 Router/Experts 组合（Shared/PerToken Router × Shared/PerToken Experts）在 16 Token 设定下均未超过 Dense，最优配置（E=16、Top2、H=4/3D）仍不如同资源的 SwiGLU。这提示 MoE 的收益前提是 Token 数量和样本规模足够大、路由足够稳定——16 Token 的推荐场景专家容量太小，路由不稳定且难以观察到 Scaling Law。作者据此把 Sparse MoE 留作"规模扩大后重探"的中长期方向，而不是在当前规模下强行推进，这个决策边界本身比结论更有参考价值。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

**两个反直觉现象揭示了架构间的一致性约束比单点优化更重要。** 其一，Mixup 后丢失 Add & Norm 反而 AUC +0.2%，补回后下降——Mixup 重组跨 Token 语义后，混合表示与原位置残差不再对齐，直接相加会稀释混合信号甚至引入噪声，说明残差连接并非"永远正确"的通用组件，其有效性依赖语义对齐前提。其二，序列层重替换（HSTU -0.44%、STCA -0.32%）全线失败，轻量改造（Gated Attention +0.06%）才是唯一正收益——在已有两段式结构上，新结构并非越先进越好。这两个现象共同指向：消融实验的意义在于发现"哪些默认假设在当前架构下不成立"，而不是寻找普适最优解。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

**ESMM 与 GradNorm 的取舍反映了多任务学习的核心矛盾在梯度流而非损失函数。** ESMM 的 PCTCVR = PCTR × PCVR 约束让 CTCVR 梯度反向传播进 CTR Tower，使 CTR 特征表示被 CTCVR 目标污染（CTCVR +0.2% 但 CTR -0.4%）；GradNorm 不改结构、只调权重，通过动态平衡任务学习速率拿到 CTR +0.7%。代价是训练 GFLOPs 翻倍——多任务收益的货币是训练算力，这与二期"用训练换效果"的整体策略一致。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

## 实践启示

- **排序模型重构优先评估"可扩展性"而非单点收益。** 判断一个新主干是否值得换，关键问题是"它能否通过加层加维持续放大"，而不是离线 AUC 一次对比谁高。碎片化堆叠的旧结构即使短期收益尚可，长期Scaling 也会被维护成本和结构瓶颈锁死。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]
- **负样本处理：保留信息优于删减信息。** 正负样本悬殊时，随机负采样会丢 Hard Negatives 并破坏概率校准；全量负样本 + Loss 加权（正加权、负降权）以极低成本拿到 AUC +0.1%。对以 AUC 为核心的排序任务，负样本是排序边界的信息来源而非噪声。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]
- **Tokenization 选简单方案是安全的。** Pad-Split 零信息损失、无额外投影矩阵，效果持平依赖人工划分的 Group-wise（差异在万分位）。RankMixer 的 Token-Mixing 本身会重组语义，初始 Token 边界不是关键——前提是你用的主干具备跨 Token 混合能力。适当增大 Token Dim、减少 Token 数（16×320 优于 32×160）还附带了 PerToken-FFN 容量提升的收益。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]
- **Dense 改造优先级：零成本结构替换 > 换高效结构 > 单纯扩容。** SwiGLU D→4/3D 零成本 +0.07% 优于 FFN D→4D 花 13M 参数换 +0.06%；有预算时 AFFN v2（19M/0.34GFLOPs 换 +0.13%）性价比最高。Sparse MoE 在 16 Token、样本量有限的设定下不要上——先用 Dense 打满，规模扩大后再重评。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]
- **Task Tower 输入用 Flat Concat，为后续 Tower Scaling 留路。** Mean Pooling 损失信息，TSAP 参数少但 RT 反增 5ms（吞吐 100→80），Flat + 大 Tower 更契合 GPU 计算形态。Tower 扩维有明确天花板信号：接近输入维度时仍有效（[4096,2048,1024,512] +0.21%），加到 5 层训练 NaN 时应立即停止。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]
- **FLOPs/参数量与线上 RT/吞吐量不总是强正相关，容量规划必须实测。** TSAP + 小 Tower 参数更少但更慢的案例说明，GPU 上的计算形态（算子粒度、并行度）比理论计算量更决定 RT。任何"参数更少所以更快"的推断都需要吞吐量实测验证。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]
- **Scaling Up 采用交替节奏而非一路放大。** 先小规模验证主干（一期），再回头优化效率和多任务平衡（二期），最后放大规模（三期），并在新业务（超抢手）上验证迁移性——243M 模型在超抢手拿到 CTR AUC +1.6% 说明收益可跨场景复用。每期都同时盯离线 AUC、吞吐量和大盘/页面侧业务指标，避免单一指标优化。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]
- **PostNorm 架构下主干深度有实际上限。** 4 层收益显著（+0.21%）、6 层边际递减（+0.12% 且拉长周期后不显著），当前 PostNorm 对过深结构不友好。若要继续加深，换 PreNorm 或类似归一化策略是前置条件，而非直接堆层。^[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16.md]

## 关联条目

- [[entities/huawei-fuxi-recommendation-system-ascend-npu-scaling-law|推荐系统进入大模型时刻：昇腾 NPU 如何支撑千亿级生成式推荐落地]] — 华为 Fuxi 推荐系统 Scaling Law 实践，与本篇淘宝爆品团形成对比视角（生成式推荐 vs Token-Based DLRM 替换）
- [[entities/onereason-kuaishou-reasoning-recommender-system|OneReason：快手 Reasoning Recommender System 实践]] — 快手的推理型推荐系统，与淘宝 Token-Based RankMixer 分别探索推荐模型的不同演进方向

## 退出

→ [[raw/articles/taobao-flash-sale-product-group-ranking-scaling-up-2026-07-16|原文存档]]
