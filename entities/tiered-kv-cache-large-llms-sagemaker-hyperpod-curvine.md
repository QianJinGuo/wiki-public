---
title: "Tiered KV Cache for Large LLMs on SageMaker HyperPod with Curvine"
created: 2026-08-13
updated: 2026-09-12
type: entity
tags: [llm, inference, kv-cache, optimization, vllm, aws, sagemaker, hyperpod]
sources: [raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Tiered KV Cache for Large LLMs on SageMaker HyperPod with Curvine

## 核心洞察：把 KV cache 从单 Pod 隔离扩展到共享三级层级

大规模 LLM 推理的 KV cache 本质上是一个内存权衡问题：要么为增长的 KV cache 购买超规格 GPU 实例，要么接受相同 prompt 反复重算导致的慢 TTFT。vLLM 的 prefix caching 只能在同一副本内复用 cache，水平扩展的 vLLM 副本各自维护隔离的 cache，路由到不同副本等于冷启动。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

本文的核心方案是把 cache 层级从 GPU/CPU 内存扩展到共享分布式 NVMe 池，构建三级层级（L0 GPU HBM → L1 CPU/主机内存 → L2 Curvine 共享跨节点 cache），并叠加 cache-aware 请求路由。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

## 三级 cache 层级架构

- **L0 – GPU prefix cache**：vLLM 原生 paged-attention 层，持有最热的 KV block。48GB GPU 上 7B bf16 模型权重约 14GB，剩余 30GB+ 给 KV；32B 模型权重 64GB 单卡放不下，分片后 KV 余量极小，并发下快速填满并驱逐。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]
- **L1 – CPU 内存 offload**：GPU block 被驱逐时由 LMCache 在 host DRAM 接住，运行在每个推理 Pod 内，由 SageMaker HyperPod Inference Operator 的 `enableL1Cache` 配置自动管理。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]
- **L2 – Curvine 共享跨节点 cache**：轻量级分布式 cache 文件系统（curvineio.github.io），作为共享 L2 层（GPU→CPU→共享 NVMe），让 KV cache 能以接近本地磁盘的速度跨副本复用。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

## 实测效果

测试部署达到：最高 100% 跨 Pod cache 命中率、最高 2.7x TTFT 改善、跨节点 L2 读延迟约 56ms（约 1,900-token prompt）。此前需要 P5 实例的工作负载可运行在更低成本的 G6e 实例上，降低每端点成本。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

## 架构决策要点

- 三层 cache 层级（L0/L1/L2）叠加 cache-aware 请求路由是核心思想：拒绝"每个 vLLM 副本各自隔离、无共享"的默认状态。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]
- 实现路径：启用 HyperPod Tiered Storage → 在节点本地 NVMe 部署 Curvine workers → patch Inference Operator 支持 filesystem-backed L2。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

## 深度分析

### 为什么约束落在 KV cache，而不是权重或算力

解码每步都要重读全部历史 K/V，KV cache 随 prompt 长度与并发线性膨胀（单 token 约占 2 × 层数 × KV 头数 × head_dim × dtype 字节）。Qwen2-7B 在 1,900 token 时写出 0.10 GB、跨节点读约 56 ms；32B 权重 bf16 约 64 GB，分片后留给 KV 的 HBM 极小。约束因此从算力转向 HBM 容量与带宽。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

浪费源于隔离：各 vLLM 副本各持私有 prefix cache，同一 system prompt 被反复 prefill，路由到新副本等同冷启动。池化容量一直存在，只是不可跨副本寻址；本架构把它变成共享资源——对比 [[entities/llm-prefix-caching-comprehensive-guide|prefix caching]] 只管单副本内的前缀复用。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

### 三级层级的权衡：延迟、容量与命中率经济学

层级越深容量越大、延迟越高：L0 亚毫秒、L1 毫秒级、跨 Pod 的 L2 数十毫秒，但都远低于重跑 prefill。命中率决定收益：共享前导 token 高于约 40% 才见效；1,000 token 以下 L2 往返与重算等价，加速比从 1,000 token 的 1.7× 升到 2,500 token 的 2.7×，3,000 token 后回落至 2.2×，但每请求绝对节省仍升至约 490 ms。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

