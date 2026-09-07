---
title: "Xiaomi-TabLDM：通用表格数据基础模型与表格数据 Scaling Law"
created: 2026-09-06
updated: 2026-09-07
type: entity
tags: [tabular-data, foundation-model, scaling-law, model-architecture, moe, test-time-scaling]
sources: [raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Xiaomi-TabLDM：通用表格数据基础模型

> 小米 2026-09-05 正式发布（代码 https://github.com/xiaomi-research/xiaomi-tabldm，权重 https://huggingface.co/occams/Xiaomi-TabLDM，技术报告 arXiv 2609.03880）。以单一预训练模型 + 统一默认配置直接适配不同表格数据集，无需针对每个任务重新训练/调参/后置集成，即可完成分类与回归预测。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]

## 核心命题：表格数据能否像语言一样获得跨数据集迁移

金融、医疗、制造、物流的核心数据多以表格形式存在。传统表格 ML（XGBoost/LightGBM/CatBoost）面对新数据集需重新训练、调参甚至集成模型。Xiaomi-TabLDM 探索「一次预训练、跨任务使用」的基础模型范式是否适用于结构化数据：表格预测从「一个数据集训练一个模型」走向「一次预训练、一个模型适配不同数据集」。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]

## 三个技术方向

1. **更大规模、更丰富的合成数据预训练**：完全基于合成表格任务预训练，通过结构化生成不同数据规模、变量类型、依赖关系和函数关系，扩大预训练任务分布覆盖。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]
2. **更高效的模型 Scaling**：引入双流 Feature Group、轻量级 Attention Residual 和 Sparse Mixture-of-Experts——从不同粒度建模特征关系、在更深网络中选择性复用历史信息、通过稀疏专家扩大模型容量，使性能提升不必伴随计算量同比增长。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]
3. **Test-Time Scaling**：保持预训练参数不变，通过增加推理阶段计算，利用不同特征排列、数据变换和预测视角产生互补结果，并根据当前数据集自适应选择和组合，持续提升预测性能。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]

## Benchmark 表现（四大公开基准第一梯队）

- 回归与二分类任务尤为突出：TALENT 二分类第一；OpenML-CTR23 回归第一；BCCO 总体第二；TabArena 回归第二。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]
- 超过 TabPFN-3、AutoGluon 1.5 extreme、TabPFN-2.6、TabICL-v2 等强基线。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]
- 性能-效率权衡：TabArena Regression 1900 Elo 同时每 1K 样本验证仅 3.12s，显著低于 TabFM 的 9.67s。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]

## 生态定位

- 与 [[entities/introducing-tabfm-a-zero-shot-foundation-model-for-tabular-d|TabFM（Google）]]、[[entities/aws-fundamentals-large-tabular-model-nexus-is-now-available-on-amazon-sagemaker-jump|AWS 大表格模型 Nexus]] 同属表格基础模型赛道；Xiaomi-TabLDM 是首个主打「合成数据预训练 + Sparse MoE + Test-Time Scaling」三合一路线的开源中文团队方案。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]
- 延续小米开源基座模型路线（[[entities/xiaomi-dasheng-audio-foundation-model-2026|Xiaomi DaSheng 音频基座模型]]），模型权重 + 代码 + 技术报告全量开源。^[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026.md]

## 相关实体

- [[entities/introducing-tabfm-a-zero-shot-foundation-model-for-tabular-d|TabFM]]
- [[entities/aws-fundamentals-large-tabular-model-nexus-is-now-available-on-amazon-sagemaker-jump|AWS 表格模型 Nexus]]
- [[entities/xiaomi-dasheng-audio-foundation-model-2026|Xiaomi DaSheng]]
- [[concepts/scaling-laws|Scaling Laws]]

→ [[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026|原文存档]]
