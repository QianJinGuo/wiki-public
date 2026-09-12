---
title: "NVIDIA MPS + Triton 降低 ASR 推理成本 75%"
created: 2026-08-30
updated: 2026-09-10
type: entity
tags: [nvidia, mps, triton, asr, inference, gpu-optimization, cost-reduction, speech-recognition]
sources: [raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# NVIDIA MPS + Triton 降低 ASR 推理成本 75%

## 摘要

用 CUDA MPS + Triton Inference Server，AWS/NVIDIA/Heidi Health 把 ASR 推理的 GPU 基础设施从 16 实例降到 4 个（-75%），延迟维持亚秒（mean < 650 ms、p99 < 1,000 ms），吞吐 92.1 RPS/GPU。Heidi Health 每周在 190 个国家处理 240 万+ 次咨询，瓶颈是 time-slicing 强制串行、单请求只占 15–20% 算力。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

## 核心要点

- **根因**：单次 ASR 请求只用 L40S 142 个 SM 的 15–20%，time-slicing 强制串行。
- **手段**：MPS 让多进程共享单一 MPS daemon GPU context，kernel 并发、零代码改动。
- **收益**：16 → 4 实例（-75%）；吞吐 ~23 → 92.1 RPS；利用率 ~20% → ~80%。
- **三层**：MPS 共享 → ONNX Runtime + TensorRT EP（融合 + FP16）→ Triton batching。
- **分区**：转写 4 × 25% SM（各 ~2.5 GB）；分离 8 × 12% SM（各 ~1.8 GB）。

## 问题背景

Heidi Health 每周处理 240 万+次临床咨询，关键瓶颈是单次请求只用 GPU 算力的 15–20%，而 CUDA 默认 time-slicing 强制顺序访问，导致 80% 硬件闲置。L40S 上单卡在 SLA 内（mean < 650 ms、p99 < 1,000 ms）仅约 62 RPS，故需 16 台 GPU 扛峰值。三种共享机制：^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

| 机制 | 隔离性 | 并发执行 | 适用场景 |
|------|--------|----------|----------|
| Time-slicing（默认） | 完整上下文切换 | 否——串行 | 少量大模型 |
| MIG | 硬物理分区 | 是——固定分区 | 多租户强隔离 |
| MPS | 共享 context、软 SM 限额 | 是——kernel 并发 | 单卡多个小模型 |

## 技术方案

### NVIDIA MPS（Multi-Process Service）

MPS 允许多个 CUDA 进程共享同一 GPU 的计算资源，实现真正的并行执行；对 ASR 这类计算密度低但延迟敏感的场景，GPU 利用率可从 ~20% 提升到 ~80%。它是 CUDA API 的 binary-compatible 实现：工作经 MPS daemon 汇入同一 context，消除上下文切换开销、支持跨进程 kernel 并发、以独立地址空间保护内存；分区由 `CUDA_MPS_ACTIVE_THREAD_PERCENTAGE` 配置。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

### Triton Inference Server

NVIDIA Triton 作为推理服务框架，管理模型加载、请求调度和批量推理；配合 MPS，可在同一 GPU 上并发处理多个 ASR 请求。转写用 dynamic batching（50 ms 累积、`max_queue_delay_microseconds: 50000`、preferred sizes [4, 8, 16]），分离用 sequence batching（15 秒块 + correlation ID、`max_idle_timeout` 600s）；每个 model instance 对应一个 MPS 分区。pipeline：FastAPI gateway（Whisper 兼容 REST、torchcodec、gRPC）→ Triton → MPS daemon（g6e / g7e.4xlarge，L40S 48 GB）。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

### 量化结果

| 指标 | 优化前 | 优化后 | 改善 |
|------|--------|--------|------|
| GPU 实例数 | 16 | 4 | -75% |
| GPU 利用率 | ~20% | ~80% | +4x |
| 延迟 | 亚秒 | 亚秒 | 持平 |
| 吞吐量/GPU | ~23 RPS | 92.1 RPS | +4x |

SLA = mean < 650 ms 且 p99 < 1,000 ms；基准扫 1–100 并发、每档 5 轮均值：

| 配置 | 实例 | Max Conc | RPS | Mean | p99 | 节省 |
|------|------|----------|-----|------|-----|------|
| Triton Baseline | g6e.4xlarge | 20 | 62.3 | 606 ms | 788 ms | — |
| Triton + MPS | g6e.4xlarge | 28 | 60.8 | 470 ms | 947 ms | 75% |
| Triton + MPS | g7e.4xlarge | 32 | 92.1 | 352 ms | 769 ms | 75% |
| TensorRT+ONNX+MPS | g7e.4xlarge | 64 | 111.6 | 590 ms | 896 ms | 88% |

g7e 最优点比 g6e 吞吐高 51%+、延迟低 25%+；TensorRT + ONNX 虽降到 2 台（-88%），但需每周重导出 ONNX，故生产选 75% 那条。分离模型（Sortformer 4-speaker v2，8 × 12% SM）warmup 后 mean 309 → 239 ms（-23%）、p50 348 → 238、p95 469 → 356、p99 499 → 389 ms（-22%~-32%），σ 12.62 → 7.13 ms；60 秒录音拆 4 × 15 秒块。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

## 适用场景

这套方案适用于所有 **计算密度低但延迟敏感** 的推理场景：
- ASR（自动语音识别）
- TTS（文本转语音）
- 轻量级分类/嵌入模型
- 实时视频分析

关键特征：单次请求只用少量算力 + 亚秒延迟需求；单请求已占满 SM 时软分区只会争抢。^[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e.md]

## 实现要点

1. **模型与实例**：Nemotron Speech ASR 0.6B V2（Parakeet TDT 0.6B V2，600M，需 NeMo 2.7+）；跑在 P4d（A100）或 g6e / g7e.4xlarge（L40S 48 GB）。
2. **MPS**：`CUDA_MPS_ACTIVE_THREAD_PERCENTAGE` 定每进程占比（转写 25% / 分离 12%）；`MPS_INSTANCE_COUNT` 决定 Triton instance 数与 SM 百分比（`auto_config.py` 生成）。
3. **Triton**：concurrent model execution + dynamic batching（[4, 8, 16]、queue delay 50000 µs）+ sequence batching（idle 600s）。
4. **模型级**：Conformer encoder（24 层 / 1024 hidden）走 TensorRT EP 融合 + FP16 校准；TDT decoder 留 PyTorch CUDA + graph caching。
5. **Direct forward**：直调 `model.forward()`（省 ~50 ms/请求）+ bfloat16 autocast + 每实例独立 CUDA stream，45 秒音频约 160 ms。
6. **graph 预热信封**：预烧 5/15/30/45/60 秒与 61 秒；命中约 165 ms，超出回退 eager ~500 ms。
7. **稳定性**：`fcntl.flock` 串行加载；1.5–2.5 秒 graph 重捕窗口被兄弟实例污染 → `cudaErrorIllegalAddress`，需 MPS-safe fallback。
8. **部署**：`Dockerfile.single` 构建后以 `MPS_INSTANCE_COUNT=4` 起容器，`/health` 200（90–120 秒）就绪。

## 深度分析

### 为什么是 4×25% 而不是更细的分区

MPS 的 SM 配额是软上限，分区越细、调度窗口越小；4×25% 与 preferred sizes [4, 8, 16] 对齐，batch 槽位与分区一对一。瓶颈在 SM 调度粒度与 CPU 侧开销，而非显存（2.5 GB × 4 远小于 48 GB）。

### 收益来自三层正交优化

MPS 管"同时能挂多少活"，TensorRT/ONNX 管"每份活多久"，Triton batching 管"多少活凑一起交"。单上 MPS 只有"实例减半、吞吐持平"，叠加 TensorRT 才把 g7e 推到 111.6 RPS。

### 亚秒 SLA 约束的是并发上限

并发 1 → 100 时 p50 由 121 ms 涨到 964.7 ms，RPS 在 32 并发后饱和（92.1 → 106）：延迟上涨来自排队而非计算变慢，杠杆是"每个分区允许多深的队列"。

### 会崩在哪：共享 context 是硬边界

地址隔离保护内存，但 CUDA graph 重捕、非法地址属 context 级状态——一个实例翻车会污染兄弟实例的 capture，`cudaErrorIllegalAddress` 由此而来；能否上生产取决于 flock 串行加载、预热信封 + eager 回退、wedge sentinel + 探针。

## 实践启示

1. **先量 request-to-SM 比例再决定买卡**：单请求只用 15–20% SM 时，扩容实例数是最贵的解。
2. **按隔离需求选机制**：强隔离用 MIG、弹性超配用 MPS，两者可组合。
3. **model instance 与 MPS 分区一对一**，并让 preferred batch sizes 与分区数对齐。
4. **生产从 g7e.4xlarge + 4 实例起步**，p99 有余量再上调 `MPS_INSTANCE_COUNT`。

## 相关实体

- [[entities/vllm|vLLM 高吞吐推理]]
- [[entities/gpu-virtualization-using-mig-technology-on-amazon-sagemaker-hyperpod|MIG GPU 虚拟化]]
- [[concepts/inference-optimization|推理优化]]
- [[entities/meta-muse-voice-transcribe-streaming-asr-2026|Meta Muse 流式 ASR]]

→ [[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e|原文存档]]
