---

title: "Hitting a billion tokens per minute on one GPU by combining a query planner and an inference engine"
created: 2026-09-29
updated: 2026-09-29
type: entity
tags: ["inference-optimization", "ai-sql", "kv-cache", "query-planner", "vllm", "modal", "benchmark"]
sources: [raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm]
confidence: 0.7
---

# Quail：AI-SQL 推理引擎（Modal × CMU）

## 定位与问题

Quail 是 Modal 与 CMU 数据库研究者合作开源的 **AI-SQL 推理引擎**：不是"用自然语言生成 SQL"（Text-to-SQL），而是方向相反——用 SQL 扩展**程序化地构造（并消费）发给 AI 系统的 prompt**，如 `AI.IF(PROMPT("{customers.profile} might buy this: {products.description}"))`。这类负载主要出现在 BI 平台对半结构化数据（文档、自由文本字段）的模糊查询场景。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md]

关键洞察：AI-SQL 负载与 chatbot/coding agent 负载的推理特征完全不同——单个查询可能产生数百万条数千 token 的序列，且所需智能远低于前沿模型（小开源模型即可胜任）。朴素地将其送入为 agentic inference 优化的引擎是大规模低效的。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md]

## 核心技术：查询计划器 + KV cache 主动管理

核心胜利点：**拿到结构化查询后，可以预先排序请求以更好地 cache（和驱逐）KV**——这需要对推理引擎的调度层做轻度改造。同时，大量针对小模型的小请求会产生可观开销，而预先知道请求结构可以规避这些开销。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md]

## 实测性能

- H100 单卡处理超过 **10 亿 tokens/分钟（TPM/GPU）**，查询计划收益最大的 multi-join 查询上比同硬件 vLLM baseline **快 >10x**
- 全任务几何平均 **1.84x faster than vLLM**（含两个刻意设计用于暴露改进空间的查询）
- 在 Modal 上折合 **每十亿 tokens 不到 6¢** 的成本
- 部署形态：`modal.Image.from_registry("nvidia/cuda:13.0.1-devel-ubuntu24.04")` + `@app.function(gpu="H100!")`，文档数据经 `quail.DocumentProvider.from_table()` 接入

## 与已有 wiki 实体的关系

与 [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026]]（Jev 快判断层）互补：Jev 解决"Agent 判断题从大模型里拆出来"（模型侧），Quail 解决"结构化 prompt 负载的推理调度"（推理侧）。两者都指向同一趋势——**大量低智能密度、高吞吐的推理负载需要专门的系统工程**，而非通用 frontier 推理引擎。作者团队自称这是"推理与数据库交叉的开源性能工程的开端"，鼓励"expert-parallel"协作（数据库视角另有专文）。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md]

## 关联

- → [[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm|原文存档]]
- [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026]] — 同为"低智能密度推理负载"工程化趋势的模型侧案例
- [[concepts/inference-optimization]] — KV cache 调度与吞吐优化方法面
- [[entities/vllm-v0-to-v1-correctness-before-corrections]] — 对照 baseline 引擎

