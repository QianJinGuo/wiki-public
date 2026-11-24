---
title: "Flattening Every Memory Peak in Long-Context Mixture-of-Experts Training"
created: 2026-09-24
updated: 2026-11-21
type: entity
tags: [moe, training, memory-optimization, long-context, distributed-training, inference-optimization]
sources: [raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026]
confidence: 0.8
provenance_state: extracted
---

# Flattening Every Memory Peak in Long-Context Mixture-of-Experts Training

## 核心问题

MoE 模型在长上下文或大 batch 训练时的失败模式是：**任何一个组件的内存峰值超过显存就会 OOM**，所以优化目标必须同时压住所有峰值，而不是只降平均占用。论文指出有四个内存峰值在常用并行方案下不受约束，且各自随不同维度增长：

1. **Expert dispatch** — 随 routing matrix 增长
2. **Vocabulary projection** — 随 tokens × vocab 增长
3. **Gradient checkpoint boundaries** — 随 depth × sequence length 增长
4. **Optimizer state** — 随参数量增长

哪个先爆取决于模型、上下文长度和设备数——只压最大那个只会让下一个暴露出来。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md]

## 四项技术（全部只改计算/数据搬运的顺序与粒度，loss 与梯度保持精确）

| 技术 | 针对峰值 | 机制 |
|------|---------|------|
| **PipelinedLLEP** | expert dispatch | 在 least-loaded expert parallelism 基础上，对每个 source 贡献到 dispatch chunk 的 token 数加 cap |
| **Ring-DTP** | vocabulary projection | 激活/权重分片沿 ring 循环，logits 分块折叠进 online log-sum-exp |
| **SCO (Selective Checkpoint Offload)** | checkpoint boundary | 每个 checkpoint boundary 只有一个长生命周期 tensor，保留在 CPU 内存 |
| **OffloadStreamAdamW** | optimizer state | 把 CPU Adam offload 的串行更新改成 bucket pipeline |

^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md]

## 实测结果

- MoE dispatch 峰值最多降 **59.3%**（吞吐无损）
- vocabulary projection 峰值降 **86.6%**
- offload optimizer step 快 **2.05×**
- 在 120B–667B 参数 MoE 模型上组合使用：**1M context length 训练**，达到调优 FSDP2 baseline 的 **8–32× 上下文 reach** 与最高 **10.4× 吞吐**

^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md]

## 与 wiki 已有内容的关联

- 与 [[entities/2026-05-04-DeepSeek做大-Mega-MoE-Tri-Dao团队加快-SonicMoE-机器之心]] 同属 MoE 训练内存优化谱系——SonicMoE 优化的是 MoE 前向/激活内存，本文系统性覆盖 dispatch/vocab/checkpoint/optimizer 四类峰值
- 与 [[entities/deepseek-moe-parallel-strategy]] 的并行策略互补：本文不改并行计划，在现有 FSDP/EP 方案之上做调度级约束
- 长上下文训练 reach 与 [[entities/750b-moe-pd-disaggregation-aws-efa-vs-roce]] 的 PD 分离部署构成训练-推理两端的长上下文工程拼图

## 相关链接

- arXiv: https://arxiv.org/abs/2609.14306（cs.DC）
- 作者：Shrey Pandit 等 4 人

## 深度分析

### 每个峰值单独有界、可独立启停的设计哲学

标准并行栈（ZeRO-3 分片持久状态、sequence parallelism 压注意力激活、expert parallelism 切专家权重）已经把"可分片的"都分掉了，剩下的恰恰是四个高度随 workload 维度增长、不随分片度收缩的 live set。论文由此立下两条约束：每个算子可单独启用，训练只为自己的峰值付费；且只改计算与数据搬运的顺序和粒度——不做低秩适配、量化状态或近似路由。精确性换来度量红利：内存与吞吐可与全参数 BF16 训练直接对比，质量只需一次端到端 SFT 验证（gpt-oss-20b 在 128K 上下文做数学微调后 AIME 2025 与基线持平）。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:43-45] ^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:399-403]

### 从运行时事故到 launch 前可预检的闭式预算

