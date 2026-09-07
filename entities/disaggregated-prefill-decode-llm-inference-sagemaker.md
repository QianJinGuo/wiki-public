---
title: "Disaggregated Prefill and Decode for LLM Inference on SageMaker HyperPod"
created: 2026-07-11
updated: 2026-09-07
type: entity
tags: [aws, sagemaker, llm-inference, disaggregated-inference, vllm, hyperpod, inference-optimization]
sources: [raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Disaggregated Prefill and Decode for LLM Inference on SageMaker HyperPod

AWS 在 SageMaker HyperPod 上实现了 Disaggregated Prefill and Decode（DPD），将 LLM 推理的 prefill（计算密集型）和 decode（内存密集型）阶段分离到不同的 GPU 池，通过 Elastic Fabric Adapter（EFA）与 RDMA 互联。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md]

## 技术原理

当 prefill 和 decode 共享 GPU 时，长 prompt 会阻塞所有并发请求的 token 生成。DPD 通过拆分两个阶段到独立 GPU 池来消除干扰，可为每阶段分配不同的并行策略，独立调优首 token 时延（TTFT）和 token 间时延（ITL），更可靠地控制尾部时延。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md:12-13]

## 与 vLLM 集成

在 SageMaker HyperPod 上使用 HyperPod Inference Operator 部署 vLLM，实现多节点编排和路由优化。适用于大规模 LLM 部署场景。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md:13-14]

## 深度分析

### 1. DPD 的适用边界与路由决策

DPD 并非适用于所有场景 — 其核心收益集中在长上下文、高并发、流式响应的推理负载上。AWS 的实现通过一个智能路由器（Intelligent Router）对每个请求做条件路由：当输入 token 数超过可配置阈值（默认 4,096）时走拆分解码路径，短请求则直接发送到 decoder 避免 KV 缓存跨节点传输的开销。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md:16-20] 这种设计保证了短请求不会因为拆分的固定开销而受损，而长请求能够获得稳定的 token 间延迟。在请求低于 4,096 token 的场景下，拆分的 KV 传输开销超过了隔离收益，路由器自动绕过 prefiller。

### 2. 四层 KV 传输栈与硬件依赖

DPD 的核心技术挑战是如何高效地在 prefiller 和 decoder 之间传递 KV 缓存。AWS 的解决方案是构建了一个四层传输栈：LMCache PD → NIXL → libfabric → EFA。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md:45-51] LMCache 负责 orchestrator 层的缓存管理，NIXL 提供跨 GPU、CPU 和远程 peer 的统一内存抽象，libfabric 暴露 EFA 的内核旁路 GPU-Direct RDMA 能力。实测在 ml.p5.48xlarge（8×H100 80GB，3,200 Gbps EFA）上，8,000 token 的 KV 传输仅需个位数毫秒。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md:51-51] 这意味着 KV 传输开销相对于 prefill 计算时间几乎可以忽略。

### 3. 性能基准与收益量化

Benchmark 数据揭示了 DPD 在不同并发度下的具体收益。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md:263-275] 在 4,096 输入 token、256 输出 token 的标准负载下：
- **Per-token 延迟（TPOT）**：并发度从 8 增至 32 时，H100 上改善 22%→66%，H200 上改善 28%→48%
- **吞吐量**：H100 上提升最高 35%，H200 上最高 64%
- **端到端 P50 延迟**：H100 改善 14-32%，H200 改善 29-41%

TPOT 随并发度保持稳定的原因是 decoder 不再受 prefill 干扰 — decoder 运行完整的 CUDA graph，不会因为突然插入的 prefill 计算而打断 token 的流式生成。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md:47-48] 代价是首 token 延迟（TTFT）有小幅增加，因为 KV 缓存传输增加了固定的延迟开销。

### 4. 弹性扩缩与部署形态

DPD 当前支持单个 decoder replica 搭配多个 prefiller replica 的拓扑。^[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema.md:253-259] 推荐起始比例为 1:1（prefiller:decoder），当 TTFT 攀升而 TPOT 稳定时说明 prefill 成为瓶颈，可扩展至 2:1 或 3:1。多 prefiller 场景下支持四种路由策略：`prefixaware`、`kvaware`、`session`、`roundrobin`，其中 `kvaware` 对重复前缀（系统提示词、多轮历史）的缓存命中率最优。

### 5. 对推理基础设施设计的启示

DPD 架构正在改变推理基础设施的设计范式。传统"单节点尽可能多塞模型"的思路被"为不同阶段分配差异化资源"的范式取代。Prefill 节点需要高计算密度（NVLink + 高带宽显存），decode 节点需要高内存带宽（HBM3/HBM3e）和低延迟互联。考虑到 P5/P6 实例系列的 EFA RDMA 要求，同一可用区内部署的约束意味着实际部署需要仔细规划集群拓扑。

## 实践启示

1. **优先评估工作负载特征**：DPS 并非通用方案。当平均输入 token 数低于 4,096、并发度不高或非流式场景时，colocated 方案更简单、成本更低。上线前应使用实际负载（而非合成 benchmark）评估拆分阈值和路由比例。

2. **从 1:1 比例起步，监控双指标**：建议从 1:1 prefiller:decoder 起步，同时监控 TTFT 和 TPOT 两个指标。TTFT 升高 → 增加 prefiller；TPOT 升高 → 检查 decoder 饱和度、增加 PD_BUFFER_SIZE 或降低 max-model-len。

3. **利用 KV-aware 路由优化缓存效率**：对于有重复前缀负载（如多轮对话、RAG 系统提示），启用 `kvaware` 路由策略可显著提升 L1 CPU 缓存命中率，降低 TTFT。

4. **警惕 EFA 网络约束**：所有 DPD 节点必须在同一可用区，且依赖 RDMA-capable EFA。P5/P6 实例系列之外的选择（如 G6/G6e/G7e）在 multi-GPU 场景下受 PCIe 瓶颈限制，不适合 DPD 部署。

5. **观测指标先行**：部署前确保 HyperPod Observability 已启用。重点关注 router 日志中的 `disaggregate=True/False` 分布、`prefill time (TTFT)` 和 `to decoder` 时间戳，这些是诊断拆分行为正确性的第一手数据。

→ [[raw/articles/disaggregated-prefill-and-decode-for-llm-inference-on-sagema|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

