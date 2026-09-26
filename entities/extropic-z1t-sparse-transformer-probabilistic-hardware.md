---
title: "Extropic Z1T：面向概率硬件的稀疏 Transformer 类模型"
created: 2026-09-08
updated: 2026-09-27
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

## 深度分析

### 概率硬件的必然稀疏性

Z1 与 GPU 的根本分歧不在算力，而在**互联拓扑**：GPU 靠 cache 层级换来了核心间全互联，因此能高效执行稠密 matmul；而 Z1 的父图直接烧死在硅里，每个 pbit 固定 16 个邻居，相互作用图的度是常数——这意味着任何部署到该硬件上的模型**在结构上就必须是稀疏的**，稀疏不是优化选项而是物理约束。同样值得注意的是反向的不对称性：在 GPU 上跑稀疏计算（如 99% 稀疏矩阵乘）并不能获得成比例的加速，因为不规则访存吃掉了理论收益。所以 Z1T 的设计方向不是"把现有模型搬上概率芯片"，而是反过来，为固定稀疏拓扑重新发明 transformer 原语——这正是作者所称"首次算法突变"的实质：算法演化方向由硬件 substrate 决定，而非延续 GPU 时代的稠密优化路径。

### dy4p/p-Int 编码与 tanh-linear：概率计数的精度红利

这一层设计里藏着一个反直觉的要点：物理上每个 pbit 只有 4-bit（dy4p，scale 为 2 的幂），但 p-Int 把量化数表示为**分布**，实数值取 pbit 流的经验均值——多次采样做统计平均后，有效位精度可超过物理编码位数（经验上界约为 k ≤ ½log₂N − log₂σ，N 为样本数）。换句话说，**精度从静态的位宽变成了动态的采样预算**：运行期可以按需增减样本数来买卖精度。tanh-linear 单元则把这一机制利用到极致——条件于隐藏自旋后，可见自旋是 Bernoulli 分布，其期望 E[v|h] = tanh(b_v + ΣJⱼhⱼ) 恰好就是"稀疏矩阵-向量积融合 tanh"这一深度学习最需要的原语，且是硬件采样过程的自然副产品，无需额外计算。权重矩阵直接编码为 spin 向量间的相互作用，等于让"存权重"和"算激活"在物理上合一，这正是其内存内计算能效来源的微观解释。

### 固定连通度作为缩放定律的新维度

这组稀疏缩放实验在方法论上有个干净的处理：唯一的新旋钮是**连通度 c**（每输出节点连接 c 个输入），且从训练起点就固定施加，而非训练后剪枝。由于 c 是绝对值而宽度会增长，百分比稀疏度随规模渐近趋于 100%（实验区间 5%–99.8%），这与产业界常见的"按百分比剪枝"是两种不同的稀疏语义。iso-FLOP 分析给出的核心权衡是：同参数量下稠密 FLOP 每单位信息量更高效，稀疏模型需约一个数量级更多 FLOP 才追平 GPT-2 的 loss——但这笔"FLOP 债"能否还得起，取决于 FLOP 在不同 substrate 上的单价。Z1 上稀疏乘加的能效约为 GPU 的三个数量级，一加一减之后净赚约两个数量级。这提示了一个评估框架：**跨 substrate 比较模型时应比较"达到目标 loss 的焦耳数"而非 FLOPs**，FLOPs 只在单一 substrate 内部才是有效货币。

### 性能估算的结构与剩余瓶颈

拆开 294.52 nJ/token 的估算，Z1 采样只占 8.74 nJ，而 FPGA 协处理占 285.78 nJ（>95%）——**当前能效瓶颈根本不在概率芯片本身**，而在为通用性保留的数字协处理器上。这解释了估算表中三档数字的巨大跨度：全系统对比 H100 是 14×–139×（取决于 MFU），而仅看 Z1-only 层是 468×–4,680×，两者之差就是 FPGA 的稀释效应。延迟侧同理：Z1T conservative 的 58.8 µs（≈17,000 tok/s）已能超过 H100 eager（702 µs）和 torch.compile（102 µs），而这还是非专用 fabric 上的保守值。作者由此推出的上限判断是自洽的：若专为 Z1T 类模型设计芯片、把 FPGA 承担的操作移入 sub-threshold CMOS，即可逼近 Z1-only 层的 ~1000× 增益。另外系统边界上"decode 用 Z1+XPU、prefill 留给 GPU"的分工，本质上是按算子对稀疏/稠密的亲和度做异构切分。

## 实践启示

1. **评估能效主张时先拆瓶颈结构**：看到"概率硬件 139× 能效"这类数字，应立即追问各组件的能耗占比——此处 FPGA 占 >95%，说明 139× 远非该技术的上限，真正值得跟踪的是专用芯片（去 FPGA 化）后的后续发布。
2. **跨硬件比较模型用"每 token 焦耳 × 达标 FLOPs"，别只看 FLOPs**：稀疏模型在 FLOP 维度吃亏（约 10×），在 substrate 单价维度赚回来（约 1000×）。若你的部署场景能耗/散热是硬约束，稀疏+专用硬件路线值得纳入技术雷达。
3. **为固定拓扑硬件做算法设计时，从"结构约束"而非"近似技巧"出发**：Z1T 的稀疏性来自硅中固定的图连通度（c 为绝对值、训练全程固定），这与训练后剪枝是不同语义——设计新型硬件适配模型时，应把硬件拓扑作为一等约束写进架构与训练配方，而非事后压缩。
4. **低精度表征可借统计采样"续位"**：p-Int 表明物理 4-bit 编码经多次采样平均可获得超越位宽的有效精度，且精度可在运行期调节。这一思路对边缘设备上"位宽不够精度来凑"的量化部署有借鉴意义（代价是延迟换精度）。
5. **异构分工按算子亲和度切分，而非整机替换**：decode 交给 Z1+XPU、prefill 留给 GPU 的方案说明，新硬件短期内最现实的落地方式是嵌入现有推理管线中取代最能耗的环节，而非端到端替代——评估新加速器时可优先寻找这类切入点。

## 相关实体

- [[concepts/transformer-architecture|Transformer 架构]] —— Z1T 对经典 transformer 原语进行稀疏化改造
- [[concepts/scaling-laws|缩放定律]] —— 稀疏/低连通度硬件的经验缩放
- [[concepts/attention-mechanism|注意力机制]] —— Gated Convolutional Attention 取代 dense softmax attention
- [[concepts/inference-optimization|推理优化]] —— 异构 decode 分解与能量效率
- [[entities/ai-infra-llm-efficient-inference-vllm|LLM 高效推理]] — 能效视角对比 GPU 主流方案

→ [[raw/articles/extropic-z1t-sparse-transformer-probabilistic-hardware|原文存档]]