---
title: 推理系统优化
created: 2026-04-30
updated: 2026-08-01
type: concept
tags: [inference, optimization, kv-cache, pd-separation, coding-agent, batching, speculative-decoding, quantization]
related:
  - [[entities/vllm-v0-to-v1-correctness-before-corrections|vLLM]]
  - [[entities/sglang|SGLang]]
  - [[entities/lightseek-tokenspeed|LightSeek TokenSpeed]]
sources: ['raw/articles/glm5-scaling-pain-inference', 'raw/articles/vllm-v0-to-v1-correctness-before-corrections', 'raw/articles/lightseek-tokenspeed']
confidence: high
---
# 推理系统优化
## 概述
大模型推理系统在 Coding Agent 场景（高并发、长上下文、复杂工具调用）下面临的特殊挑战及优化方案。推理优化不是单一技术的改进，而是涉及内存管理、调度策略、计算精度等多个维度的系统性工程。在 Agent 场景下，每一次工具调用（tool-call）和上下文切换都对延迟和吞吐量提出了严格要求，优化结果直接影响 Agent 的响应速度和可靠性。

## 相关查询

- [[queries/chrome-gemini-nano-pied-piper-distributed-ai-nodes|分布式 AI 节点与浏览器集成]] — Chrome Gemini Nano 静默部署、浏览器本地 AI 节点与边缘推理基础设施

## 核心问题：Scaling Pain
当大模型从简单对话转向复杂 Coding Agent 任务时，推理基础设施承受数亿次调用，一些在标准推理环境下不存在的问题开始集中出现：
- **乱码**（garbled output）
- **复读**（repetition）
- **生僻字**（rare character）

这些问题只在高并发、长上下文场景下触发，难以稳定复现。问题的根因通常不在模型本身，而在于推理链路中高负载下的状态管理缺陷。^[raw/articles/glm5-scaling-pain-inference.md]

## KV-Cache 管理：PagedAttention 与 RadixAttention

### PagedAttention（vLLM）
PagedAttention 是 vLLM 提出的 KV-Cache 管理方案，将 KV Cache 按固定大小的 "page" 进行组织，类似于操作系统的虚拟内存分页管理。传统推理引擎在预分配 KV Cache 时需要按最大上下文长度预留连续显存，造成大量浪费。PagedAttention 通过动态分配离散显存块解决了这一问题：

- **显存利用率提升**：无需为每个请求预留完整上下文长度的连续显存
- **支持更长的上下文**：上下文长度不再受限于预分配的连续显存大小
- **KV Cache 共享**：多个请求可以共享相同的 KV Cache page，支持前缀复用

PagedAttention 是 [[entities/vllm-v0-to-v1-correctness-before-corrections|vLLM]] 引擎的核心内存管理基础，在 vLLM V1 中还引入了 prefix caching 优化，但需要注意其可能引入的非确定性执行路径。^[raw/articles/vllm-v0-to-v1-correctness-before-corrections.md]

### RadixAttention（SGLang）
RadixAttention 是 [[entities/sglang|SGLang]] 框架实现的 KV Cache 复用策略，专门针对 Agent 场景中常见的前缀复用模式进行了优化。在多轮对话或重复工具调用的场景中，用户 prompt 或系统 prompt 的 KV Cache 可以被多个请求共享：

- **前缀共享**：相同前缀的请求可以复用已计算的 KV Cache
- **LRU 淘汰**：RadixAttention 维护一棵 Radix Tree，按 Least Recently Used 策略淘汰冷数据
- **Async Swap**：支持 KV Cache 在 GPU 和 CPU 内存之间的异步换入换出

在 GLM-5 的 Coding Agent 场景中，[[entities/sglang|SGLang]] 的 HiCache（多级 KV Cache）机制通过 RadixAttention 实现历史前缀缓存的快速加载，显著降低了首 token 延迟（TTFT）。^[raw/articles/glm5-scaling-pain-inference.md]

## 连续批处理（Continuous Batching）

