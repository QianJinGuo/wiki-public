---
title: "AI Infra进阶：如何让大模型输出确定的结果"
created: 2026-08-13
updated: 2026-11-20
type: entity
tags: [ai, research, rl, reinforcement-learning, post-training, inference, llm-inference]
sources: [raw/articles/ai-infra进阶如何让大模型输出确定的结果.md]
confidence: 0.6
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# AI Infra进阶：如何让大模型输出确定的结果

> WeChat-腾讯技术工程 | 发布于 2026-08-05 | 评分入库 v×c≥49

## 核心内容

原创 腾讯程序员 2026-08-05 17:25 广东 一文进阶AI Infra 作者：binnnliu 你有没有发现跟大模型对话，同样的提示词每次结果都不一样。 这符合我们对于“大模型的本质就是个猜词器”的一贯认知。 然而，可重复性是科学进步的基石。因此让大模型输出完全确定的结果是一个非常值得研究的问题，特别是强化学习需要确定性的 Rollout，来保证实验的可复现性和训练过程的稳定性。 Reproducibility is a bedrock of scientific progress. 其实很多同学第一时间会想到： 同样的硬件，同样的提示词： 1. 是不是把 Temperature 设置为 0，关掉随机采样就好了？ 2. 如果业务需要 Temperature 0，那把随机数种子（Seed）锁死，是不是也能保证每次输出一样？ 我已经测试过了，环境：vllm serve Qwen/Qwen3-8B 答案： 是也不是。 是（Run-to-run 确定性）： 同一时间，没有其他请求时，多次同样的提示词请求，回答是一样的； 不是（Batch Invariance / 批次不变性）： 同一时间，有其他请求时，多次同样的提示词请求，回答是不一样的； 什么！难道不同的请求间会相互影响？不可能，不同请求间的注意力机制是严格物理隔离，不存在上下文污染的问题。 其实这一切的根源是推理引擎底层的动态组批与算子调度策略引发了浮点加法的顺序变化，而浮点加法不满足结合律：(a + b) + c ≠ a + (b + c)。 从系统设计的角度来看，导致结果波动的根本原因，不在于信息干扰，而在于系统为。^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md]

## 关键要点

- 原文完整记录：[[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md|原文存档]]
- 关联主题："Agent 架构"、"Agent 评估基准体系"

## 深度分析

### 一个反直觉的实验事实：Temperature=0 只保证 Run-to-run，不保证批次不变

作者在 `vllm serve Qwen/Qwen3-8B` 上的实测把"确定性"拆成了两层^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:35-45]：同一时间、没有并发请求时，同样的提示词多次请求结果一致（Run-to-run 确定性）；一旦有其他并发请求，结果就会漂移（Batch Invariance 缺失）。这推翻了"Temperature=0 或锁死 Seed 就能确定"的直觉——请求间的注意力严格物理隔离，不存在上下文污染；波动的真正根源是底层动态组批与算子调度改变了浮点加法的执行顺序，而浮点加法不满足结合律 (a+b)+c ≠ a+(b+c)。从系统设计视角看，这是硬件为掩盖访存延迟而触发的非确定性并行优化，而非信息干扰。对强化学习而言这不是理论洁癖：确定性 Rollout 是实验可复现与训练稳定的前提。

### 浮点本质与 FP16 舍入的最小反例

浮点数的本质是科学计数法，在有限位数内做数值范围与绝对精度的 trade-off：ULP(x) = 2^(E-M)，指数越大步长越大。最小反例是 FP16 的 2048+1：2049 恰好落在 2048 与下一个可表示数 2050 的正中间，按"向偶数舍入"规则被强行舍回 2048^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:47-49]。但作者澄清了一个常见误读：真实推理中 GEMM 是 FP16/BF16 输入、FP32 累加，批次差异依然存在——根因是规约拓扑改变，而非 FP16 存储精度太低^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:53]。真正的拨动者则是 [[entities/gpu-kernel|GPU Kernel]] 的调度参数。

### GEMM 调度参数对确定性的分级影响

内存墙（Tensor Core 算力增速远超显存带宽）驱动了 Tiling、Split-K、Swizzle 这条优化路线^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:74,86,122-126]：Tiling 化整为零复用 SRAM，Split-K 在 M/N 极小、K 极大时把 K 切给多 CTA 并行，Swizzle 重排 CTA 调度顺序以复用 L2。但分块参数的动态变化会重塑浮点累加的归约拓扑，这正是动态 Batch 下丢失批次不变性的根本原因^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:145]。逐参数判定（关键分界）：BLOCK_M/BLOCK_N 与 GROUP_M/Swizzle 只做空间映射与调度排序，不改变单个元素的点积累加逻辑；BLOCK_K 改变 Tensor Core 内 MMA 指令的累加树拓扑，引入"确定性误差"（同一 BLOCK_K 结果恒定，不同 BLOCK_K 结果不同）；SPLIT_K 则引入非确定性误差——atomic_add 的加法顺序取决于硬件调度，即使用 Workspace Reduction 有序合并，段数一旦随输入动态变化，累加树拓扑照样变^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:249-265]。num_warps/num_stages 属于资源分配与流水线调度，不碰规约树。结论：推理引擎在性能与确定性之间存在固有的架构冲突^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:270]。

