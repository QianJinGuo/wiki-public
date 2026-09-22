---
title: "腾讯混元 Hy3 preview 在 Hopper 卡上的推理优化实践"
created: 2026-06-30
updated: 2026-09-23
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

## 深度分析

### MoE + 稀疏注意力为什么改变了 Hopper 优化问题

Hy3 preview 的架构画像决定了优化重心：295B 总参但每 token 只激活 21B（GQA + MoE），意味着单卡显存装不下全部专家，而计算量相对显存占用偏小——瓶颈从 FLOPs 转移到访存、跨卡通信和调度气泡上。这解释了为什么优化清单里通信类（AllReduce 融合、GEMM+Comm 通算融合）和调度类（动态 Attention 装桶、Async EPLB）占了近半篇幅，而非传统意义上的"把矩阵乘写快"。256K 上下文又叠加了 KV Cache 膨胀问题，于是 KV 写入路径上的 5 算子融合（Rope+Norm+Hadamard+Quant+Store）把在线量化直接挪进 Kernel、以低比特落盘，相当于在数据产生的现场就完成压缩，避免了"先写高比特、再读出来降比特"的往返。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

更值得注意的是 Attention 动态调度的思路：真实生产流量里序列长度高度不均（测试集输入从数千到 192k），按 Tile 粒度拆分后贪心装桶，把"负载不均"从系统问题降维成单次推理前的一次映射表生成。这是典型的"承认数据分布、改造调度粒度"路线，比假设均匀 batch 的静态 kernel 更贴合生产。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

### 三级缓存与 MTP 异步调度：各自买到什么

三级缓存（GPU→CPU→KVStore）买的是长上下文复用率：测试数据里缓存理论命中 80%，意味着五分之四的 Prefill 计算理论上可跳过。设计要点不在"存"，而在新 Block 异步下沉——下沉不阻塞当前请求，才不会把省下的计算又还回去。MTP 异步调度买的是另一类东西：多 token 预测引入"接收长度不可预知"的不确定性，做法是解除 CPU 对真实接收长度的同步依赖，按最大长度提前准备、用上轮结果修正，消掉 decode 间 5~10ms 的 CPU 气泡。两者本质相同——把"必须等一个不确定结果"改成"按上界乐观准备 + 事后修正"，这是推理调度里反复出现的模式。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

### 量化与稀疏：精度换算力的两条不同兑换率

AngelSlim（GPTQ 重建 + 激活平滑旋转 + QAT 轻量微调）在 Attn FP8 + W4A8 下与 BF16 基线差距 <1%，吞吐 +28%——这是一条"几乎免费"的兑换率，说明对激活分布做过平滑/旋转变换后，低比特的精度损失基本可以被训练侧补偿。稀疏注意力则是另一条：Stem 用 Token Position-Decay（k_end = μ·k_start）和 Output-Aware Metric（QK^T 加上 V 范数的对数修正项）来决定哪些 token 值得算，25% 计算预算逼近稠密精度、128K Prefill 降 3.6x。兑换率更激进，但依赖一个架构假设——注意力重要性沿位置衰减——对长文档场景成立，对需要全局精确检索的任务未必。两条路线的取舍点不同：量化动数值表示，稀疏动计算集合，工程上先做前者风险更低。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

### 映射到更广的推理成本工程趋势

这份实践与 [[entities/mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo 2.5 推理系统优化]]、[[entities/deepseek-moe-parallel-strategy|DeepSeek MoE 并行策略]] 呈现同一个收敛方向：当模型规模进入百 B 级 MoE，推理优化从"算子军备竞赛"转入"系统协同设计"——通信与计算在 SM 级重叠、Prefill/Decode 采用不同并行策略（TPSP vs DP+EP）、CPU 调度与 GPU 执行全异步。单点 kernel 加速数字（2.86x~5.5x）看着大，但端到端收益被 Amdahl 定律稀释，真正决定成本的是把 Prefill TTFT（4s 约束）和 decode 吞吐（50ms TPOP 约束）当一等公民、按服务等级反推优化优先级。这套"约束驱动的全栈优化"方法论比任何单一加速比数字更可迁移。相关概念背景见 [[concepts/inference-optimization]] 与 [[concepts/moe-mixture-of-experts-2025]]。^[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization.md]

## 实践启示

1. 优化前先把流量画像测清楚（长度分布、缓存命中率、输入输出比）：本案例中平均输入 68k/输出 0.9k 的极端比例，直接决定了优化重心在 Prefill 与 KV 复用而非 decode 采样。
2. 量化先行：带激活变换的 W4A8/FP8 通常是风险收益比最高的第一刀，<1% 精度差距换 28%+ 吞吐；稀疏注意力等结构性改动放在量化验证之后再上。
3. 长上下文服务把 KV Cache 当分层存储设计：GPU→CPU→KVStore 三级查询 + 异步下沉，80% 理论命中率意味着缓存策略的 ROI 可能高于继续打磨 kernel。
4. 调度不确定性用"乐观上界 + 事后修正"处理：MTP 接收长度、batch 组合、专家负载均可套用此模式，关键是把同步等待从关键路径上摘掉。
5. 通信优化优先做融合而非换硬件：AllReduce+Norm+Add 融合与 GEMM+Comm SM 级重叠在现有 NVSwitch 拓扑上拿到 1.68x~1.81x，无需新硬件。
6. 端到端指标（TTFT/TPOP）设服务约束、按约束反推优化优先级，避免陷入单点 kernel 加速比数字的局部最优。

## 与现有知识库的关联

- [[entities/腾讯混元新里程碑hy3-preview-发布开源agent-表现全面提升|腾讯混元 Hy3 preview 发布]]：互补实体，该篇讲模型能力与发布，本篇讲推理优化技术细节
- [[entities/llm-inference-pipeline-internals|LLM 推理流水线]]：推理优化基础知识，本篇是 Hy3 的具体工程实践
- [[concepts/model-distillation-compression|模型蒸馏与压缩]]：量化压缩（W4A8、AngelSlim）是模型压缩的推理侧实践
- [[concepts/transformer-architecture|Transformer 架构]]：GQA + MoE 架构是 Hy3 的基础

→ [[raw/articles/tencent-hunyuan-hy3-preview-hopper-inference-optimization|原文存档]]
