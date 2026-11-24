---
title: "NVIDIA MPS + Triton 降低 ASR 推理成本 75%"
created: 2026-08-30
updated: 2026-11-24
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

### 为什么是 4×25%，而不是更细的软分区

单条 ASR 请求占 L40S 142 个 SM 的 15–20%，而 `CUDA_MPS_ACTIVE_THREAD_PERCENTAGE` 是软上限而非预留——分区一旦小于单请求的峰值需求，相邻分区的 kernel 就会互相挤占，延迟抖动随之放大。4×25% 是"每个分区装得下一个满负荷请求且留有余量"的最粗安全粒度：142 SM 的 25% 约 35 个 SM，对 21–28 个 SM 的单请求需求留出余量，四个分区又恰好铺满整卡。更细的 8 分区确实存在——分离模型走的就是 8×12%（各 ~1.8 GB）——但那服务的是算力需求更低的 Sortformer；转写若拆 8×12.5%，每分区反而低于单请求峰值。显存不是约束（2.5 GB × 4 远小于 48 GB），真正要对齐的是 preferred batch sizes [4, 8, 16] 与分区间的一对一映射。

### 三层正交优化：MPS 管并发，TensorRT 管单请求，Triton 管排队

三层的收益维度互不重叠，缺一层就少一段量化跳变。MPS 解决"同一时刻能挂多少活"：利用率 ~20%→~80%、实例 16→4；但单上 MPS 在 g6e 的最优点吞吐几乎持平（62.3→60.8 RPS），因为每份活的计算时间没变。模型层解决"每份活多久"：Conformer encoder（24 层 / 1024 hidden）走 TensorRT EP 做 kernel 融合 + FP16 校准，TDT decoder 因变长 token 生成留在 PyTorch CUDA + graph caching；叠加后 g7e 才推到 111.6 RPS（64 并发才饱和）。Triton batching 解决"多少活凑一批"：50 ms 累积窗口 + preferred sizes [4, 8, 16] 把零散请求填满各分区。生产最终停在 75% 那条线，是第四个维度——运维成本——说了算：每周重导出 ONNX 的代价，压过了再省 2 台实例的收益。

### 亚秒 SLA 约束的是每分区的队列深度

g7e 的并发扫描（1→100，每档 5 轮均值）显示吞吐在 32 并发后基本饱和：92.1 → 99.7 → 106.0 RPS，而 p99 从 768.8 ms 翻倍到 1,544.6 ms，p50 从 290.5 ms 涨到 964.7 ms——多买到的 ~15% 吞吐全是用延迟换的。原因是 4 个分区的算力上限决定了稳态吞吐，超出的并发只是把等待从客户端挪进 Triton 的 50 ms 累积队列；单请求计算本身没有变慢（45 秒音频 ~160 ms）。所以"最优运行点"被定义为 mean < 650 ms 且 p99 < 1,000 ms 的最后一个并发档位——SLA 实际约束的是每分区允许排多深的队，容量规划的锚点是 4 GPU × 92.1 RPS，而不是压测能摸到的峰值 106 RPS。

### 共享 context 是崩溃的硬边界

MPS 的隔离承诺只到地址空间一级：内存受独立地址空间保护，但 CUDA context 与 graph 捕获状态是全卡共享的。实测的翻车路径：NeMo decoder 遇到预热信封之外的新 tensor 形状时触发 graph 重捕，1.5–2.5 秒的捕获窗口内兄弟实例的 kernel 会污染捕获，进程以 `cudaErrorIllegalAddress` 崩溃。生产可用的三个前置条件全部围绕这条边界设计：`fcntl.flock` 串行化加载（600M 模型 × 4 进程同时初始化会打爆显存）；5/15/30/45/60 秒 + 61 秒 batch 2 的预热信封让生产形状全部命中缓存 graph（~165 ms，超出回退 eager ~500 ms）；wedge sentinel——CUDA stream 探针失败即写 `/tmp/parakeet_wedged`，纳入 /health 存活判定。75% 的节省，代价是故障域从"单进程"扩大到"单卡"。

### 成本账：-75% 的构成与 -88% 的取舍

基线是 16 台 g6e.4xlarge 扛 ~62.3 RPS/GPU（mean 606 ms、p99 788 ms）。同一实例类型叠加 MPS 后每卡吞吐几乎持平（60.8 RPS）但实例数砍到 4 台——这 75% 完全来自消灭闲置，模型未动。换 g7e.4xlarge 后同样 4 台把每卡抬到 92.1 RPS（+51%）、mean 降到 352 ms，实例数不变：硬件换代比软件改造便宜。TensorRT + ONNX + MPS 再进一步到 111.6 RPS、2 台实例（-88%），边际收益是 2 台 GPU 与 +21% 吞吐，边际成本是每个微调周期一次 ONNX 重导出，生产选择了 75% 那条。分离模型侧同样务实：Sortformer warmup 后 mean 309→239 ms、σ 12.62→7.13 ms，60 秒录音拆 4×15 秒块，实时因子 0.016x。


## 实践启示

1. **先测单请求算力占比，再决定扩容方式**：nvidia-smi 观察单请求 SM 占用；15–20% 量级走 MPS 软分区，单请求已占满 SM 的模型上软分区只会加剧争抢。
2. **用 SLA 扫描定运行点，不用压测峰值**：并发 1→100、每档 5 轮均值，取"mean < 650 ms 且 p99 < 1,000 ms 的最后一个档位"为容量锚点——本案锚点是 4 GPU × 92.1 RPS，而非峰值 106 RPS。
3. **model instance 与 MPS 分区保持一对一**：`MPS_INSTANCE_COUNT` 同时决定两者（`auto_config.py` 生成），preferred batch sizes [4, 8, 16] 与分区数对齐，避免调度粒度错位。
4. **CUDA graph 预热信封必须覆盖全部生产形状**：预烧线上会出现的所有时长 × batch 组合（5/15/30/45/60 秒 + batch 2 × 61 秒）；信封外的形状在 MPS 下重捕 graph 会拖垮兄弟实例，eager 回退是保命项而非可选项。
5. **把 wedge sentinel 接进健康检查**：CUDA stream 探针失败即写 sentinel 文件，/health 同时查存活与 sentinel，让不可恢复实例被编排层快速替换而不是拖垮整卡。
6. **88% 方案留作二阶段**：TensorRT + ONNX 路线的 +21% 吞吐（111.6 vs 92.1 RPS）要付"每个微调周期重导出一次"的运维税；先跑通 75% 路径，确有容量压力再切换。

## 相关实体

- [[entities/vllm|vLLM 高吞吐推理]]
- [[entities/gpu-virtualization-using-mig-technology-on-amazon-sagemaker-hyperpod|MIG GPU 虚拟化]]
- [[concepts/inference-optimization|推理优化]]
- [[entities/meta-muse-voice-transcribe-streaming-asr-2026|Meta Muse 流式 ASR]]

→ [[raw/articles/reduce-asr-inference-costs-by-75-with-nvidia-mps-on-amazon-e|原文存档]]
