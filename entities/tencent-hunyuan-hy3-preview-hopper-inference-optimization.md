---
title: "腾讯混元 Hy3 preview 在 Hopper 卡上的推理优化实践"
created: 2026-06-30
updated: 2026-09-07
type: entity
tags: [hunyuan, hy3, inference-optimization, hopper, moe, attention, quantization, sparse-attention, mtp, tpsp, fused-moe]
sources:
  - raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization
confidence: 0.9
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 腾讯混元 Hy3 preview 在 Hopper 卡上的推理优化实践

腾讯混元 AI Infra 推理团队对 Hy3 preview（GQA+MoE，295B/21B，256K 上下文）在 NVIDIA Hopper 96G 卡上进行了全栈推理优化，覆盖算子优化与融合、并行策略、多级缓存、MTP 异步调度、量化与稀疏五大维度。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

## 算子优化

**动态调度 Attention**：所有请求按统一 Tile 粒度拆分，贪心装桶算法实现极致均分，Task Assign 模块每次推理前生成专属任务映射表。单 batch 长文本加速 2.95x，混合长度 batch 加速 1.59x~1.76x。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

**双 BF16 Router GEMM**：FP32 权重拆分为高位 BF16 + 低位残差 BF16，两次 BF16 GEMM 线性组合，融合至单一 Kernel，全程无 HBM 往返。相比 FP32(cuBLAS) 2.86x~3.22x 加速。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

**FusedMoE 全流水线重构**：路由与索引预处理、Gate-Up GEMM、激活量化+Down GEMM、Top-K 加权聚合、PDL 无气泡串联。TP=8/EP=1 场景相比 vLLM CUTLASS/Triton、SGLang 1.5x~1.6x 加速。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

## 算子融合

**Fused Rope+Norm+Hadamard+Quant+Store KV**：5 个 Element-wise 算子重构为单一微型流水线 Kernel，寄存器级数据流转，在线量化直接低比特写入 KV Cache，加速约 5x。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

**Fused AllReduce+Norm+Add**：通信、残差计算、归一化全链路融合，高吞吐版（NVSwitch 多播）+ 低延迟版（Lamport P2P），覆盖 8~32k tokens，最高加速 1.68x。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

**采样融合算子**：10 余个零碎 Kernel 融合为 2 个核心 CUDA Kernel，全词表单次加载，GPU 闭环惩罚计算，相比 vLLM/FlashInfer 提升约 5.5x/2.5x。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

**GEMM+Comm 通算融合**：SM 显式划分为计算 SM（矩阵乘）与通信 SM（RS 搬运），Load→MMA→Epilogue 三级流水，Tile 级计算与通信重叠，加速比 1.68x~1.81x。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

## 并行策略

**Prefill TPSP**：SP 拆分 + 通算融合 + 通信量化 + 并行模式调整。Prefill 16k TTFT 降 29.9%（764→536ms），32k 降 24.5%（1885→1424ms）。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

**Decode DP+EP**：Attention DP + MoE EP 跨节点混合并行，自研 HPC Kernel，Async EPLB 权重重排与 Decode 完全重叠，端到端吞吐提升 15.7~44.7%。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

## 多级缓存

GPU→CPU→KVStore 三级缓存体系，请求按 L1→L2→L3 顺序查询可复用前缀，新 Block 异步下沉至 L2/L3。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

## MTP 异步调度

解除 CPU 对真实接收长度的同步依赖，按最大接收长度提前准备，减少 decode 间 5~10ms CPU 气泡，端到端提升 10%~20%。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

## 量化压缩

**AngelSlim 量化**：GPTQ 权重重建 + 激活平滑与旋转变换 + QAT 轻量化微调。Attn FP8 + W4A8 配置下精度无损（与 BF16 基线差距 < 1%），端到端吞吐提升 28%+。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

**Stem 稀疏注意力**：Token Position-Decay（头部 k_start 线性衰减到尾部 k_end = μ·k_start）+ Output-Aware Metric（OAM: QK^T + β·max(0, log(||V_j||₂))）。25% 计算预算实现接近稠密注意力精度，128K 上下文 Prefill 延迟降低 3.6x。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

## 与现有知识库的关联

- [[entities/腾讯混元新里程碑hy3-preview-发布开源agent-表现全面提升|腾讯混元 Hy3 preview 发布]]：互补实体，该篇讲模型能力与发布，本篇讲推理优化技术细节
- [[entities/llm-inference-pipeline-internals|LLM 推理流水线]]：推理优化基础知识，本篇是 Hy3 的具体工程实践
- [[concepts/model-distillation-compression|模型蒸馏与压缩]]：量化压缩（W4A8、AngelSlim）是模型压缩的推理侧实践
- [[concepts/transformer-architecture|Transformer 架构]]：GQA + MoE 架构是 Hy3 的基础

→ [[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization|原文存档]]