Curvine 只做缓存/数据面：Master 只存 metadata（落 EBS），Worker 用本地 NVMe 存块、按热度 promote/demote，miss 才回源 UFS。故系统能容忍 eviction 与节点故障，不必为共享层付强一致与持久化的代价。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

### 该部署强制你做出的架构决策

cache key 跨副本后即成分布式契约：相同 prefix 需在所有副本生成相同 entry，路由层（prefix-aware 前缀树、kv-aware 查各 worker cache 状态）把请求送往最可能命中的副本——路由是收益的前置条件。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

其余强制决策：一致性与失效（手工 patch 被 reconcile 覆盖，生产需 MutatingWebhook/Kyverno 自动注入）；故障域（元数据在 EBS、数据在本地 NVMe）；容量规划（`InstanceMemoryAllocationPercentage` 从 20 起按吞吐与命中率上调，L2 随节点数线性扩到数百 GB）；与既有优化的交互——PagedAttention 定 L0 的 block 粒度、LMCache 默认 256-token chunk 定复用单元、continuous batching 在高并发下更快填满 L0 从而加重对 L1/L2 的依赖。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

### 实测数字该怎么读

100% 跨 Pod 命中即 Pod B 完整复用 Pod A 的 1,925 token、完全跳过 prefill（774 ms 冷启动降至 287 ms）。加速比环境相关：多轮对话在 L40S/L4 上仅 1.30×，因其 prefill 本就快。写入几乎免费（同节点约 9.6 GB/s），成本在跨节点读（约 1.8 GB/s），故前缀感知路由把 L2 读换成 L0 命中，即把数十毫秒换成亚毫秒。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

反之，短上下文（<1,000 token）、低并发或要求隔离时 L2 不划算：往返与重算相当，却引入跨节点依赖、共享故障域与运维复杂度。同一 L2 也支撑 PD 拆分（见 [[entities/disaggregated-prefill-decode-llm-inference-sagemaker|PD 拆分]]），可用 `LMCACHE_REMOTE_URL` 覆盖扩展多节点大模型，但受跨节点延迟约束。^[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine.md]

## 实践启示

1. **先量命中率再上共享层**：共享前导 token（约 >40%）与 prompt 长度（约 >1,000 token）双双过线时 L2 才有正收益，否则先调好 L0/L1。
2. **分层起步、参数保守**：`InstanceMemoryAllocationPercentage` 从 20 起步按吞吐与命中率递增；L2 复用节点自带 NVMe，容量随节点数线性增长，无需额外采购存储。
3. **路由等于一半收益**：多轮/共享 system prompt 用 prefix-aware，长文档/长会话用 kv-aware，让请求落到已持有 cache 的副本，把 L2 读换成 L0 命中。
4. **补丁配置化**：手工 patch 会被 reconcile 覆盖，生产用 MutatingWebhook/Kyverno 注入 Curvine 卷/挂载与 `LMCACHE_REMOTE_URL=fs://localhost:0/mnt/curvine/l2cache/`。
5. **划清故障与一致性边界**：L2 是可重算缓存而非持久层，别叠加强一致或持久化假设；整体取舍参考 [[concepts/inference-optimization|推理优化]]。
6. **用 PD 拆分复用同一套栈**：prefill/decode 分离同样以这层 L2 交换 KV，可用同一 `LMCACHE_REMOTE_URL` 扩展多节点大模型，代价是跨节点延迟。

## 相关实体

- [[entities/vllm|vLLM]]
- [[entities/deepseek-cost-migration-system-layer-kv-cache-harness|DeepSeek 系统层 KV cache 成本迁移]]
- [[entities/ai-agent-storage-curvine-eks-2026|Curvine 存储选型（EKS 万级 Agent）]]
- [[entities/sagemaker-inference-observability-cloudwatch-insights|SageMaker 推理可观测性]]
- [[concepts/inference-optimization|推理优化]]
- GPU 优化

→ [[raw/articles/tiered-kv-cache-for-large-llms-on-amazon-sagemaker-hyperpod-with-curvine|原文存档]]
