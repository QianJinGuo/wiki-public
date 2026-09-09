---
title: "Extropic Z1T：面向概率硬件的稀疏 Transformer 类模型"
created: 2026-09-08
updated: 2026-09-08
type: entity
tags: [transformer, hardware, sparsity, inference-optimization, scaling-law, probabilistic-computing, model-architecture, hardware-software-co-design]
sources: [raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware]
confidence: 0.68
provenance_state: extracted
---

# Extropic Z1T：面向概率硬件的稀疏 Transformer 类模型

Extropic（作者 Guillaume Verdon、Alexander Neagoe、Owen Lockwood、Seth Morton）发布 Z1T —— 一款为概率 sub-threshold CMOS 芯片 Z1 协同设计、稀疏连接的 transformer 类模型架构。作为"从面向 GPU 稠密矩阵乘优化转向面向下一代稀疏内存内计算硬件"的首次算法突变，Z1T 将 transforme 原语（RMSNorm、softmax 注意力、FeedForward）适配为稀疏 tanh-linear 操作，并开放稀疏 transformer 训练配方与 Z1T 权重。^[raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware.md]

## 概率硬件基底：Z1

Z1 是概率比特（pbit）的图模型：二进制随机 CMOS 电路，样本从可编程 Ising 模型以 50 MHz 内部时钟经 Gibbs 采样获得。每个 pbit 的耦合度为 16，单芯片 269,568 pbit、2,135,904 条耦合边；父图固定于硅中，任何嵌入硬件的操作必然稀疏。GPU 通过 cache 内存层级实现核心全互联，可执行稠密 matmul；GPU 上的稀疏操作产生不规则访存且不带来与稀疏度成比例的速度提升（99% 稀疏矩阵相乘并非 100 倍快）。^[raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware.md]

## 硬件编码：dy4p / p-Int 与 tanh-linear

连续激活/权重通过 dy4p（dyadic 4-bit 精度）量化到 pbit 表示；p-Int 是量化数上的分布，实数值取 pbit 流的经验均值。关键洞察：概率计算机可获得高于物理 4-bit 编码的有效位精度（含有效分数位），并可在运行期通过缩放样本数调节精度 —— 天然契合现代深度学习低精度表征。

tanh-linear 单元从单个概率 cell 平均构造，条件于隐藏节点后可见自旋为 {−1,+1} Bernoulli，平均后是局部场 tanh：E[v|h] = tanh(b_v + ΣⱼJⱼhⱼ)。权重矩阵直接编码到展平 spin 向量相互作用；硬件天然采样得到稀疏矩阵-向量积融合 tanh 的分布期望。^[raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware.md]

## Z1T 架构改造

- **RMSNorm → Dynamic Tanh（DyT）**：RMSNorm 分量近似 tanh 激活，可被缩放 DyT 替换，天然在 Z1 上实现。
- **Softmax Attention → Gated Convolutional Attention（GCA）**：用 4-sparse 投影矩阵计算 Q、K、V gates；Y = tanh(Q) ⊙ N/D，N、D 由 conv1d 与累积求和给出。部分算术/超越函数卸载到 FPGA/XPU。
- **FeedForward**：MLP 由 tanh sampling 程序在 fabric 上组合，将 d_out 个 tanh-linear 单元布局于 Z1 并跨核流式传输样本。

异构 decode 分解：Z1 + FPGA 模型并行跨 Z1、流水并行跨 FPGA/Z1；FPGA 做残差、注意力/MLP 变换与词表读出，Z1 采样下一 token。稠密词表 matmul 也可跑在 GPU/Cerebras 等。^[raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware.md]

## 稀疏缩放定律

在标准 GPT-2 风格解码器上引入唯一旋钮"连通度 c"（固定连通度非百分比稀疏；宽度增长时百分比稀疏渐近 100%，实验中 5% 到 99.8%）。基准 c∈{4,16,32,64,128}、序列长 256、3×10¹⁴ 到 10¹⁸ FLOPs，训练于 byte-tokenized OpenWebText（iso-FLOP 分析）。结论：相同参数下稠密 FLOP 比稀疏 FLOP 更高效；但 Z1 上的稀疏乘加远比 GPU 节能（内存内特性），故稀疏模型可在功率一小部分达相同性能。Z1T（4-bit 权重、每节点 4 入边，OpenWebText + GPT-2 BPE）：需约一个数量级更多 FLOP 才能匹配 GPT-2 loss，但 Z1 操作相对 GPU 约三个数量级能效增益，净得约两个数量级能效增益。^[raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware.md]

## 性能估算

- 能效（每 token，排除最终稠密 logit）：Z1T = 294.52 nJ（8.74 nJ Z1 采样 + 285.78 nJ FPGA）；对比 H100 @10% MFU 40.9 µJ ≈ 139×，@100% MFU 4.09 µJ ≈ 14×；Z1-only layers ≈ 468×–4,680×。
- 延迟（每 token）：Z1T conservative 58.8 µs（≈17,000 tok/s）；H100 eager 702 µs；H100 torch.compile 102 µs。建议 Z1+XPU 做 decode 而非 prefill（prefill 现 GPU 更优）。
- FPGA 消耗 >95% 能量；去掉 FPGA 瓶颈、将更多操作移至 sub-threshold CMOS 可望达 GPU 1000× 能效。^[raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware.md]

## 相关实体

- [[concepts/transformer-architecture|Transformer 架构]] —— Z1T 对经典 transformer 原语进行稀疏化改造
- [[concepts/scaling-laws|缩放定律]] —— 稀疏/低连通度硬件的经验缩放
- [[concepts/attention-mechanism|注意力机制]] —— Gated Convolutional Attention 取代 dense softmax attention
- [[concepts/inference-optimization|推理优化]] —— 异构 decode 分解与能量效率
- [[entities/ai-infra-llm-efficient-inference-vllm|LLM 高效推理]] — 能效视角对比 GPU 主流方案

→ [[raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware|原文存档]]