工程修补按硬件代际分层：SM90/SM100（Hopper/Blackwell）唯一的批次方差来源是 Split-K，锁死即可；SM8x（Ampere）则需同时锁 BLOCK_K 与 Split-K——且这两条结论都是实测得出的，因为 cuBLAS/cuBLASLt 闭源^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:311-313]。[[entities/vllm|vLLM]] 的 `matmul_persistent` 走得更彻底：按 dtype 硬编码一套分块参数、彻底关掉 autotune，与输入 M/N/K 无关^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:352]。注意两处隐蔽边界：compile mode 下 aten override 不生效^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:346]；SM80 上 cuBLASLt 不支持禁用 BLOCK_K，只能整体换 Triton persistent 实现^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:335]。

### 同一条主线贯穿 RMSNorm、Attention 与 NCCL

RMSNorm 侧，block_size 决定线程本地累加分组与 BlockReduce 树形（层数、warp 分组），它一变加法顺序就变^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:478]；开启 Batch Invariance 后的做法是固定 reduction 宽度、与 num_tokens 彻底解耦，实现跨 batch bit-exact^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:513]。编译态最反直觉：TorchInductor 的 Split-Reduction 启发式里，hidden_size ≤ 8192 时 split 恒为 1、归约保持单程，天然批不变；一旦 hidden_size 超限或显式配置 compile_ranges，静态图与动态图的累加树形态会发生结构性突变^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:609-611]。Attention 侧（[[concepts/attention-mechanism|注意力机制]]），FlashAttention 的 tile 取值只由 (head_dim, causal, dtype, arch) 决定、是编译期常量，真正破坏批不变的是切 KV 规约轴的 SplitKV/num_splits——其启发式输入正比于 batch^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:653]；decode 阶段的 FlashDecoding 更是按 (batch, heads, seq_kv, num_sms) 动态决定段数，段数一变 LSE 合并的规约拓扑就变^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:675-677]。跨设备层是第三次重演：NCCL 不保证跨 Batch Size 采用相同规约算法，All-Reduce 消息大小按 [num_tokens, hidden_size] 计算，跨阈值即在 Ring 与 Tree 间切换，多通道并发还引入拓扑异构与切分边界的动态滑动^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:710-718,727-745]。vLLM 的对策是把规约拓扑整体钉死：禁用 Custom All-Reduce，NCCL 固定 Tree + Simple 协议、通道数锁 1，并关闭 NVLS/CollNet 等替代拓扑^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:752]。三层的答案最终归一：阻止系统根据 Batch Shape 动态改变 Reduction Topology^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:780-784]。

## 实践启示

1. **要批次可复现，开 Batch Invariance，而不是只锁 Seed**：动态组批下 Temperature=0 与固定种子都救不回批次间的漂移^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:35-39]。RL 训练、离线评测、回归基线这类必须逐 bit 对齐的通道，应显式开启 `VLLM_BATCH_INVARIANT` 并验证生效路径。
2. **确定性是用吞吐换的，预算要提前算**：锁 Split-K/BLOCK_K、强制 num_splits=1、单通道 NCCL 都直接降低 SM 利用率，decode 小 batch 场景代价最明显^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:691,752]。不要全局常开：确定性通道与性能通道分池部署更划算。
3. **优先 SM90+ 并认清代际差异**：Hopper/Blackwell 上只需禁 Split-K；Ampere 还要额外锁 BLOCK_K，裸 torch.mm 甚至要整体换 Triton persistent kernel^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:311-313,335]。同一"确定性开关"在不同代际覆盖面不同。
4. **编译配置会悄悄破坏确定性**：compile mode 下 aten override 不生效^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:346]；hidden_size > 8192 或显式 compile_ranges 会让 Inductor 的 Split-Reduction 拓扑改变^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:609-611]。任何 torch.compile / 版本 / 编译参数变更后都应重跑 bit-exact 回归，而不只看吞吐。
5. **把"规约拓扑"当作一等排查对象**：结果抖动不应停在采样参数层，而应沿 BLOCK_K → Split-K → num_splits → NCCL 算法/协议/通道数逐层定位^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:249-265,718]。这也是 [[entities/martin-fowler-的-ai-研发提醒非确定性进了研发链路harness-才真正开始承重|非确定性进入研发链路]]时 harness 层必须承重的底层原因。
6. **AI Infra 的本质是在多层抽象间找性能与数学等价性的最优 Trade-off**：从 Triton/CUDA 算子层到 Inductor/IR 编译层再到 NCCL 通信层，同一问题三次重演^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:788]；而 MoE 架构特有的批不变性与更深的 NCCL 通信拓扑仍是尚未覆盖的坑^[raw/articles/ai-infra进阶如何让大模型输出确定的结果.md:786]。

## 相关实体

"Agent 架构" "Agent 评估基准体系"
