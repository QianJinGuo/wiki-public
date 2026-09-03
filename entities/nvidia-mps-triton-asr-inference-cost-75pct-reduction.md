---
title: "NVIDIA MPS + Triton 降低 ASR 推理成本 75%"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: [nvidia, mps, triton, asr, inference, gpu-optimization, cost-reduction, speech-recognition]
sources: [raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e]
confidence: 0.8
---

# NVIDIA MPS + Triton 降低 ASR 推理成本 75%

AWS + NVIDIA + Heidi Health 联合展示了如何用 NVIDIA CUDA Multi-Process Service (MPS) + Triton Inference Server 将 ASR 推理的 GPU 基础设施需求从 16 个实例降至 4 个（75% 成本降低），同时保持亚秒级延迟和 92.1 RPS/GPU 吞吐量。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

## 问题背景

Heidi Health 每周处理 240 万+次临床咨询，需要实时 ASR 转录。关键瓶颈：单次 ASR 推理请求只使用 GPU 算力的 15-20%，但 NVIDIA CUDA 默认的 time-slicing 行为强制顺序访问，导致 80% 硬件闲置。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

## 技术方案

### NVIDIA MPS（Multi-Process Service）

MPS 允许多个 CUDA 进程共享同一 GPU 的计算资源，实现真正的并行执行（而非 time-slicing 的伪并行）。对于 ASR 这类计算密度低但延迟敏感的场景，MPS 可以将 GPU 利用率从 ~20% 提升到 ~80%。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

### Triton Inference Server

NVIDIA Triton 作为推理服务框架，管理模型加载、请求调度和批量推理。配合 MPS，Triton 可以在同一 GPU 上并发处理多个 ASR 请求。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

### 量化结果

| 指标 | 优化前 | 优化后 | 改善 |
|------|--------|--------|------|
| GPU 实例数 | 16 | 4 | -75% |
| GPU 利用率 | ~20% | ~80% | +4x |
| 延迟 | 亚秒 | 亚秒 | 持平 |
| 吞吐量/GPU | ~23 RPS | 92.1 RPS | +4x |

## 适用场景

这套方案适用于所有 **计算密度低但延迟敏感** 的推理场景：
- ASR（自动语音识别）
- TTS（文本转语音）
- 轻量级分类/嵌入模型
- 实时视频分析

关键特征：单次请求只用 GPU 少量算力 + 需要亚秒延迟 = MPS 并行的最优场景。

## 实现要点

1. **模型选择**：Nemotron Speech ASR 0.6B V2（轻量级，适合 MPS 并行）
2. **GPU 实例**：AWS P4d（A100）或类似高算力实例
3. **MPS 配置**：设置 `CUDA_MPS_ACTIVE_THREAD_PERCENTAGE` 控制每进程资源占比
4. **Triton 配置**：启用 concurrent model execution + dynamic batching

## 相关实体

- [[entities/vllm|vLLM 高吞吐推理]]

→ [[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e|原文存档]]
