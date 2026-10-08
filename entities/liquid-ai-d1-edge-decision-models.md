---
title: "Open d1: Edge Decision Models"
description: "Liquid AI 开源 d1 决策模型家族（d1-3B + d1-omni-600M）：单次前向输出决策而非逐 token 生成，Decision Index v0.2.1 48.57 领先 10B 以下全模型，Jetson 端侧单问 16-50ms，llama.cpp day-one 支持"
created: 2026-10-09
updated: 2026-10-09
type: entity
sources: [raw/articles/liquid-ai-d1-edge-decision-models]
tags: [small-model, edge-deployment, decision-model, liquid-ai, multimodal, inference-optimization, open-weights]
confidence: 0.8
provenance_state: extracted
---

# Open d1: Edge Decision Models

> **Background**：Liquid AI 于 2026-10-07 发布开源 d1 决策模型家族。与生成式 LFM 家族（[[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|LFM2.5-2.6B]]、[[entities/liquid-ai-lfm2-5-230m|LFM2.5-230M]]、[[entities/liquid-ai-lfm2-5-encoders-fast-long-context-cpu|LFM2.5 Encoders]]）互补，d1 定位"决策"而非"生成"——单次前向直接输出答案，不做 token 序列生成，面向实时结构化决策场景。^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 模型家族与核心架构差异

- **d1-3B**：从 LFM2.5-VL-3B（decoder-only VLM）训练，接受文本+图像输入。Decision Index v0.2.1（public split）48.57，领先 10B 以下全部模型，与 12 倍大的 Decider 35B-A3B 持平。训练配方：先平均 LFM2.5-2.6B 与 LFM2.5-VL-3B 文本骨干权重构建更强 base，再以不同种子/数据混合微调多个 checkpoint 后合并；作者强调长输入训练、打乱选项顺序、修数据 shortcut 比更高级技术收益更大。^[raw/articles/liquid-ai-d1-edge-decision-models.md]
- **d1-omni-600M**：首个实验 checkpoint，从双向 encoder LFM2.5-Encoder-350M 训练，覆盖文本+图像与文本+音频两种模态组合。音频路径用 FastConformer encoder + adapter（冻结文本骨干微调音频编码器）；视觉路径从 LFM2.5-VL-450M 取 encoder 训练 adapter + LoRA（仅在图像输入时激活），随后全模型微调、合并 LoRA、与前 checkpoint 权重平均正则化。Decision Index 15.95。^[raw/articles/liquid-ai-d1-edge-decision-models.md]
- **与生成式 LFM 的本质区别**：d1 不产出 token，而是**单次前向传播直接给出答案**。这使延迟计量从 per-token 变为端到端输入→输出，也是端侧实时决策（每帧推理）可行的前提。^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 文本 Benchmark（Decision Index v0.3 七项公开基准均值 82.9）

d1-3B 均值 82.9 领先全表（Decider 4B 81.1）；d1-omni-600M 78.4 以 1/4 参数量超 Decider 2B（77.1）。亮点：Civil Comments 毒性检测 95.8（全表最高）、PAWS-X 释义识别 79.5。SQuAD 2.0 85.3 / XNLI 85.0 / BoolQ 87.3。视觉能力沿用 LFM2.5-VL-3B backbone 标准视觉基准验证（专用视觉 split 未公开报告）；音频决策基准被明确指出是 open problem。^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 端到端延迟（决策模型的核心指标）

| 设备 | 单问 | 3 问 | 3.4K-token 状态 | 384px 图像 |
|------|-----|------|----------------|-----------|
| Apple M5 Pro | 30ms | 41ms | 640ms | 62ms |
| Jetson AGX Thor | 16ms | 20ms | 220ms | 35ms |
| Jetson AGX Orin 64GB | 26ms | 35ms | 560ms | 83ms |
| Jetson Orin Nano | 50ms | 73ms | 1640ms | 202ms |
| NVIDIA RTX 4090 | 8ms | 21ms | 102ms | 17ms |
| AMD MI325X | 9ms | 14ms | 44ms | 18ms |

单问在所有实测设备 <50ms；3 问仅 1.3× 单问时间（AGX Thor 16→20ms）——单次前向架构下问题数与延迟接近线性且斜率极小。^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 部署生态

- 全 NVIDIA 栈（DGX → RTX → Jetson）day-one 支持，llama.cpp 原生支持覆盖 Apple/AMD/Qualcomm/NVIDIA（NVFP4）
- 两个模型均已在 Hugging Face 开放权重，可下载/微调/部署无限制
- 十个实机 demo：d1-3B 对实时摄像头输入逐帧单次前向循环推理（手势控制游戏、实时内容审核等），另与 NVIDIA 合作展示 Jetson + Isaac Sim hardware-in-the-loop 环境导航^[raw/articles/liquid-ai-d1-edge-decision-models.md]

## 关联

- [[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|LFM2.5-2.6B]] — d1-3B 的权重平均来源之一，同厂商生成式侧
- [[entities/liquid-ai-lfm2-5-230m|LFM2.5-230M]] — 同厂商边缘小模型
- [[entities/liquid-ai-lfm2-5-encoders-fast-long-context-cpu|LFM2.5 Encoders]] — d1-omni-600M 的 encoder 骨干家族
- [[concepts/inference-optimization]] — 单次前向决策 vs 自回归生成的推理成本对比维度
- [[concepts/model-distillation-compression]] — 小模型能力上限话题

→ [[raw/articles/liquid-ai-d1-edge-decision-models|原文存档]]
