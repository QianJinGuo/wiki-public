---
title: "Benchmarking Small LLM Inference on SageMaker：G7 vs G5/G6（30B MoE 跨代 GPU 评测）"
created: 2026-09-09
updated: 2026-09-09
type: entity
tags: [llm, inference, benchmark, moe, quantization, gpu, aws, inference-optimization]
sources: [raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026]
confidence: 0.7
provenance_state: extracted
---

# Benchmarking Small LLM Inference：G7 vs G5/G6

> AWS ML Blog 对两个代表性 30B MoE 模型跨 GPU 实例家族的基准评测（G7 Blackwell / G5 A10G / G6 L4 / G6e L40S），展示 G7 在吞吐/延迟/单 token 成本的可量化收益。核心价值在**跨代 GPU 选型的评测方法论与 MoE 推理硬件权衡**，而非平台绑定。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 为什么 MoE 推理对内存带宽敏感

MoE 架构在 token 生成（decode）阶段内存带宽受限：每个 token 只激活一小部分 expert，驻留权重远超单次激活量。**更高内存带宽直接降低 inter-token 延迟、提升吞吐**——这是选型判断比单纯看参数量更关键的维度。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## NVFP4 与低精度原生支持

- **NVFP4**：NVIDIA Blackwell 引入的 4-bit 浮点格式，Tensor Core 原生支持，权重压到约每权重 4 bit 且质量损失最小。
- **结构性优势**：只有 G7 有原生 FP4 Tensor Core；G5/G6 无硬件加速跑 NVFP4 权重，导致 G7 在 MoE 部署上有结构性优势。这是「量化格式 × GPU 代次」协同选型的典型案例。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 使用案例1：Qwen3-Coder-30B（编码助手）

同模型 + DJL LMI 28.0 容器 + 同 workload 对比 ml.g5/g6/g7.12xlarge（G5/G6 各 4 GPU×96GB，G7 2 GPU×64GB，**G7 用一半加速器和更少显存胜出**）：

| 指标 | G5 | G6 | G7 |
|---|---|---|---|
| 输出 token 吞吐 | 346.3 tok/s | 243.4 | **391.3** |
| 请求吞吐 | 2.69 req/s | 1.89 | **3.04** |
| 平均请求延迟 | 1475 ms | 2110 | **1316** |
| P99 延迟 | 1881 ms | 3316 | **1501** |

G7 vs G6：吞吐 +60.8%、平均延迟降 37.6%、P99 降 54.7%；流式场景 G7 中位 TTFT≈118ms、平均 ITL 8.9ms。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 评估方法论（可迁移的框架）

- **基准流程**：单模型 + 多实例 + 同 workload 的 apples-to-apples 对比；用 `Workload.synthetic` 定义 token 分布/并发/请求数；AIPerf 采集吞吐、请求延迟、P50/P90/P99；交互应用再测流式 TTFT 与 ITL。
- **结论皆 workload 特定**：结果只对所用模型/服务配置/token 分布/并发有效。生产应按期望流量特征度量。
- **推荐流程（生成式 AI Inference Recommendations）**：给定模型/候选实例/目标（吞吐优化），系统评测候选配置并返回**排名的成本-延迟-吞吐推荐**——比人工逐实例 benchmark 更接近真实生产最优。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

## 与 wiki 焦点框架的连接

属 [[concepts/inference-optimization|推理优化]] 与 [[concepts/moe-mixture-of-experts-2025|MoE]] 的实证层：把 [[concepts/harness-engineering-framework|Harness Engineering]] 的「评测要区分能力vs预算」思路落到推理基础设施选型（G7 用一半显存赢，正是 [[entities/agent-harness-evolution-from-llm-call-to-harness-tencent-2026|Harness 评测对照]] 里『少资源多办事』的硬件版）。与 [[entities/ai-infra-llm-efficient-inference-vllm|vLLM 高效推理]]、[[entities/750b-moe-pd-disaggregation-aws-efa-vs-roce|750B MoE 分载]]、[[entities/amazon-bedrock-multimodal-model-benchmark-gaming-qa-2026|Bedrock 多模态基准]] 互补——本篇聚焦「小模型（30B）+ 大批量承载」的低成本推理选型方法论。^[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026.md]

→ [[raw/articles/benchmarking-small-llm-inference-sagemaker-g7-vs-g5-g6-aws-2026|原文存档]]