---
title: "腾讯混元AI Infra如何优化Hy3 Preview 一次大模型推理性能提升的 腾讯技术工程"
created: 2026-08-01
updated: 2026-11-24
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 腾讯混元AI Infra如何优化Hy3 Preview 一次大模型推理性能提升的 腾讯技术工程

> -> [[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md|原文存档]]

## 摘要

腾讯混元 AI Infra 推理团队剖析了 Hy3 preview 模型（GQA+MoE 混合架构、原生 256K 上下文）在算力更低、显存更紧凑的 NVIDIA Hopper 卡上的推理全栈优化实践，从算子优化与融合、并行策略、多级缓存、MTP 和异步调度、量化与稀疏五大维度展开。算子层面：动态调度的 Attention 负载均衡在单 batch 长文本场景单算子最高加速 2.95x；Router GEMM 用双 BF16 重构 FP32 计算，相比 cuBLAS FP32 实现加速 2.86x~3.22x；FusedMoE 流水线重构相比 vLLM/SGLang 加速 1.5x-1.6x；采样算子将十余个 Kernel 融合为 2 个，全词表只加载 1 次，相比 vLLM 提升约 5.5x。^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md]

并行与系统层面：Prefill 采用 TPSP 并行策略使 32k 上下文 TTFT 降低 24.5%；Decode 采用 Attention DP + MoE EP 跨节点混合并行，端到端吞吐提升 15.7%~44.7%；构建 GPU→CPU→KVStore 三级缓存体系扩大有效缓存容量；MTP 场景下解除 CPU 对真实接收长度的同步依赖，消除 decode 间 5~10ms CPU 气泡、端到端提升 10%~20%。量化与稀疏层面：AngelSlim 框架通过"GPTQ 权重重建 + 激活平滑与旋转变换 + QAT 轻量微调"实现 W4A8 + Attn FP8 精度无损（与 BF16 差距 <1%）、吞吐提升 28%+；自研 Stem 稀疏注意力算法配合 HPC-BSA 算子，仅用 25% 计算预算达到接近稠密注意力精度，128K 上下文 Prefill 延迟降低 3.6 倍。测试基于 5000 条真实数据（最大输入 192k、平均 68k），约束 50ms TPOT、4s TTFT。^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md]

## 关键要点

- Fused Rope+Norm+[Hadamard]+Quant+Store KV 将 5 个低算力强度算子融合为单 Kernel，融合算子加速约 5x。
- Fused AllReduce+Norm+Add 与腾讯网络平台部联合实现，基于 NVSwitch 多播与 Lamport P2P 两种版本，相比 NCCL/FlashInfer 最高加速 1.68x。
- GEMM 与 ReduceScatter 细粒度通算融合将 SM 显式划分为计算/通信两类角色，端到端加速比 1.68x~1.81x。
- Stem 稀疏注意力的两个关键技术：Token Position-Decay（Top-k 预算从头部向尾部线性衰减）和 Output-Aware Metric（把 Value 模长引入 token 选择标准）。
- 后续方向：C4 与 W4 量化优化、并行投机解码、PD 高效传输、多级缓存中心及其他硬件平台适配。

## 深度分析

### 算子优化与融合：从单 Kernel 调优走向全链路重构

Hy3 的算子优化有一条清晰主线：凡是被 HBM 访存或 Kernel 启动开销主导的环节，一律用融合与执行重排压缩到极限。Attention 侧针对线上长短请求混杂造成的长尾延迟，放弃静态 split-kv 在长序列吞吐与短序列开销之间的固定权衡，改为统一 Tile 粒度拆分加贪心装桶的全局负载均衡，由 Task Assign 映射表驱动各层 Kernel 领取任务，单 batch 长文本单算子最高加速 2.95x^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:45-65]。Router GEMM 的双 BF16 分解（W ≈ W_high + scale×W_low）思路是不动精度敏感模块的数据精度、只迁移计算路径——把 FP32 计算整体搬到 BF16 Tensor Core，两次 GEMM 融合进单 Kernel、全程无 HBM 往返，相比 cuBLAS FP32 加速 2.86x~3.22x^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:67-83]。[[concepts/moe-mixture-of-experts-2025|MoE]] 侧的 FusedMoE 更彻底，把路由索引、Gate-Up、激活量化+Down、加权聚合、PDL 串联五个阶段整合成一条无额外 HBM 往返的一体化流水线，也正呼应了 [[entities/2026-05-04-DeepSeek做大-Mega-MoE-Tri-Dao团队加快-SonicMoE-机器之心|SonicMoE 等 MoE 推理加速]] 的行业方向^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:84-113]。

融合则分层推进：QKV 投影后的 Rope+Norm+Hadamard+Quant+StoreKV 是典型的低算力强度 Element-wise 链，收益来自把多次 HBM 往返压减为寄存器级流转加一次写出^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:115-129]；AllReduce+Norm+Add 把张量并行下的通信、残差与归一化合并为 NVLink 原生操作，并按 Prefill 大 Token 与 Decode 小批量分别提供 NVSwitch 多播与 Lamport P2P 两个内核版本^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:130-143]；采样链路从十余个零碎 Kernel 收敛为 2 个，配合全词表单次加载与 GPU 内闭环重复惩罚，消除的不仅是算子开销，还有 CPU-GPU 同步拷贝^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:145-176]。最激进的一步是 GEMM 与 ReduceScatter 的细粒度通算融合：SM 显式划分为计算与通信两类角色，Tile 级重叠让端到端加速比稳定在 1.68x~1.81x，且序列越长覆盖率越高^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:177-241]。