连续批处理是推理引擎提高 GPU 利用率的核心调度策略。传统的静态批处理（Static Batching）要求同一批次内所有请求同时完成，造成"气泡"——即部分请求完成后 GPU 处于空闲状态等待最慢请求。

连续批处理（又称 Iteration-level Batching 或 Dynamic Batching）通过在每个 decode step 后动态加入新请求、移除已完成请求来解决这一问题：

- **GPU 利用率提升**：请求完成即退出，新请求立即加入，消除了静态批处理的"气泡"
- **吞吐提升**：在同样硬件条件下支持更多并发请求
- **延迟权衡**：长上下文请求可能因为短请求的插队而面临排队延迟增加

在 [[entities/lightseek-tokenspeed|LightSeek TokenSpeed]] 的设计中，连续批处理与投机解码、KV Cache 形成了协同优化，TokenSpeed 声称在这三者结合下达到了优于 TensorRT-LLM 的性能表现。^[raw/articles/lightseek-tokenspeed.md]

## 投机解码（Speculative Decoding）

投机解码通过一个小模型（草稿模型/Draft Model）先生成若干候选 token，再由大模型（目标模型）并行验证，从而加速 decode 阶段。关键指标包括：

- **spec_accept_length**：目标模型连续接受的 draft token 前缀长度
- **spec_accept_rate**：draft token 被接受的比例

### 异常检测创新
[[entities/sglang|SGLang]] 团队发现，投机采样指标可以反过来用于异常检测：^[raw/articles/glm5-scaling-pain-inference.md]

- **乱码/生僻字**：通常伴随极低的 `spec_accept_length`（< 1.4），说明草稿模型和目标模型的 KV Cache 状态存在显著偏差
- **复读**：通常伴随偏高的 `spec_accept_rate`（> 0.96），损坏的 KV Cache 使注意力模式退化，生成推向高置信度的重复循环

当 `spec_accept_length < 1.4` 且生成长度 > 128 token，或 `spec_accept_rate > 0.96` 时，系统主动中止生成并重试。

### 对工具调用的影响
投机解码会显著影响 tool-call 的延迟稳定性。由于草稿模型生成的 tool-call JSON 可能被目标模型拒绝重算，导致端到端延迟不可预测。在高可靠性的 Agent 生产环境中，需要评估是否启用投机解码以及接受阈值。

## 前缀缓存（Prefix Caching）

前缀缓存通过识别和复用相同 prompt 前缀的 KV Cache 来加速推理。在 Agent 场景中，这尤为重要：

- **系统 prompt 复用**：相同的系统指令、few-shot examples 可以被所有请求复用
- **工具描述复用**：tool schema 定义通常在多轮对话中保持不变
- **用户意图前缀**：在复杂任务中，用户请求可能有共同的前缀结构

但 [[entities/vllm-v0-to-v1-correctness-before-corrections|vLLM V1 的 prefix caching]] 默认开启，在 RL 训练等需要确定性执行路径的场景中可能引入非确定性：相同 `input_ids` 可能因为 cache 命中状态不同而走不同的执行路径。对于依赖精确 token-level logprob 的 RL 训练场景，建议关闭 `enable_prefix_caching: false`。^[raw/articles/vllm-v0-to-v1-correctness-before-corrections.md]

## PD 分离（Prefill/Decode Disaggregation）

PD 分离将 Prefill 阶段（计算密集型，负责处理输入 prompt）和 Decode 阶段（内存带宽密集型，负责自回归生成）部署在不同的 GPU 资源池中。这是解决长上下文场景下 Prefill 排队导致 TTFT（Time To First Token）过高的核心方案。^[raw/articles/glm5-scaling-pain-inference.md]

### KV Cache 竞态问题
PD 分离架构引入了新的竞态条件。[[entities/sglang|SGLang]] 团队在 GLM-5 排查过程中发现：

