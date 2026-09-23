---
title: "Flattening Every Memory Peak in Long-Context Mixture-of-Experts Training"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: [moe, training, memory-optimization, long-context, distributed-training, inference-optimization]
sources: [raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026]
confidence: 0.8
provenance_state: extracted
---

# Flattening Every Memory Peak in Long-Context Mixture-of-Experts Training

## 核心问题

MoE 模型在长上下文或大 batch 训练时的失败模式是：**任何一个组件的内存峰值超过显存就会 OOM**，所以优化目标必须同时压住所有峰值，而不是只降平均占用。论文指出有四个内存峰值在常用并行方案下不受约束，且各自随不同维度增长：

1. **Expert dispatch** — 随 routing matrix 增长
2. **Vocabulary projection** — 随 tokens × vocab 增长
3. **Gradient checkpoint boundaries** — 随 depth × sequence length 增长
4. **Optimizer state** — 随参数量增长

哪个先爆取决于模型、上下文长度和设备数——只压最大那个只会让下一个暴露出来。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md]

## 四项技术（全部只改计算/数据搬运的顺序与粒度，loss 与梯度保持精确）

| 技术 | 针对峰值 | 机制 |
|------|---------|------|
| **PipelinedLLEP** | expert dispatch | 在 least-loaded expert parallelism 基础上，对每个 source 贡献到 dispatch chunk 的 token 数加 cap |
| **Ring-DTP** | vocabulary projection | 激活/权重分片沿 ring 循环，logits 分块折叠进 online log-sum-exp |
| **SCO (Selective Checkpoint Offload)** | checkpoint boundary | 每个 checkpoint boundary 只有一个长生命周期 tensor，保留在 CPU 内存 |
| **OffloadStreamAdamW** | optimizer state | 把 CPU Adam offload 的串行更新改成 bucket pipeline |

^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md]

## 实测结果

- MoE dispatch 峰值最多降 **59.3%**（吞吐无损）
- vocabulary projection 峰值降 **86.6%**
- offload optimizer step 快 **2.05×**
- 在 120B–667B 参数 MoE 模型上组合使用：**1M context length 训练**，达到调优 FSDP2 baseline 的 **8–32× 上下文 reach** 与最高 **10.4× 吞吐**

^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md]

## 与 wiki 已有内容的关联

- 与 [[entities/2026-05-04-DeepSeek做大-Mega-MoE-Tri-Dao团队加快-SonicMoE-机器之心]] 同属 MoE 训练内存优化谱系——SonicMoE 优化的是 MoE 前向/激活内存，本文系统性覆盖 dispatch/vocab/checkpoint/optimizer 四类峰值
- 与 [[entities/deepseek-moe-parallel-strategy]] 的并行策略互补：本文不改并行计划，在现有 FSDP/EP 方案之上做调度级约束
- 长上下文训练 reach 与 [[entities/750b-moe-pd-disaggregation-aws-efa-vs-roce]] 的 PD 分离部署构成训练-推理两端的长上下文工程拼图

## 相关链接

- 来源：[[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026|原文存档]]
- arXiv: https://arxiv.org/abs/2609.14306（cs.DC）
- 作者：Shrey Pandit 等 4 人
