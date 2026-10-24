---
title: "LMSYS 在 H20 上服务 DeepSeek-V4-Pro 的场景化 Serving 优化"
created: 2026-08-20
updated: 2026-10-25
type: entity
tags: [ml, inference, serving, moe, lmsys, speculative-decoding, optimization]
sources: [raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LMSYS 在 H20 上服务 DeepSeek-V4-Pro 的场景化 Serving 优化

## 概述

LMSYS 团队发表了一篇关于在 **NVIDIA H20 GPU**（而非 Blackwell）上服务 1.6T 参数 MoE 模型 **DeepSeek-V4-Pro** 的系统优化长文。核心论点是「一个模型需要多个 serving profile」：没有一种通用配置能同时满足长上下文 prefill、交互式 decode 的 TPOT、以及高并发吞吐的 KV 容量约束，必须按工作负载、SLO 与实测硬件行为分别选择部署拓扑与执行路径。^[raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08.md]

## 核心方法论：场景化 Serving Profile

文章提出「场景驱动」的 serving 方法论：prefill 依据实测 context-length 在 **PP2 与 PP4** 间切换，decode 则按低延迟/高吞吐/KV 容量分别优化。关键设计决策包括：^[raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08.md]

- **Prefill 用 MoE-TP 而非 MoE-EP**：MoE-EP 虽只交换路由 token，但真实 prefill 流量存在显著 expert 倾斜，hot expert 所在 rank 成为 straggler；MoE-TP 的全序列 all-gather/reduce-scatter 走 NVLink 高带宽、成本可预测，「可预测的通信比不可预测的不均衡更便宜」。
- **容量取舍**：用 **Humming MXFP4AFP8**（MXFP4 expert 权重 + 在线 FP8 激活）缩减权重足迹，叠加 **Online C128** 压缩 KV 辅助状态，组合容量提升至 baseline 的 3.88×（DP32-EP32）/ 10.14×（PP2-TP8）。
- **低延迟 decode 用 PP2-TP8，高吞吐用 DP32-EP32**：单节点 TP8 最快但 KV 容量受限；PP2-TP8 跨节点分摊权重换取 KV 空间；DP32-EP32 扩展 EP 组释放 HBM 提升并发容量（256K/512K/1M 并发请求 2× 扩展）。

## 关键量化结果

论文给出了 H20 与 B300 的对比与多场景实测：^[raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08.md]

- batch size 1 时单节点 H20-141GB 达 **271 tokens/s**，对比 B300 的 383.7 tokens/s，优化后观测差距缩至 **1.42×**（尽管 B300 峰值 Tensor Core 算力为 H20 的 45.6×）。
- 优化 prefill 达 **8.45k input tokens/s/node**，1M-token prompt 处理耗时 **43.7s**；高吞吐 decode（DP16-EP16）达 **4.67k output tokens/s/node**，平均 TPOT 27.4ms。
- DSpark 投机解码优化后 peak TPOT 在 batch size 1 下降 **74.8%–78.0%**，大 batch 仍降 52.2%–60.0%；Humming decode 热路径融合 SwiGLU 量化后 per-GPU 吞吐提升 **44.0%**。

## 深度分析

### 场景化 profile 的必要性：prefill 与 decode 的瓶颈不在同一处

prefill 不保留跨请求状态，成本由 TTFT、算力与通信决定，H20-96GB 容量就够；decode 要为每个活跃请求驻留 KV，HBM 容量直接成为上下文长度与并发的硬上限，于是换 H20-141GB——同模型按角色配两种硬件，已说明全局配置不成立。prefill 内部同理：PP2 在 4K/32K 把 TTFT 压低 16.7%/19.5%，8K/16K/64K 相差不到 2%，128K 后 PP4 反超 26.2%–44.8%，任何单一 pipeline 深度都会在某些长度区间处于劣势，切换点是实测 context 分布导出的策略。decode 同理：单节点 TP8 路径最短，1M 时却只剩 batch 1；PP2-TP8 多付 pipeline 开销换 4/8/16 的 batch 容量。「最快拓扑」与「最可服务拓扑」在此分叉，单配置的本质是替你提前选定违反哪条 SLO。

### DP16-EP16 与 TP 之争：可预测的通信比不可预测的不均衡更便宜

prefill 侧弃 MoE-EP 用 MoE-TP：EP 只搬路由 token，真实流量却 expert 严重倾斜，持热专家的 rank 成为 straggler，combine 时全体等最慢分支，通信量降了 TTFT 没降。MoE-TP 的 all-gather/reduce-scatter 字节更多，却走 900 GB/s NVLink、成本稳定，倾斜变不成 rank 级长尾。判据因此从搬多少字节换成尾部时间由谁决定：H20 上算力稀缺，NVLink 相对未退化，多付字节换满负荷划算。高吞吐 decode 反向选 EP，约束是 HBM 容量：EP 组越大权重摊得越开（256K/512K/1M 并发 8/4/2→16/8/4）。DP16-EP16 在 16/32 并发下 per-GPU 高 3.6%–20%，但扫描非单调，只宜作效率参考，见 [[entities/deepseek-moe-parallel-strategy|MoE 并行策略]]。

### 投机解码的收益曲线由 batch 决定，而非由算法决定

no-spec 的 bs1 peak TPOT 为 25.72–26.42ms，优化后降到 5.76–6.77ms，降幅 74.8%–78.0%；最大共享 batch 上只剩 52.2%–60.0%。衰减是结构性的：bs1 每步要读完全部专家权重才产一个 token，算术强度极低、被带宽钉住，而验证 k 个 draft token 的前向在字节层面几乎不增加权重读取，加速比可逼近每步接受的 token 数；batch 一大，读取摊到几十行，step 转为验证 GEMM 算力受限，收益回落到接受率减验证开销。投机红利正比于算力有多闲：H20 算力穷而带宽相对富裕，271 tokens/s 才逼近 B300 的 383.7（1.42×）；上界既被算力比锁死，后续要落在验证路径，refinement 逐行 dot-reduce 合成一次转置 GEMM 即换来 22.8%。见 [[concepts/speculative-decoding|投机解码]]。

### 量化融合 SwiGLU：+44.0% 来自减少往返，不来自少算

融合 SwiGLU 与量化的 kernel 直接产出 W2 所需的 data 与 scale，省掉独立量化 pass 与对中间 buffer 的重复访问，W2 因而更早启动；算术量没变，同 GEMM 同公式，变的是 HBM 往返次数、launch 数量与依赖链长度。decode 里这笔账值钱：step 很短（bs32 也只有 19.23–24.78ms），launch 间隙加一趟 activation 读写就占掉可观比例；删掉中间 buffer 还缩短串行段，使 W2 首批 tile 早启动。同一逻辑解释 MXFP4AFP8：H20 无原生 FP4 Tensor Core，位宽下降不自动变成算力节省，须先反量化到 FP8，它买的是权重足迹（1.71×/4.47×）与访存流量而非 FLOPs，代价是 GSM8K1000 的 95.5% 对 95.0% 门槛只剩 0.5pt。见 [[entities/quantization-techniques|量化技术]]。

### 受限硬件下的服务经济学与可复现基准

45.6× 峰值算力差只兑现成 1.42× 观测吞吐差，说明负载从未逼近峰值算力；任何 H20 vs B300 的单点比值不带 batch、context、concurrency 三坐标都无意义。投入排序也随之改变：Blackwell 上算力富余，省算力是主战场；H20 上 HBM 容量与通信可预测性才是主战场，Humming + Online C128 把容量做到 3.88×（DP32-EP32）/10.14×（PP2-TP8）比算子微调更值钱，而投机解码本质是用闲置算力买回带宽受限的时间。方法同样可迁移：吞吐取自服务端 TP0 的 Decode batch 日志、丢弃最高最低各 20% 后取均值以避开排队噪声与 warmup 抖动，A/B 严格同 operating point，并附精度门槛 95.0%——缺此三件，提升 X% 无法复核。参见 [[entities/sglang-inference-deployment-practice-benchmark-tuning|SGLang 部署与调优基准]]。

## 实践启示

1. **按流量类切 profile，而不是按模型切。** prefill 依实测 context 分布在 PP2/PP4 间切换（4K/32K 下 PP2 快 16.7%/19.5%，128K–1M 下 PP4 快 26.2%–44.8%，8K/16K/64K 差异 <2%）；decode 至少分出低延迟与高吞吐两套 profile。
2. **先定位 binding resource 再选拓扑。** decode 先算 full-token capacity：1M 下单节点 TP8 只够 batch 1，PP2-TP8 可到 4，512K 到 8，256K 到 16；prefill 才轮到算力与通信效率。
3. **TTFT 与 TPOT 分开测、分开报。** 两者由不同资源决定（prefill 看算力/通信，decode 看 HBM 容量与带宽），合并成一个平均数会正好抹掉你要做的那个 trade-off。
4. **在自己的 batch 分布上测投机解码，报曲线不报峰值。** bs1 的 74.8%–78.0% 与大 batch 的 52.2%–60.0% 是两个不同的产品承诺。
5. **把量化当系统问题。** 先数它省掉几次 HBM 往返与几个 kernel launch（融合 SwiGLU +44.0%），再核对精度余量（95.5% vs 95.0%，仅 0.5pt），换模型必须重跑精度门。
6. **MoE 并行按尾部时间选，不按通信量选。** prefill 用 MoE-TP 消除 expert 倾斜造成的 straggler；高吞吐 decode 用 EP 换 KV 容量，EP 组越小节点内流量占比越高（DP16-EP16 高 3.6%–20%）但不单调，不可当普适替换。
7. **基准可复现三件套：服务端日志取值、裁掉头尾 20%、同一 operating point 的匹配 A/B。** 缺一项，跨团队的「提升 X%」都不可比。

## 与既有知识的关联

这篇文章是 LMSYS serving 优化系列的重要延续，与 [[entities/lmsys-dflash-speculative-decoding-2026-06|DFlash 投机解码]]、[[entities/agent-assisted-sglang-development-lmsys-2026-07|LMSYS Agent 辅助 SGLang 开发]] 同源同团队；其 DSpark 投机解码路径与 [[entities/deepseek-dspark-v4-speculative-decoding-deepspec|DeepSeek DSpark V4]] 一脉相承。方法论层面可对照 [[entities/ai-infra-llm-efficient-inference-vllm|LLM 高效推理]] 与 [[entities/native-speed-vllm-transformers-modeling-backend|vLLM Native Speed]]，其「可预测通信优于不可预测不均衡」的 MoE-TP 论证是通用可迁移结论。

→ [[raw/articles/lmsys-deepseek-v4-pro-serving-h20-engine-optimization-2026-08|原文存档]]