1. Decode 侧触发 Abort 后，KV Cache 被回收并分配给新请求
2. 但 Prefill 侧的 RDMA 写入尚未同步取消，继续写入已被复用的显存地址
3. 新请求的 KV Cache 被覆盖，导致乱码输出

**修复方案**：在请求终止与 KV Cache 写入完成之间建立显式同步关系：
- Decode 触发 Abort 后向 Prefill 侧发送通知
- Prefill 仅在所有已提交写入完成后才返回"可释放"信号
- Decode 仅在收到确认后才允许回收并复用 KV Cache 槽位

**效果**：异常率从万分之十几降至万分之三以下。^[raw/articles/glm5-scaling-pain-inference.md]

### LayerSplit：KV Cache 分层存储
每张 GPU 保存全部层 KV Cache 的传统方式在长上下文场景下成为显存瓶颈。LayerSplit 方案让每张 GPU 仅持有部分层的 KV Cache，通过广播协同：

- 持有某一层 KV Cache 的 rank 在计算前将该层广播给其他 rank
- 通信与计算重叠，Indexer Cache 广播（KV Cache 的 1/8）作为额外开销
- 在 Cache 命中率 90%、上下文长度 40k-120k 条件下，吞吐提升 **10%-132%**

## 量化（INT8/FP8）

量化通过降低权重和激活的数值精度来减少显存占用和加速计算。在 Agent 推理场景中，主要关注：

### INT8 量化
- **Weight Only 量化**：仅对权重进行 INT8 量化，激活保持 FP16/BF16
- **kv_int8 量化**：对 KV Cache 进行 INT8 量化，显著降低长上下文场景的显存压力
- **延迟影响**：INT8 的 Tensor Core 加速需要足够大的 batch size 才能体现，小 batch 下可能因量化开销反而变慢

### FP8 量化（NVIDIA Blackwell 架构）
[[entities/lightseek-tokenspeed|LightSeek TokenSpeed]] 提及的 MLA（Multi-head Latent Attention）优化已在 vLLM 主干实现，配合 FP8 可在 Blackwell 架构上获得显著加速。FP8 量化的优势在于：

- 与 FP16/BF16 的数值范围接近，精度损失更小
- Tensor Core 原生支持，硬件加速效率高
- 对 KV Cache 量化友好，长上下文场景收益更大

参见 [[entities/tliveomni-vllm-quantization|vLLM 量化实践]]。

## 对 Agent 上下文窗口与工具调用延迟的影响

### 上下文窗口
Agent 场景的核心特征是超长上下文（平均 > 70K tokens）和高前缀复用率。各项优化对上下文窗口的影响：

| 优化技术 | 上下文窗口影响 | 机制 |
|---------|--------------|------|
| PagedAttention | 有效扩展 | 消除连续显存预分配限制 |
| RadixAttention | 加速长上下文 | 前缀 KV Cache 复用 |
| LayerSplit | 间接扩展 | 降低单卡显存压力 |
| KV Cache 量化 | 有效扩展 | kv_int8 减少显存占用 |
| PD 分离 | 降低 TTFT | Prefill 排队减少 |

### 工具调用延迟
Tool-call 场景对推理系统有特殊要求：

1. **JSON 输出稳定性**：乱码/复读会导致 tool-call JSON 解析失败，需要投机解码监控作为异常信号
2. **多轮工具调用**：连续 tool-call 依赖上下文连贯性，KV Cache 损坏会导致工具参数错误
3. **尾延迟敏感**：用户对 tool-call 响应时间的容忍度低于普通文本生成

[[entities/lightseek-tokenspeed|LightSeek TokenSpeed]] 声称在 PD disaggregation 场景下有更大优化空间，配合 SMG（低开销 CPU 侧请求入口）和 MLA kernel 优化，可降低首 token 延迟和工具调用的端到端延迟。^[raw/articles/lightseek-tokenspeed.md]

## 关键技术方案汇总

