---
title: Scaling Laws
created: 2026-04-28
updated: 2026-08-29
type: concept
tags: [scaling, training, compute, llm, inference]
related:
  - [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes]]
  - [[entities/foundation-model-building-blocks|Foundation Model Building Blocks]]
  - [[entities/aws-generative-ai-model-agility-framework|AWS Model Agility Framework]]
sources:
  - raw/articles/scaling-law-triple-decomposition-sujianlin-paperweekly-2026-08-24
confidence: high
---
# Scaling Laws（扩展定律）
## Overview
扩展定律描述神经网络的性能如何随模型参数量、数据量和计算量的增加而提升，是大模型训练的核心指导原则。2020年代中后期，scaling 研究从单一的预训练 loss 曲线演化为**三重扩展 regimes**：预训练、后训练（RLHF/SFT）、测试时计算（long-thinking、search）。这三个 regimes 共同强化而非分化基础设施需求——都要求紧耦合加速计算、高带宽低延迟网络和可扩展分布式存储。^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]

## Classical Scaling Laws
2020 年 Kaplan 等人（OpenAI）提出的 Scaling Laws：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]
- **L(N)** ∝ N^(-0.076)：模型越大，loss 下降越快
- **L(D)** ∝ D^(-0.095)：数据越多，效果越好
- **L(C)** ∝ C^(-0.050)：计算量增加，loss 按幂律下降

这些定律奠定了"bigger is better"的行业共识，推动了 GPT-3 (175B)、PaLM (540B) 等超大模型的涌现。

## Compute-Optimal Training
### Chinchilla（Hoffmann et al., DeepMind, 2022）
Chinchilla 提出：模型规模和数据量应等比例扩展。^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]
- 参数量 N 和训练 token 数 D 应满足 N ∝ D^0.54
- 相同算力下，Chinchilla 推荐的模型比 GPT-3 小但数据量更大
- 这意味着 GPT-3 规模的模型应该用约 4.6x 更多数据训练

### Post-Chinchilla 的争论与修正
Chinchilla 定律并非终极答案。后续研究和实践揭示了多个关键争议点：

**Data Quality vs. Quantity**：多项数据工程研究表明，数据质量（去重、过滤、多样性）对最终模型性能的影响可能超过简单数量扩展（注：The Bitter Lesson 讨论的是通用方法+算力优于人类先验，与数据质量无直接关系）。DeepSeek-V4 等工作通过大规模数据清洗和合成数据补充，在同等算力下达到更好效果。

**Repetition and Data Curation**：GPT-4o 和 Llama 3 的训练实践表明，.repeat() 过采样数据会导致模型记忆而非泛化。业界逐渐形成"在更高质量数据上用更少 tokens 训练"的共识。^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]

**Fixed Compute Budget 的局限性**：当研究者有固定 GPU-hours 而非固定预训练目标时，Chinchilla 最优解可能不是实际最优。Llama 3 的 15T-token 训练实践表明，在高质量 web data 上训练超过 1T tokens 后仍能持续提升，暗示 Kaplan 曲线在数据质量足够高时可能比 Chinchilla 预测的更陡峭。

### Llama 3 15T-token 训练：大规模训练的里程碑
Meta Llama 3（15T = 15 万亿训练 token，约 405B 参数的 405B 版本为最大型号）在 2025-2026 年展示了超大规模训练的工程极限：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]
- 使用超过 100,000 个 H100 GPU，训练时间以月计
- 训练涉及复杂的 pipeline parallelism、tensor parallelism 和 data parallelism 的 3D 并行配置
- Checkpoint 管理、故障恢复和弹性训练成为核心工程挑战
- 训练后的 RLHF/SFT 后训练阶段（post-training）占用了与预训练相当或更多的算力

Llama 3 的实践揭示了一个关键 insight：**后训练 scaling 与预训练 scaling 同样重要**，甚至在某些方面更难优化。

## Emergent Capabilities（涌现能力）
Scaling 的一个核心神秘现象是**涌现能力**：模型在突破特定参数量或训练计算量阈值后，突然具备此前不存在的能力。

### 典型涌现能力
- **多步推理**：在足够大规模下出现 chain-of-thought 能力
- **代码生成**：GPT-3 5B 规模以下是随机字符，GPT-3 175B 出现功能性代码
- **指令遵循**：O1/O3 系列模型在 post-training 后涌现出复杂推理
- **多语言能力**：往往在 10B+ 参数量后显著提升

### 涌现的争议
部分研究者（ Schaeffer et al., 2023 ）认为涌现可能是评估指标的 artifact 而非真实现象：使用连续指标时，能力往往呈现平滑提升而非突变。但在实践中，能力曲线确实存在明显拐点，这使得 scaling 成为可预测的工程问题而非纯粹的玄学。

## Inference vs. Training Scaling（推理与训练扩展的分化）
传统 scaling laws 聚焦于训练阶段，但 2024-2026 年的研究揭示了**推理扩展与训练扩展有本质区别**：

### Test-Time Compute Scaling（测试时计算扩展）
O1、O3、DeepSeek-R1 等 reasoning 模型开创了**测试时计算扩展**的新范式：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]

