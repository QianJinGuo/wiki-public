---
title: Reranker 在 Domain Adaptation 中的作用与最佳实践？
created: 2026-05-21
updated: 2026-05-21
type: query
tags: [reranker, retrieval, domain-adaptation, fine-tuning]
sources:
  - raw/articles/reranker-domain-adaptation-data-engineering-finetuning
confidence: high
---
# Reranker 在 Domain Adaptation 中的作用与最佳实践？
## 核心问题
Reranker 如何在垂类场景（医疗、金融）Domain Adaptation 中发挥作用？有哪些数据工程和微调最佳实践？

## 一、训练数据来源与成本
### BGE-Reranker 预训练数据构成
| 来源 | 比例 | 说明 |
|------|------|------|
| NLP 公开数据集（NQ, HotpotQA, MS MARCO, BEIR）| ~40% | 通用问答 pairs |
| 用户点击日志（Query-Doc pairs）| ~35% | 弱标签 |
| LLM synthetic pairs（合成 query-doc 对）| ~20% | 大模型批量生成 |
| 人工精标 gold pairs | ~5% | 高质量标注 |

### 垂类训练数据获取路径
| 路径 | 成本 | 优点 | 缺点 |
|------|------|------|------|
| A：规则 + 合成（冷启动） | < 0.1 元/条 | 低成本快速启动 | 噪声较大，需多轮清洗 |
| B：用户行为数据 | ≈ 0 | 最便宜 | 曝光偏置严重 |
| C：人工精标 | 5-30 元/条 | 高质量 | 成本高，500-2000 条/垂类 |

## 二、效果退化感知→排查→修复
### 感知体系
- **每日 A/B 对比**：shadow vs 在线 reranker，Recall@5 / MRR@10 diff > 2% → 告警
- **用户负向反馈**：点击"没找到答案"的 session 占比
- **分场景监控**：医疗/金融/通用 分开看

### 排查维度
| 维度 | 可能原因 | 排查方法 |
|------|---------|---------|
| 数据漂移 | 新增 doc 分布变化、季节性 query 上升 | 查最近 N 天新增 doc/query 分布 |
| 误训脏数据 | finetune 用了含噪声的点击日志 | 回查训练数据来源和清洗流程 |
| LoRA adapter 被覆盖 | 多人协作时 adapter 被覆盖 | 查模型版本记录 |
| 工程变更 | embedding 服务版本不一致 | 查最近部署记录 |

### 修复策略
```
紧急止血（1 小时内）
├── 回滚 Reranker 到上一稳定版本
├── 切 shadow pipeline
└── 减小 K 值（Top-50 → Top-20）

短期修复（1-3 天）
├── 增量 finetune：最近 7 天用户行为数据 + 黄金集
├── Hard negative mining：针对垂类 query 挖困难负例
└── 数据增强：LLM 批量生成垂类 query 困难负例

中期基建（1-2 周）
├── 建立垂类 golden set（每月 +50 条）
├── 垂类术语表 + query/doc terminology normalization
└── 分场景 A/B 监控 dashboard
```

## 三、小样本微调（10 条标注不过拟合）
### 核心矛盾
10 条数据数量少、覆盖窄 → 直接 finetune = 过拟合记住 10 条

### 三阶段 Pipeline
```
阶段 1：LLM 合成扩展（10 条 → 300 pairs）
├── Paraphrase：同一 query，LLM 生成 10 种不同表达
├── 规则构造负例：同领域、不同子话题的 doc 作负例
└── Soft label：LLM 判定 doc_negative 的不相关程度（0.1 vs 0.05）

阶段 2：Hard Negative Mining
├── 同科室不同病种 → 中等难度负例
├── 表面相似但语义相反 → 困难负例
└── Embedding cosine distance 筛选 0.4-0.7 区间

阶段 3：微调策略
├── LoRA rank=8~16，lr=1e-4 ~ 5e-5
├── Pairwise ranking loss 替代分类 loss
└── Label smoothing + Dropout=0.3 + Weight decay=0.01
```

### 最终验证
- 10 条中留 2 条做验证（绝对不在训练集中）
- 训练完后验证 Recall@5 ≥ 0.8
- 上线 shadow 模式跑 3 天
- shadow Recall@5 diff < 2% → 全量上线

## 相关知识
- [[WORKFLOW|RAG Pipeline 整体架构]]

## 相关实体


## 相关概念

- [[concepts/retrieval-augmented-generation-rag|Retrieval-Augmented Generation]]
- [[concepts/inference-optimization|Inference Optimization]]