| 优化项 | 问题 | 方案 | 效果 |
|--------|------|------|------|
| **PD 分离架构 KV Cache 竞态修复** | Abort 信号未传 Prefill 侧，RDMA 写入跨显存复用边界覆盖 | 建立 Abort 与 KV Cache 回收的显式时序同步 | 异常率从万分之一降至万分之一以下 |
| **HiCache 加载时序修复** | Load Stream 与 Forward Stream 依赖未显式同步，Read-before-Ready | Indexer 算子启动前引入同步点 | 异常完全消失（已提交 [[entities/sglang|SGLang]] PR #22811） |
| **LayerSplit KV Cache 分层存储** | 每张 GPU 保存全部层 KV Cache，显存成为瓶颈 | 每张 GPU 仅持有部分层，通过广播协同 | 吞吐提升 10%-132%（上下文越长收益越大）|
| **vLLM V0→V1 logprob 对齐** | V1 默认 raw logprobs 与训练期望的 processed logprobs 语义不同 | 显式设置 `logprobs_mode: processed_logprobs`，关闭 prefix caching | RL 训练曲线系统性偏差修复 |
| **TokenSpeed PD 优化** | TensorRT-LLM 在 Agent 场景下调度开销大 | C++ FSM 控制平面 + MLA kernel + 连续批处理 | 特定场景性能优于 TensorRT-LLM |

## 异常检测创新
**投机采样指标作为异常信号**：^[raw/articles/glm5-scaling-pain-inference.md]
- `spec_accept_length`（草稿模型连续被接受的 token 前缀长度）极低 → 乱码/生僻字
- `spec_accept_rate`（草稿 token 被接受比例）偏高 → 复读
当 spec_accept_length < 1.4 且生成长度 > 128 token，或 spec_accept_rate > 0.96 时，主动中止生成并重试。

## 相关页面
- [[entities/glm5-scaling-pain|GLM-5 Scaling Pain 推理复盘]] — 高并发 Coding Agent 场景的推理竞态问题完整复盘
- [[entities/glm5-scaling-pain-inference|GLM-5 Scaling Pain 推理优化详解]] — GLM-5 推理系统在高并发场景下的 KV Cache 竞态、HiCache 时序问题及 LayerSplit 优化方案
- [[entities/sglang|SGLang]] — 开源推理框架，HiCache/KV Cache 修复已贡献回社区
- [[entities/vllm-v0-to-v1-correctness-before-corrections|vLLM V0→V1 迁移]] — logprob 语义差异与 prefix caching 的影响
- [[entities/lightseek-tokenspeed|LightSeek TokenSpeed]] — 投机解码 + KV Cache + 连续批处理的协同优化
- [[entities/servicenow-vllm-correctness|ServiceNow vLLM Correctness]] — vLLM V0→V1 迁移中的 backend 问题诊断
- [[entities/agent-memory-architecture|Agent Memory 架构]] — Agent 场景下的记忆系统设计与推理引擎的交互

## 新增关联实体
- [[entities/elasticpp重塑elasticsearch查询性能的c内核引擎]]

## 关联实体

**上游依赖**:
- [[entities/vllm-v0-to-v1-correctness-before-corrections]] — 提供基础理论/方法
- [[entities/sglang]] — 提供基础理论/方法
- [[entities/lightseek-tokenspeed]] — 提供基础理论/方法

**下游应用**:
- [[entities/sglang]] — 具体应用场景
- [[entities/vllm-v0-to-v1-correctness-before-corrections]] — 具体应用场景
- [[entities/sglang]] — 具体应用场景

**平行协作**:
- [[entities/glm5-scaling-pain]] — 替代/补充方案
- [[entities/glm5-scaling-pain-inference]] — 替代/补充方案
- [[entities/sglang]] — 替代/补充方案

## 所属 MOC

- [[moc/ai-misc-topics-frontier|Ai Misc Topics Frontier]]
- [[moc/amazon-aws-ai|Amazon Aws Ai]]
- [[moc/layer-1-llm-principles|Layer 1 Llm Principles]]
- [[moc/wiki-master-map|Wiki Master Map]]