| 维度 | 训练时扩展 | 测试时扩展 |
|------|-----------|-----------|
| 扩展资源 | 训练 tokens、GPU-hours | Inference tokens、思考时间 |
| 扩展效率 | 随参数量线性或超线性 | 随计算量亚线性 |
| 瓶颈 | 通信（all-to-all in MoE） | 内存（KV cache） |
| 最优策略 | 3D 并行、流水线 | Beam search、Tree search、Self-evolution |

测试时计算允许模型在推理时"思考更久"，这对基础设施的影响是：推理基础设施需要更高的内存带宽和更大的 KV cache 容量，而非单纯的峰值 FLOPS。^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]

### 推理架构的分化趋势
vLLM 的 PagedAttention 和 SGLang 的 RadixAttention 代表了两种 KV cache 管理策略：
- **PagedAttention**：通过分页虚拟内存减少碎片化，支持更大 batch size
- **RadixAttention**：实现跨请求的自动前缀复用和 cache-aware 负载均衡

两者都支持 tensor parallelism 以处理单 GPU 显存不足的模型。^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]

### Disaggregated Serving（分解式服务）
NVIDIA Dynamo 的 disaggregated serving 将 prefill 和 decode 阶段分离到不同 GPU pool：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]
- Prefill 阶段：计算密集，需要高吞吐
- Decode 阶段：内存密集，需要大容量 HBM
- NIXL（Inference Xfer Library）提供统一的 point-to-point 传输 API，跨越 HBM/DRAM/NVMe/分布式存储多种内存层

这种分解式架构对 ML infrastructure 的设计提出了新的要求。

## Efficient Scaling（高效扩展技术）
### AttnRes（Kimi）
注意力残差（AttnRes）通过新的训练效率维度，在同等性能下节省约 1.25x 算力。^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]

### Sparse MoE（稀疏专家混合）
Mixtral、DeepSeek-V4 等 MoE 模型通过稀疏激活减少实际算力需求：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]
- 每个 token 只激活部分 experts（如 8 个中的 2 个）
- 实际推理算力接近同等参数量的 dense 模型
- 通信瓶颈：expert parallelism 依赖 all-to-all 通信，通信量随专家并行度线性增长

DeepSeek MoE Parallel Strategy 提供了针对 MoE 架构的并行策略优化。^[raw/articles/deepseek-moe-parallel-strategy.md]

### Quantization（量化）
INT8/INT4 量化减少推理算力需求：
- INT8：在大部分推理场景下无损或近乎无损
- INT4：需要精细 calibration，在特定任务上可能有显著损失
- FP8：NVIDIA H100/B100 原生支持，提供精度与效率的折中

### Checkpoint Optimization
大规模训练的故障恢复成本极高。NVIDIA nvcomp 和 HyperPod checkpointless training 通过 P2P 状态复制减少 checkpoint 开销：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]

## Infrastructure Implications（基础设施含义）
Scaling laws 的物理实现需要硬件和软件的协同设计：

### GPU 演进路径
从 H100 (80GB HBM3, 3.35TB/s) 到 B300 (288GB HBM3e, 8TB/s)：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]
- 显存容量提升 3.6×
- NVLink 聚合带宽从 7.2TB/s 翻倍到 14.4TB/s
- EFA 从 v2 升级到 v4，端到端网络延迟持续优化（v2→v3 降低 35%，v3→v4 再降低 18%）

### 通信瓶颈的关键性
随着模型规模扩大，step time 越来越被集体通信和内存移动主导，而非原始计算吞吐：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]
- NVLink domain 的大小成为首要约束因素
- UltraServers 通过 GB200 NVL72 平台将 NVLink domain 扩展到 72 GPU (13.4TB aggregate HBM3e)
- 这使通信密集型工作负载能在 NVLink fabric 内部完成，减少对 EFA 跨节点网络的依赖

### 资源编排
Slurm 和 Kubernetes 代表两种路线：^[raw/articles/building-blocks-for-foundation-model-training-and-inference-on-aws.md]
- **Slurm**：以作业级原子调度适合 HPC 风格的大规模训练，512 GPU 训练任务需同时分配 64 个 8-GPU 节点
- **Kubernetes**：以声明式 API 适合云原生部署，但需要 Kueue/Volcano/KAI Scheduler 补齐作业级调度和拓扑感知

## 框架选择
- **HuggingFace Transformers + Accelerate**：适合微调和中等规模训练，注重易用性和模型兼容性
- **NVIDIA Megatron Core**：适合对规模效率有极致追求的场景，提供 3D 并行（tensor/pipeline/expert）和 FP8 混合精度优化
- **veRL**：适合 RLHF 类后训练工作负载，HybridFlow 架构允许训练和推理引擎共享权重避免同步开销