组合后的每 rank 峰值是一个闭式和式：权重梯度、优化器 staging、注意力激活、dispatch、logit strip、checkpoint boundary 六项，每一项在 token 上限设定后由模型或启动配置唯一确定。这是四算子存在的真正目的——把随 routing matrix 等运行时行为波动的高度改写成配置固定的量，让一个配置在启动前就能验算可行性，OOM 从运行时事故变成规划期约束。实验佐证了确定性：65K token sweep 重复三次，七个峰值复现到 milli-GiB 级。这与 [[entities/notes-on-pretraining-parallelisms-and-failed-training-runs]] 中"并行配置失败往往中途才暴露"的经验形成方法学对照。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:215-219] ^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:436]

### 分块的两个隐藏陷阱：不均衡复活与 autograd 引用生命周期

其一，LLEP 靠把 token 按 destination 排成连续索引再平衡，而按连续位置切块会让每块的 route 几乎全涌向同一 rank——不均衡被 chunk 边界原样复活；改用 strided 等间距选块才摊平每块负载。由此得到的 bound 是"路由器推不动的保证"，而不是对流量的描述：缓冲区按实际分片数分配，bound 处处被尊重却不必被达到。其二，朴素的 per-chunk 流水线并不省内存：循环里每块都调一次 expert 层，autograd 图把所有块的中间张量攥到 backward 才放。解法是把每块 expert matmul 包进 reentrant 内层 checkpoint、嵌在 decoder 层的 non-reentrant checkpoint 里——若用 non-reentrant 内层，前向函数与参数会以强 Python 引用滞留在 unpack closure 中，游离于 saved-tensor 机制之外，堆叠专家权重将存活整个前向到反向周期；且只 checkpoint expert matmul 而非整块，否则三阶段在 Python 视角串行化，挡住下一块的 dispatch 预取。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:104-120] ^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:403-416]

### 换一个提问方式：与既有系统的分界

FasterMoE、Lina、PipeMoE、ScheMoE 都对 dispatch 做分区以重叠通信，但它们为带宽利用率选分区数，施加后足迹仍跟随实际负载；PipelinedLLEP 问的是不同的问题——"一个 destination 最多可能收到多少"，用 per-source token 预算回答。词投影一侧，Cut Cross-Entropy、Liger 这类 fused kernel 在单卡上流式算 log-sum-exp，Megatron vocab 并行分权重但要求每 rank 持有同一 batch（等效 batch 缩水）；Ring-DTP 用 data-parallel 式布局让各 rank 保留自己的 batch，沿 ring 循环权重分片或激活，把每块 logits 折进 online log-sum-exp 的三标量状态（running max、指数和、target logit），logit 张量从未成形；move-weights 分支随上下文长度扩展性最好，auto 模式动态二选一。routing 不均衡这条线与 [[entities/ultaep-moe-expert-load-balancing-300us]] 的专家负载均衡、[[entities/deepseek-v3-moe-architecture]] 的 auxiliary-loss-free 路由同属一个问题谱系。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:243-252] ^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:253-260] ^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:126-130] ^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:151-163]

## 实践启示

1. **先盘点全部随 workload 增长的峰值，再谈优化**：最先爆的峰值随模型、上下文长度、设备数变化，只压最大者等于把预算让给下一个。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:37-42]
2. **优先改调度而非数值**：顺序与粒度层面的重排（分块、ring、offload、流水线）保住精确 loss 与梯度，收益可直接对比；低秩、量化、近似路线每一步都要额外付质量验证成本。
3. **把内存目标写成启动配置的函数**：工作集只依赖模型与 launch 配置的设计，才能把 OOM 检查提前到 launch 前；闭式预算比事后 profile 可靠。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:215-219]
4. **分块不会自动省内存**：连续切块复活 routing 不均衡，chunk 循环经 autograd 攒住全部中间张量；strided 选块加 reentrant 内层 checkpoint 才让 bound 在 backward 中仍然成立。
5. **offload 的真实瓶颈是 host 链路往返**：CPU 更新期间 GPU 空闲且 HBM 近乎全空，把这段空转改成流式回 GPU 更新的 bucket pipeline 换来 2.05× 提速，且实测 transfer-bound、对 staging 深度不敏感。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:182-190] ^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:200-205]
6. **警惕快互联假设**：all-to-all 与 ring 流量都假设快速互联，更慢 fabric 上 chunk 数的代价结构会变；token 预算收紧会同比抬高 chunk 数，host 容量与链路带宽终将成为新瓶颈。^[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026.md:276-283]

- 来源：[[raw/articles/flattening-memory-peaks-long-context-moe-training-arxiv-2026|原文存档]]
