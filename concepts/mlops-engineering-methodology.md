---
title: "MLOps 工程方法论"
created: 2026-07-02
updated: 2026-08-29
type: concept
tags: [mlops, methodology, production, ai-infra]
provenance_state: inferred
confidence: 0.75
---

# MLOps 工程方法论

> 知识库中 mlops 标签覆盖 166 个页面，是第 19 高频标签。但缺乏统一的 MLOps 概念框架。

## MLOps 五层模型

### L1: 数据工程
- 数据版本管理、特征存储、数据质量监控
- 训练/评估数据隔离（防止 benchmark contamination）
- 合成数据生成与验证

### L2: 训练工程
- 分布式训练编排（multi-node / multi-GPU）
- 实验追踪（MLflow / W&B / 自研）
- 超参搜索与 AutoML
- Checkpoint 管理与容错恢复

### L3: 模型管理
- 模型注册与版本管理
- A/B 测试框架与金丝雀发布
- 模型压缩（量化、蒸馏、剪枝）
- 模型签名与输入/输出 Schema

### L4: 部署与推理
- 推理服务化（vLLM / TensorRT / Triton）
- PD 分离（Prefill-Decode Disaggregation）
- KV Cache 管理与显存优化
- 弹性扩缩容与成本控制

### L5: 监控与治理
- 模型漂移检测（数据漂移 / 概念漂移）
- 推理质量监控（延迟 / 吞吐 / 正确率）
- 成本归因与 Token 经济学
- 合规审计与模型卡片

## AI 时代的 MLOps 新挑战

- **Agent 的可观测性**：Agent 的行为链路比传统模型复杂得多
- **Prompt 与 Skill 的版本管理**：新资产类型需要新治理模式
- **多模型路由**：根据成本/延迟/质量动态路由，使 MLOps 从"管一个模型"变为"管一个模型舰队"

## 关联

- [[concepts/inference-optimization|推理优化]]
- Agent 可观测性
- [[entities/750b-moe-pd-disaggregation-aws-efa-vs-roce|750B MoE PD 分离推理]]

## 所属 MOC

- [[moc/mlops-training-inference|Mlops Training Inference]]