## Related
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes]] — 新的训练效率维度
- [[entities/foundation-model-building-blocks|Foundation Model Building Blocks]] — AWS 基础设施视角的三重扩展
- [[entities/aws-generative-ai-model-agility-framework|AWS Model Agility Framework]] — 跨代际模型迁移框架
- [[entities/aws-ec2-capacity-blocks-gpu-ml|EC2 Capacity Blocks for GPU]] — 短期 GPU 容量决策
- [[entities/deepseek-moe-parallel-strategy|DeepSeek MoE Parallel Strategy]] — MoE 架构并行策略
- [[entities/nvidia-cut-checkpoint-costs-nvcomp|NVIDIA nvcomp]] — Checkpoint 压缩优化
- [[concepts/transformer-architecture]] — Transformer 架构基础

## 新增关联实体
- [[entities/cola-dlm-byte-dance-continuous-latent-diffusion-language-model]]
- [[entities/digitalocean-serverless-inference-55-models]]
- [[entities/fusedash-generative-analytics-platform]]
- [[entities/karpathy-boris-software3-llm-era-programming-2026]]
- [[entities/laser-acl2026-latent-superposition-visual-reasoning]]

## 关联实体

**上游依赖**:
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres]] — 提供基础理论/方法
- [[entities/foundation-model-building-blocks]] — 提供基础理论/方法
- [[entities/aws-generative-ai-model-agility-framework]] — 提供基础理论/方法

**下游应用**:
- [[entities/foundation-model-building-blocks]] — 具体应用场景
- [[entities/aws-generative-ai-model-agility-framework]] — 具体应用场景
- [[entities/aws-ec2-capacity-blocks-gpu-ml]] — 具体应用场景

**平行协作**:
- [[entities/cola-dlm-byte-dance-continuous-latent-diffusion-language-model]] — 替代/补充方案
- [[entities/digitalocean-serverless-inference-55-models]] — 替代/补充方案
- [[entities/fusedash-generative-analytics-platform]] — 替代/补充方案

## 三重分解统一框架（2026-08-24 SUPP：解构 Scaling Law，v=8 c=7 v×c=56）

> 来源：苏剑林（科学空间，PaperWeekly）原创数学解构。把模型在理想分布上的损失与训练现状的差距，分解成三层递进距离：数据误差（泛化）→ 优化误差（优化器天花板）→ 架构误差（架构天花板），并基于「异幂不等式」统一推导、对照 Kaplan/Chinchilla/Step/Microsoft/DeepSeek 各 Law 的指数。^[raw/articles/scaling-law-triple-decomposition-sujianlin-paperweekly-2026-08-24.md]

- **三重分解**：总差距 = 泛化误差（训练集 vs 理想分布，关键=数据数量/质量/多样性）+ 优化误差（实践优化器 vs 完美优化器天花板，衡量学习率/步数/批大小）+ 架构误差（实践模型 vs 数据自身理论极限，衡量参数量/深度/宽度）。尽可能解耦各变量对损失的影响。^[raw/articles/scaling-law-triple-decomposition-sujianlin-paperweekly-2026-08-24.md]
- **核心工具「异幂不等式」**：固定两项幂律乘积下求幂律之和最小值，最小值点与最小值本身依然是幂律——后续推导基石。在「最优学习率」「最优批大小」假设下不断简化 Scaling Law 形式。^[raw/articles/scaling-law-triple-decomposition-sujianlin-paperweekly-2026-08-24.md]
- **统一对照各 Law 指数**：最优批大小正比于样本数 ≤1 幂（Step Law 吻合），最优学习率反比（Microsoft Law 吻合、Step Law 相反）；代入理论值后可同时逼近 Step Law/Microsoft Law/Chinchilla Law；固定算力预算下求最优参数量与数据量接近 Chinchilla（最优规模与数据量等比例扩展）。各 Law 在最优批大小上较一致、最优学习率分歧大。^[raw/articles/scaling-law-triple-decomposition-sujianlin-paperweekly-2026-08-24.md]
- **MoE/记忆之层扩展**：MoE 理论计算量只取决于激活参数量，增加稀疏度理论上不增加计算量；等效参数量概念（激活参数 a、总参数 n 的 MoE 等效某 Dense 模型）；MoE 与 Memory 层（PKM/UltraMem/Engram 等）同时用则 Scaling Law 可能加性，存在最优配比而非全给其一。^[raw/articles/scaling-law-triple-decomposition-sujianlin-paperweekly-2026-08-24.md]
- **数据篇**：区分训练量 D 与 Multi-Epoch，推导最优 Epoch 数——数据越少反而应训更多轮（与 2511.13421 相反）；领域配比/质量/模态等维度难统一量化。^[raw/articles/scaling-law-triple-decomposition-sujianlin-paperweekly-2026-08-24.md]
- **幂律之问与系数之问**：为什么幂律——指数函数短尾衰减太快与「持续投入持续改善」体感不符，幂函数长尾、等价无标度性、log-log 直线易拟合、异幂不等式封闭。系数之问——指数普适（=问题难度/数据分布任务决定，类比统计物理相变临界指数），系数随工程改进变化（=解决方案工程水平）；若有限工程改进能改指数则相对优势随规模无界放大不现实。^[raw/articles/scaling-law-triple-decomposition-sujianlin-paperweekly-2026-08-24.md]

## 所属 MOC

- [[moc/layer-1-llm-principles|Layer 1 Llm Principles]]
- [[moc/wiki-master-map|Wiki Master Map]]