### 并行策略：Prefill 与 Decode 的双轨设计

团队没有追求一套全局统一的并行方案，而是按两个阶段的负载特征分轨建模。Prefill 侧，纯 TP8 存在 token-wise 算子冗余计算、AllReduce 全量交换、Grouped GEMM 被切至极窄 shape 三重代价，改用 SP 拆分 + 通算融合 + 通信量化的组合后，32k 上下文 TTFT 从 1885ms 降到 1424ms^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:242-279]。Decode 侧走 [[concepts/inference-optimization|推理优化]] 中典型的 Attention DP + MoE EP 跨节点混合并行：扩大 EP 把权重分散到多机、腾出的显存转产 KV Cache，跨节点聚合 batch 又把 Grouped GEMM 从 Memory-bound 推回 Compute-bound 区域，端到端吞吐提升 15.7%~44.7%，配套的 Async EPLB 还把每步仅一层的权重重排用 NCCL P2P 藏进前向计算之后^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:280-304]。

### 多级缓存与 MTP 异步调度：向调度要性能

缓存部分把 Prefix Cache 从显存容量问题重新定义为存储体系问题。Agent/Coding 场景的长上下文与公共前缀使仅靠 GPU 显存的缓存面临容量受限、淘汰加速、重复计算、跨实例不可复用四重瓶颈；GPU→CPU→KVStore 三级体系按 L1→L2→L3 顺序查询、命中即跳过对应 Prefill、新 Block 异步下沉，在不增加显存占用的前提下扩大有效缓存容量^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:305-327]。这与测试集"缓存理论命中 80%"的设定互相印证——对多轮长上下文业务，缓存命中率就是第一成本杠杆。MTP 侧则打破传统异步调度"每轮稳定生成 1 个 token"的假设：多层 [[concepts/speculative-decoding|投机解码]] 引入动态接收长度，CPU 被迫等 GPU 验证结果同步；解法是数据准备一律按最大接收长度乐观推进、下一轮计算前再以真实验证结果修正关键值，消除 decode 间 5~10ms CPU 气泡、端到端提升 10%~20%^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:328-338]。

### 量化与稀疏：压缩与剪枝并举的显存突围

W4A8 与 Attn FP8 直接应用会因极低比特表示和激活离群值导致精度显著退化，AngelSlim 用"GPTQ 权重重建 + 激活平滑与旋转 + QAT 轻量微调"三级联用把与 BF16 的差距压回 1% 以内、吞吐提升 28%+；其中对 Q/K 施加正交旋转打散离群值的 Hadamard 变换可融合进推理 Kernel，几乎零额外开销^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:339-387]。稀疏注意力则针对 256K 上下文下自注意力二次方复杂度这一 [[concepts/attention-mechanism|注意力机制]] 固有瓶颈：Stem 算法配合 HPC-BSA 算子，在仅 25% 计算预算下逼近稠密精度，128K Prefill 延迟降低 3.6 倍^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:389-437]。它的两个关键设计各有明确的架构洞察——Position-Decay 源于"头部 token 误差逐层递归放大、尾部影响局部化"，用总预算不变的再分配保护递归依赖链；Output-Aware Metric 则指出注意力分数只是路由概率而非信息贡献，把 Value 模长以对数形式并入选择标准，因此可直接复用标准 Top-k 内核。后续推进的 C4/W4 量化与并行投机解码，说明显存与吐字速度被定位为下一阶段的两个主攻方向^[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md:445-452]。

## 实践启示

1. 算子优化的优先级应按算力强度与访存特征排序：低算力强度的 Element-wise 链（Rope/Norm/Quant/Store）优先融合成单 Kernel，风险最小且收益可达 5x 级；数值敏感模块（如 Router）不动数据精度、只迁移计算路径（FP32→双 BF16 Tensor Core），是兼顾精度与性能的通用套路。
2. 并行策略不要一套方案打天下：Prefill 追求 TTFT 用序列并行 + 通算融合 + 通信量化，Decode 追求吞吐用 EP 扩大换 KV Cache 显存 + 跨节点聚合 batch，两阶段分别对齐各自的 SLO（50ms TPOT、4s TTFT）再选型。
3. 长上下文业务的成本杠杆首先在 Prefix Cache 命中率：把缓存从显存单层扩展为 GPU→CPU→KVStore 三级，工程量远小于算法改造，对多轮对话与公共前缀场景收益直接（该文测试集理论命中 80%）。
4. 遇到"CPU 准备依赖 GPU 结果"的同步链路，优先尝试"按最大值乐观推进 + 计算前用真实结果修正"模式：凡下游消费方可以事后纠错的场景（MTP 动态接收长度即典型），都可用少量纠错成本换掉整段同步等待。
5. 量化落地要打组合拳：单点 W4A8 会显著掉精度，GPTQ 重建 + 激活平滑/旋转 + 轻量 QAT 三级联用才能做到差距 <1% 且吞吐 +28%；其中能融合进推理 Kernel 的旋转变换（如 Hadamard）几乎白拿收益。
6. 稀疏化的预算分配比预算总量更关键：同样 25% 计算预算，按位置衰减重新分配、并把"信息贡献"（Value 模长）而非"路由概率"（注意力分数）作为剪枝依据，即可逼近稠密精度——这提示 token 剪枝策略应从注意力分数本身跳出来。

## 来源

- 原文: [[raw/articles/2026-06-26-腾讯混元AI-Infra如何优化Hy3-Preview-一次大模型推理性能提升的-腾讯技术工程.md|腾讯混元AI Infra如何优化Hy3 Preview 一次大模型推理性能提升的 腾讯技术工程]]
