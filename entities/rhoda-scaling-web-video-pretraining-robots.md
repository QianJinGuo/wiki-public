---
title: "Does Scaling Web-Video Pre-training Help Real Robots Do Real Work?"
created: 2026-09-22
updated: 2026-10-06
type: entity
tags: [robotics, pretraining, scaling, embodied-ai]
sources: [raw/articles/rhoda-scaling-web-video-pretraining-robots]
confidence: 0.7
---

# Does Scaling Web-Video Pre-training Help Real Robots Do Real Work?

## 摘要

Rhoda AI 严格实证：scale model size + web-video pretraining compute 对真实工业操作任务的影响——打破 scaling web-video pretraining 必然更好的未检验信念，200+ 小时真实机器人评估^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]

## 核心要点

- v*c=56（value=8, confidence=7, stars=4），newsletter ingest 2026-09-22
- 详细分析见原文存档

## 深度分析

### 从二元对比到剂量-响应曲线

这篇工作的方法论突破在于把长期停留在口号层面的信念变成可测量的剂量-响应关系。此前 robot scaling 研究（GEN-0、Dyna-2、EgoScale）大多只做"有 pretraining vs 从头训练"的二元对比，回答不了"多 scale 一档值多少"。Rhoda 用 7 个 checkpoint（XS/S/L，加上 M 在 0.08×/0.18×/0.37×/1× 四档 compute）画出完整曲线：XS→S→L 的 at-speed completion rate 从 3.7% 涨到 65.0% 再到 84.7%，M 系列随 compute 从 57.8% 爬到 75.3%。曲线最有信息量的地方恰恰是"不平滑"——M·1× 与 M·0.37× 几乎打平，compute 的边际回报在高段明显递减，这是二元对比永远看不到的结构^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。

### DINO FD：视频领域的"pre-training loss"

LLM 时代 scaling law 成立的前提是 pre-training loss 与下游性能可预测挂钩（Kaplan、Gadre 路线）；视频模型此前缺少对应的 checkpoint 级指标——post-training 之前无从判断好坏。Rhoda 提出 DINO FD（Fréchet distance）：让模型预测 held-out web video 的续帧，用固定 DINOv2 encoder 嵌入预测帧与真值帧，度量两个特征分布的距离。关键发现是这个纯 web-video 指标与真机任务性能在全部 7 个 checkpoint 上单调相关，等于给视频 pre-training 装上了类似 pre-training loss 的仪表盘——以后可以在昂贵的 post-training 和真机部署之前，仅凭 held-out video 预测质量完成 checkpoint 筛选^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。

### General web video 与 manipulation data 的鸿沟

实验设定里藏着更尖锐的问题：DVA 的 pre-training 数据是 video-only 的 general web video，不含动作、不挑 egocentric 或机器人视角，而已发表的机器人 scaling 工作预训练的都是带动作标注的 manipulation 数据。中间的桥是架构解耦：causal video model 只负责预测未来帧，一个在多机器人数据集上训练一次、全程冻结的 inverse dynamics model 把帧翻译成动作。这让"web-video 预训练的 scale"成为唯一变量，归因干净。结论因此比以往更强：它证明的不是 manipulation 数据的价值，而是视频世界模型本身的可迁移性，与 [[concepts/world-models]] 和 [[concepts/scaling-laws]] 的路线在具身领域汇合^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。

### 200+ 小时真机评估：把 evaluation protocol 当作一等结果

作者明确把评估协议本身列为工作成果。物理评估会以与 policy 无关的方式失败（机械卡顿、环境扰动），少样本 trial 的成败几乎不携带统计信息。他们的应对是：按客户要求的更严格版本定义 at-speed completion rate、每个条件跑上百次 trial（七个条件合计 750+ 次）、报告 95% credible interval，总投入超 200 小时真机时间。对照 XS 的 3.7%（1/27）可知，没有这个协议规模，"XS 彻底失败 vs 偶尔成功"根本无法区分；L 档 84.7%（76.8%–90.2%）的可信正是这样买来的——真机 scaling 研究的成本大头不在训练而在评估^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。

## 实践启示

1. **用 checkpoint 级指标提前止损**：DINO FD 与真机性能全程单调相关，post-training 之前就能用 held-out video 预测质量淘汰差 checkpoint，省掉最贵的真机环节^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。
2. **机器人数据越贵，越该把钱花在 pre-training compute 上**：compute 增益在每档 post-training 数据量下都成立，且机器人 demo 越稀缺回报最大——demo 采集成本高的团队优先扩 pre-training 而非扩采数据^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。
3. **警惕 compute 高段的边际递减**：M·0.37× 与 M·1× 完成率几乎持平（74.5% vs 75.3%），盲目拉满预训练 compute 不划算，应沿剂量-响应曲线找拐点^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。
4. **解耦视频预测与动作生成**：固定 inverse dynamics model + 只 scale video model 的设计让归因干净，也说明动作头不必跟着预训练一起 scale，可跨机器人数据集复用一次训练^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。
5. **用客户侧指标而非 proxy metric**：at-speed completion rate 直接对应真实工业交付要求，比实验室简单任务的成功率更能暴露模型差距^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。
6. **真机结论必须有统计区间和足够 trial 数**：七个条件合计 750+ 次 trial、报告 credible interval；引用任何真机结果前先看区间宽度，个位数 trial 的成功率无意义^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。
7. **general web video（无动作标注）的 scale 价值首次被真机实证**：此前只有 manipulation 数据的 scaling 证据，这条结果扩宽了可用预训练数据的边界，与 [[entities/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026]] 的 video-action 预训练路线形成互补参照^[raw/articles/rhoda-scaling-web-video-pretraining-robots.md]。

相关：[[raw/articles/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026]]

→ [[raw/articles/rhoda-scaling-web-video-pretraining-robots|原文存档]]
