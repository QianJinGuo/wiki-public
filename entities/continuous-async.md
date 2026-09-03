---
title: "Unlocking asynchronicity in continuous batching"
type: entity
tags: [llm, inference, cuda, huggingface, performance-optimization]
created: 2026-05-17
updated: 2026-08-30
review_value: 8
sources: []
review_confidence: 9
source_url:

---
> 来源：[[raw/articles/continuous-async.md|原文存档]]

## 摘要
HuggingFace 深度技术文章，系统性地解析了连续批处理（Continuous Batching）中同步瓶颈的根源，并提出基于 CUDA streams 和 events 的异步优化方案。核心发现：同步模式下 CPU 和 GPU 交替空闲，造成近 24% 的 GPU 空闲时间；通过异步化将两者解耦后，GPU 利用率从 76% 提升至 99.4%，生成速度提升 22%。

## 核心要点
- 连续批处理默认是同步的：CPU 准备新批次时 GPU 空闲，GPU 计算时 CPU 等待，两者从不同时工作
- 实验数据：生成 8K tokens、batch size 32、8B 模型，总时间 300.6 秒，24% 为 GPU 空闲
- 使用 CUDA streams 将 H2D 传输、计算、D2H 传输分配到独立 stream，实现流水线并行
- CUDA events 提供纯 GPU 侧的同步语义，CPU 调用 `wait()` 后立即返回，不阻塞
- 双 buffer 槽位设计解决数据竞争：slot A 和 slot B 交替使用，CUDA Graphs 通过 memory pool 共享 VRAM
- Carry-over 机制处理跨 batch 的 token 依赖：用占位符构建 batch N+1 输入，batch N 完成后填充实际 token
- 异步化零成本：无需新 kernel 或模型修改，仅通过流管理实现

## 深度分析

### 同步批处理的效率陷阱
连续批处理通过动态打包请求显著提升了 GPU 利用率，解决了传统静态批处理的 padding 浪费问题。但它的默认实现是同步的——CPU 和 GPU 严格串行交替。在高频推理场景下（每秒数百步生成），这些空闲间隙累积成显著的效率损失。^[raw/articles/continuous-async.md]

HuggingFace 的实验数据揭示了这一问题的严重性：生成 8K tokens、batch size 32、8B 模型，总时间 300.6 秒，其中 24% 时间为空闲 GPU。这意味着如果能消除 CPU 开销，理论上可获得 24% 的免费加速——无需任何新 kernel 或模型修改。^[raw/articles/continuous-async.md]

这种"CPU 做完 GPU 等、GPU 做完 CPU 等"的模式在单次 forward pass 中影响不大，但在 continuous batching 循环中每秒执行数百步时，空闲间隙的累积效应就变得不可忽视。文章将此称为"悲观视角"（GPU 浪费 24%）和"乐观视角"（可免费提速 24%）。

### CUDA Streams 与并发执行机制
CUDA stream 是理解异步批处理的关键基础设施。每个 stream 是 GPU 操作的顺序队列（kernel launch、memory copy、sync barrier），同一 stream 内操作串行执行，不同 stream 可并发。通过将 H2D 传输（Host-to-Device）、计算（forward pass + sampling）、D2H 传输（Device-to-Host）分配到三个独立 stream，可实现数据传输与计算的重叠执行。^[raw/articles/continuous-async.md]

但存在一个关键陷阱：PyTorch 的默认 stream 具有全局同步语义——任何操作调度到默认 stream 前，必须等待所有其他 stream 完成；反之亦然。这意味着如果不显式使用非默认 stream，所有异步努力都会被默认 stream 的隐式同步破坏。因此，异步批处理的第一步是确保所有 GPU 操作都调度到非默认 stream 上。^[raw/articles/continuous-async.md]

