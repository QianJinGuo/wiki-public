---
title: "Xiaomi-TabLDM：通用表格数据基础模型与表格数据 Scaling Law"
created: 2026-09-06
updated: 2026-11-07
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

## 深度分析

### 设计取舍：用可控的合成任务分布替代真实语料

语言模型可以「吸」整个互联网文本，表格数据却没有天然的大规模语料——企业数据分散、敏感且互不兼容。Xiaomi-TabLDM 选择完全基于合成表格任务预训练：结构化地生成不同数据规模、变量类型、依赖关系与函数关系。这条路线的优势是任务分布完全可控（可审计、可定向扩充、无数据合规与泄漏问题）；代价是模型能力上限受制于生成器对真实函数族的覆盖程度，合成先验与具体业务表之间的分布偏移成为核心风险。这与 [[entities/introducing-tabfm-a-zero-shot-foundation-model-for-tabular-d|TabFM（Google）]]、[[entities/tabclustpfn-tabular-clustering-prior-data-fitted-network-2026|TabPFN 系 prior-fitted 网络]] 的思路同源：表格基础模型的竞争，本质上是「谁的预训练任务分布更接近真实世界的表格函数空间」。

### 容量扩展三件套：Feature Group、Attention Residual 与 Sparse MoE

表格输入没有语言那样的天然 token 序列，「往深做、往宽做」不能照搬 Transformer 标准配方。Xiaomi-TabLDM 的组合是：双流 Feature Group 在不同粒度上建模特征关系（相当于替模型预先定义特征分组结构）；轻量级 Attention Residual 让更深网络可以选择性复用历史信息（缓解深层网络的信息通路退化）；[[concepts/moe-mixture-of-experts-2025|Sparse MoE]] 用稀疏专家扩大总容量而每个样本只激活部分参数。三者指向同一目标——性能提升不必伴随计算量同比增长。这正是表格 Scaling Law 区别于 LLM Scaling Law 的工程重心：表格模型可以对异构特征主动设计计算分配，而 LLM 只能对均匀 token 一视同仁。

### Test-Time Scaling 在表格数据上首次成轴

Test-Time Scaling（用推理阶段计算换精度）在语言与推理模型上已被反复验证，在表格预测上此前缺少系统证据。Xiaomi-TabLDM 保持预训练参数冻结，利用不同特征排列、数据变换和预测视角产生互补预测，再按当前数据集自适应选择与组合。配合「把新数据集的标注样本当上下文」的 in-context 适配，表格预测实际拥有了一个三维扩展面：参数规模 × 预训练任务分布 × 推理计算，与 [[concepts/scaling-laws|Scaling Laws]] 的通用叙事以及 [[entities/harness-evolution-vs-test-time-scaling-2026|Agent 领域的 Test-Time Scaling 演进]] 相互印证。效率数据是关键佐证：TabArena Regression 1900 Elo 的同时每 1K 样本验证仅 3.12s（TabFM 为 9.67s）——性能与推理成本没有互相绑架。

### 小样本工况适应：基础模型成色的试金石

公开榜单之外，工业案例更能检验「基础模型」的成色——判据是适应新任务是否只需上下文、不需梯度更新。三个信号：材料性能预测无需微调精度提升 130%，并减少约 90% 的无效试模与装机测试；工况变化时仅补充约 30 条新样本作上下文，平均误差再降约 62%，且仅用传统方法约 15% 的样本量即反超补足 200 条样本的 XGBoost；零件重量预测失误样本比例降约 31%、生产组分预测平均误差降约 54%。「少量新样本即插即用」正是 GBDT 范式做不到的能力，也是表格基础模型最直接的落地价值。

## 实践启示

1. **新表格任务先跑 zero-shot 基线，再决定是否调参。** 在手写 XGBoost/LightGBM 流水线之前，先用 Xiaomi-TabLDM 这类基础模型直出一次结果；标注样本越少（几十到几百条），上下文学习越可能直接取胜，省下的调参时间可以投向数据质量。
2. **工况漂移时「补样本当上下文」优先于触发重训。** 约 30 条新工况样本即可把平均误差降低约 62% 且无需微调——产线分布漂移的第一反应应是补采少量标注样本喂给模型，而非重启训练流水线。
3. **选型时把「每千样本推理秒数」纳入 benchmark 决策矩阵。** 1900 Elo @ 3.12s/1K（对比 TabFM 9.67s/1K）说明榜首模型之间效率差距可达 3 倍；只看榜单名次，可能选到线上跑不起的模型。
4. **数据合规受限时，合成任务分布是可行的预训练替代路线。** 无法汇集真实表格语料的团队可以走结构化合成数据路线，但需对生成器做覆盖度审计（变量类型、依赖结构、函数族是否覆盖业务场景），否则分布偏移会直接封顶下游精度。
5. **自研表格模型的容量扩展优先考虑稀疏化，而非单纯加深加宽。** Feature Group + Attention Residual + Sparse MoE 的组合展示了「容量涨、计算不涨」的可行路径，对算力预算敏感的团队尤其值得参考。
6. **把多视角推理当作免费的性能后备杠杆。** 预训练参数冻结时，特征排列/数据变换/多预测视角加自适应组合是无需重训的提升通道，适合作为线上低成本 A/B 实验的首选。

## 相关实体

- [[entities/introducing-tabfm-a-zero-shot-foundation-model-for-tabular-d|TabFM]]
- [[entities/aws-fundamentals-large-tabular-model-nexus-is-now-available-on-amazon-sagemaker-jump|AWS 表格模型 Nexus]]
- [[entities/xiaomi-dasheng-audio-foundation-model-2026|Xiaomi DaSheng]]
- [[concepts/scaling-laws|Scaling Laws]]

→ [[raw/articles/xiaomi-tabldm-tabular-foundation-model-scaling-law-2026|原文存档]]
