---
title: "Native-speed vLLM transformers modeling backend"
created: 2026-08-14
updated: 2026-09-07
type: entity
tags: [vllm, transformers, inference, optimization, huggingface, model-integration]
sources: [raw/articles/native-speed-vllm-transformers-modeling-backend]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Native-speed vLLM transformers modeling backend

Hugging Face 2026-07 发布 vLLM transformers modeling backend 提速成果：**transformers 实现的模型在 vLLM 引擎内达到（或超过）手写 vLLM 实现的速度**——模型作者无需再为每个框架各写一遍自定义实现。^[raw/articles/native-speed-vllm-transformers-modeling-backend.md]

## 机制

- 旧模式：新模型要集成两次——transformers 一次 + vLLM 自定义优化一次；追求极致性能仍需手写 vLLM 实现
- 新模式：模型作者写好 transformers 实现即自动获得 vLLM 高速推理；vLLM 引擎在运行时插入注意力实现等高性能组件
- 支持范围：多数 LLM 架构；线性注意力模型暂不支持；Hub repo 中的自定义模型因未按规范编写而大概率不兼容

## 基准方法

三条件对照（唯一差异是代码路径）：`native`（vLLM 手写，基准线）/ `after`（transformers + PR）/ `before`（transformers 无 PR）。可复现 runner 以 gist 形式公开。^[raw/articles/native-speed-vllm-transformers-modeling-backend.md]

## 意义

这是 [[entities/vllm|vLLM]] 生态与 [[entities/vllm-v0-to-v1-correctness-before-corrections|transformers 兼容性]] 的关键里程碑：把"写一次代码"的愿景从训练侧（transformers）延伸到推理侧（vLLM），属于 [[concepts/inference-optimization|推理优化]] 的工程范式变化——推理性能不再要求重复实现。

→ [[raw/articles/native-speed-vllm-transformers-modeling-backend|原文存档]]