### CUDA Events 的同步语义与依赖编排
stream 之间的独立性既是优势也是问题——它们不知道彼此的存在，导致 compute stream 可能在 H2D 传输完成前就启动，D2H 可能在计算完成前就传输结果。CUDA event 是解决这一问题的机制：通过 `stream.record(event)` 在 stream 中插入标记，GPU 执行到该点时标记完成；通过 `stream.wait(event)` 让另一个 stream 阻塞直到该 event 被设置。关键是 `wait` 只阻塞 stream，不阻塞 CPU——CPU 调用立即返回。^[raw/articles/continuous-async.md]

这种纯 GPU 侧的同步使得 CPU 可以真正"放手"，让硬件自行管理依赖关系。实际的同步点只有一个：CPU 在 `d2h_done_event.synchronize()` 处阻塞等待 batch N 的输出，这是不可避免的（需要 CPU 采样 token 并更新状态），但仅占极小比例。

### 双 Buffer 与 CUDA Graphs 的协同设计
异步批处理需要在 GPU 处理 batch N 时准备 batch N+1 的输入，这引发两个核心挑战。^[raw/articles/continuous-async.md]

**数据竞争与双槽位**：batch N 和 batch N+1 不能共享同一内存区域，否则 GPU 可能读到部分覆写的数据。解决方案是使用两个独立的内存槽位（slot A 和 slot B），交替使用。代价是 RAM 和 VRAM 翻倍，但使用 FlashAttention 时（不需要 attention mask——最大的输入 tensor），这个 tradeoff 通常是值得的。

**CUDA Graphs 兼容性**：生产环境常使用 CUDA Graphs 加速推理（预录制的 CUDA 操作序列绑定特定内存地址）。双槽位需要两个 graph，但通过 memory pool 让多个 graph 共享池化内存，总 VRAM 接近单个 graph 的使用量。唯一约束是同一 pool 中的两个 graph 不能并发执行——由于 batch N 必须在 batch N+1 开始前完成，这一约束自然满足。

**Carry-over 机制**：跨 batch 的请求需要将 batch N 的输出 token 作为 batch N+1 的输入。由于 batch N 仍在计算，该 token 尚未产生，因此用 0 作为占位符构建 batch N+1 输入。batch N 完成后通过 carry-over 操作（选择、置零、截断、相加）填充实际 token，这四个操作足够轻量，可被 CUDA Graph 捕获。

## 实践启示
1. **推理优化应关注 CPU/GPU 协同**：即使 GPU 计算能力充足，CPU 侧的批次准备调度开销可能成为瓶颈。异步化将两者解耦，让 CPU 和 GPU 同时做有用工作。
2. **使用非默认 CUDA Stream 避免隐式同步**：在 PyTorch 中显式使用非默认 stream 处理异步操作。任何传输操作都必须是非阻塞的，否则默认 stream 的全局同步效应会破坏所有异步努力。
3. **双 buffer 槽位是异步推理的标准模式**：任何需要"一边执行一边准备"的场景，都应考虑双缓冲。空间换时间的 tradeoff 在 GPU 内存充足时通常值得。
4. **CUDA Graphs + Memory Pool 兼顾延迟与吞吐**：异步批处理提升吞吐量，CUDA Graphs 优化单批次延迟。通过 memory pool 可在保持 Graphs 效果的同时支持多 batch 并行，不必二选一。
5. **Profiling 先行**：在优化模型或硬件之前，先用 profiling 工具（如 HuggingFace 提供的 CPU/GPU activity timeline 脚本）确认是否存在 CPU-GPU 交替空闲问题。22% 的加速可能是"免费午餐"。
6. **注意 unavoidable sync point**：异步优化无法消除所有同步——CPU 仍需在每个 batch 结束时采样 token 并更新状态。这个不可避免的同步点是剩余 1% GPU 空闲的来源。

## 相关实体
- [[entities/ai-infra-llm-efficient-inference-vllm|LLM 高效推理 vLLM]]
- [[entities/agent-assisted-sglang-development-lmsys-2026-07|SGLang Agent 开发]]
- [[concepts/inference-optimization|推理优化]]

→ [[raw/articles/continuous-async.md|原文存档]]^[raw/articles/continuous-async.md]