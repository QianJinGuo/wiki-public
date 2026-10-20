---
title: "vLLM 分离式推理的 push 式 KV 传输：TTFT 降约 3 倍、TPOT 保持平稳"
created: 2026-10-20
updated: 2026-10-20
type: entity
tags: [vllm, kv-cache, disaggregated-inference, nixl, latency, inference-optimization]
sources: [raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026]
confidence: 0.65
---

# vLLM 分离式推理的 push 式 KV 传输

分离式（disaggregated）部署把 Prefill 与 Decode 拆到不同节点，代价是**每个请求都要把 KV cache 从 prefill 节点搬到 decode 节点**。AWS 这篇工程分析对比了两种搬运语义：**pull（decode 节点主动拉）** 与 **push（prefill 节点主动推）**，结论是 push 在高负载长上下文下显著更优。^[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026.md]

## 关键数据（1P1D，128 输出 token，decode 节点关闭 prefix caching）

- **TTFT**：低负载短提示下两者几乎相同；随 QPS 与输入长度上升，pull 在 8 QPS / 16k 输入退化到 **约 1744 ms**，push 保持在 **约 540 ms**，约 **3 倍**改善。^[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026.md]
- **TPOT**：轻载时两者接近（5.6–5.8 ms）；高负载长上下文时 pull 升到 **约 10.4 ms**，push 稳定在 **约 7 ms**。

## 机制解释：谁在关键路径上

差距的根因是**描述符提交发生在哪个线程**：pull 模式在 decode worker 上**内联**提交 NIXL 描述符，且位于引擎循环中同步执行，直接阻塞 token 生成；push 模式把描述符准备与传输提交移到**后台线程**，脱离关键路径，因此每 token 延迟在高并发长上下文下仍保持平稳。^[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026.md]

这与 [[concepts/inference-optimization|推理优化]] 的一条通用规律同构：**延迟优化的核心不是「搬得更快」，而是「别把搬运放在生成的关键路径上」**。同为 KV 层优化，[[entities/tiered-kv-cache-large-llms-sagemaker-hyperpod-curvine|分层 KV cache]] 解决容量，push 模式解决时序。

## 实验设计的严谨点

对比时特意在 decode 节点**关闭 prefix caching**，否则重叠提示可以复用本地缓存从而绕过传输，导致对比失真——这是一个容易踩的评测陷阱：**缓存复用会掩盖传输开销**。^[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026.md]

## 适用判断

- 何时值得：PD 分离 + 长上下文 + 中高 QPS（≥4 QPS）场景，pull 模式的同步描述符提交会成为 TTFT/TPOT 的主要瓶颈。
- 何时无差别：低负载、短提示，两种模式几乎等价，改用 push 只增加实现复杂度。

## 局限

原文抓取不完整（仅保留 push/pull 对比与图表解读段落），缺少 push 模式在极端并发下的背压/内存放大分析，以及与 NIXL 之外的传输后端（如 EFA/RDMA 自定义路径）的对比。相关实体：[[entities/disaggregated-prefill-decode-llm-inference-sagemaker|SageMaker 分离式 PD 推理]]、[[entities/750b-moe-pd-disaggregation-aws-efa-vs-roce|750B MoE 的 PD 分离与 EFA/RoCE 对比]]。

## 关联

- [[concepts/inference-optimization|推理优化]]
- [[entities/ai-infra-llm-efficient-inference-vllm|vLLM 高效推理栈]]
- [[entities/disaggregated-prefill-decode-llm-inference-sagemaker|分离式 Prefill/Decode 推理]]
- [[entities/tiered-kv-cache-large-llms-sagemaker-hyperpod-curvine|分层 KV Cache]]

→ [[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026|原文存档]]
