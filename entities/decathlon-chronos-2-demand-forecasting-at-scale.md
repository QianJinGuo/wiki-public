---

title: "Decathlon Chronos-2 需求预测规模化部署"
created: 2026-08-31
updated: 2026-09-05
type: entity
tags: [time-series, forecasting, foundation-model, chronos, retail, supply-chain, lora, autogluon]
sources: [raw/articles/decathlon-chronos-2-demand-forecasting-at-scale]
confidence: 0.8
---

# Decathlon Chronos-2 需求预测规模化部署

Decathlon（迪卡侬）将 Amazon Chronos-2 时序基础模型部署到生产环境，替代原有的 DeepAR + TFT 方案，在东南亚和拉美两个供应区域实现 11-15 个百分点的 WAPE 改进。 ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]

## 核心架构

**模型选择**：Chronos-2（120M 参数 encoder-only transformer，T5 encoder 设计），通过 AutoGluon 集成使用 LoRA 微调，每 6 个月更新一次。 ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]

**部署模式**：
- 推理：Amazon EC2 m6i.8xlarge（CPU），每周批量推理
- 微调：g5.4xlarge（GPU），每 6 个月一次
- 编排：Databricks + Airflow
- 模型管理：MLflow 注册表，按供应区域版本化 ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]

**规模**：每个供应区域 25,000 个产品，12 周和 52 周两个预测窗口，每周运行。

## 关键结果

| 区域 | 窗口 | 旧 WAPE | Chronos-2 WAPE | 改进 |
|------|------|---------|----------------|------|
| 东南亚 | 12 周 | 39% | 28% | -11pp |
| 拉美 | 12 周 | 53% | 38% | -15pp |
| 东南亚 | 52 周 | 44% | 38% | -6pp |
| 拉美 | 52 周 | 55% | 46% | -9pp |

**业务影响**：每 1pp WAPE 改进 = 0.3 天库存节省 + 0.3pp 产品可用性 + 0.12pp 销售增长。 ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]

**运维效率**：
- 新区域部署时间：6 个月 → 2-3 个月
- 推理时间：10-15 分钟 → 40-75 秒
- 微调频率：每周 → 每 6 个月
- 推理成本：~$0.03/周/区域 ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]

## Chronos-2 技术要点

Chronos-2 与原始 Chronos 的关键区别：
- 原始 Chronos 将值量化为离散 token；Chronos-2 使用 robust scaling + patch embedding
- 交替注意力模式：time attention（单序列时间轴）+ group attention（跨序列 patch 对齐）
- 原生协变量支持（通过 group attention 机制），无需 workaround
- 支持 Matryoshka 表示学习（MRL），可动态调整嵌入维度 ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]

## 微调策略

使用 AutoGluon-TimeSeries 的 LoRA 微调：
- 低频微调（每 6 个月）仍显著优于零样本
- 按供应区域独立微调，超参数通过 MLflow 版本化
- 未来计划：MoE 集成（多模型融合，基准显示 40% 产品上其他模型仍优于单一最佳专家） ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]

## 经验教训

1. **在自有数据上做基准测试**：全局排行榜的排名可能与领域数据不一致
2. **微调释放全部潜力**：即使低频 LoRA 微调也显著优于零样本
3. **从小处着手，迭代扩展**：先单模型生产，逐步添加协变量
4. **基础模型民主化预测**：单台 CPU 实例即可运行，无需 GPU 基础设施 ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]

→ [[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale|原文存档]] ^[raw/articles/decathlon-chronos-2-demand-forecasting-at-scale.md]