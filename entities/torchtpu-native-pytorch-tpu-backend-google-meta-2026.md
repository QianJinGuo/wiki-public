---
title: "TorchTPU: 为 PyTorch 原生构建的 TPU 后端（Google × Meta）"
created: 2026-10-21
updated: 2026-10-24
type: entity
tags: [pytorch, tpu, xla, training, inference, compiler, distributed-training, infra]
sources: [raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026, raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx]
confidence: 0.8
provenance_state: extracted
---

# TorchTPU: 为 PyTorch 原生构建的 TPU 后端（Google × Meta）

TorchTPU 是 Google 与 Meta PyTorch 团队的开源合作（2025 年中启动），目标是让 PyTorch 在 Google TPU 上**原生**运行，而不是把 PyTorch "塞进 JAX 的思维方式里"。官方给的定位很直白：开箱即用的模型执行与 TPU 迁移，同时通过 eager 执行优化、torch.compile 集成与硬件原生加速拿到有竞争力的训练/推理性能。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

## 为什么 PyTorch/XLA 不够：一次后端设计复盘

文章花了不小篇幅复盘 2020 年的 PyTorch/XLA（基于 Lazy Tensors）：它虽然奠基，但跟不上 PyTorch 社区演进——需要在源码里显式标注可编译区域（侵入式改动）、编译时延严重、不支持 PyTorch 原生 eager 执行、对新出现的分布式原语支持滞后。TorchTPU 与 TorchAX 的差别被明确点出：后者的路线是把 PyTorch 迁就 JAX 的运作方式，而 TorchTPU 选择做**一个有原则的 TPU 后端**——把 eager 当第一优先级，让既有 PyTorch 工作负载几乎零改动就能跑。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

这个复盘对做框架/编译器集成的人有通用价值：**"能编译"不等于"能用"**，编译期延迟、eager 语义缺失、分布式原语滞后这三件事，任何一项都会让一个后端在生产里被判死刑。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

## 设计原则与实现要点

四条原则构成了实现取舍的依据：基于 Torch Dynamo（而非 Lazy Tensor）的 `torch.compile()`，目标是与 JAX 编译相当的性能；在 PyTorch 栈最底层（Process Group API 等）暴露 TPU collectives，从而广泛支持 DDP、FSDP、DTensor 等分布式惯用法；与 PyTorch 生态（torch.profiler、DTensor、Ray）无缝；把工作负载迁到 TPU 应该简单到"改一个 `.to('tpu')`"——作者同时诚实指出，仅靠这一改拿不到峰值性能，因为模型的 shape、sharding 与算子仍偏向 GPU 架构。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

落到具体优化，几个手段值得记录：Pallas 内核层面的 Tokamax Splash Attention（文章称为显著的性能提升来源）、融合的 cross-entropy loss 内核、MoE 层的 grouped matmul；结构层面把 94 个 decoder layer 合成一个 scan 循环，对 decoder 层输入做 activation offloading，并用 collective pipelining 让通信与相邻层计算重叠；并行策略上，单 slice 用 FSDP（经 SimpleFSDP），多 slice 用 HSDP；精度上 BF16 混合 FP32（grad reduce、RoPE、优化器状态均用 f32）；参照配置为每 TensorCore local batch 16、序列长度 4096。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

另一个工程细节是拓扑映射：TorchTPU 提供机制去取 rank 对齐的 JAX Mesh，使内核能继承 PyTorch ProcessGroup 的拓扑，从而在 full-world 与隔离子组两种形态下都高效执行。这正是"在框架最底层暴露 collectives"这条原则的具体兑现。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

## 性能与验证口径

官方给的训练侧数字是逐层优化后的累积效果：从开箱即用的 eager 基线出发，FLUX 模型在 TorchTitan + TorchTPU 上先提升到 3.02s/step（3.80× 加速），再叠加目标模型的 peephole 重写后到 2.78s/step（1.08 samples/s）；多机集群上表现为吞吐随规模线性增长、MFU 基本持平——这是"扩展效率没被通信吃掉"的直接信号。推理侧则引用第三方审计：TPU v7 搭配 vLLM-TorchTPU 栈，在性能/美元上具备竞争力；已验证的栈包括 vLLM、SGLang（多机推理）与 TorchTitan（训练/微调），编排走 Ray，另有 TorchRec。作者同时声明 TorchTPU 仍在活跃开发中，但已在部分 Google Cloud 大客户的生产推理与训练负载上使用。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

## 2026-10 PyTorch Conference：规模化验证数字与生态落点

2026 年 10 月圣何塞 PyTorch Conference 上谷歌正式开源 TorchTPU，最直观的变化是 Hugging Face Transformers v5 里把模型部署到 TPU 只需传 `device_map="tpu"` 一个参数。项目由谷歌与 Meta PyTorch 团队联合开发，vLLM、SGLang、Hugging Face、Ray、TorchTitan 同步完成接入，并发布 23 页技术报告（arXiv 2610.24161）；按 PyTorch/XLA 官方文档的说法，TorchTPU 公开后将取代 PyTorch/XLA。^[raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx.md]

实现层三个细节值得单独记：一是基于 PyTorch 预留给第三方硬件的 `PrivateUse1` 接口在分发器中注册为 C++ 后端，开发者拿到的是普通 PyTorch 张量，没有子类也没有包装器；二是 eager 三档（Strict 逐算子异步、Debug 同步、DeferAndFuse 攒块融合）共享同一套分层编译缓存，DeferAndFuse 比 Strict 快 50% 以上并降低峰值显存；三是 `torch.compile` 刻意不走 Inductor——Dynamo 捕获图后把算子映射到 StableHLO 交 XLA 生成 TPU 程序，理由之一是 Inductor 的成功很大程度建立在 Triton 之上，而 Triton 很难移植到 TPU 架构。分布式在 Process Group 层实现 TPU 集合通信，DDP / FSDP2 / DTensor 开箱即用，并支持 rank 0 多做日志这类非 SPMD 的「不对称」写法——这是只支持 SPMD 的 PyTorch/XLA 做不到的。^[raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx.md]

覆盖面与数值一致性是发布时的第一张答卷：谷歌持续在 TPU 上测试 Hugging Face 下载量最高的约 5000 个模型，报告撰写时共 4917 个、横跨 500 多个架构族，其中 98% 能跑通不崩溃，657 个代表性架构里 97% 的输出与 GPU 结果数值一致。^[raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx.md]

性能侧的三组数字口径不同，不能混读：

- **不做专门优化的基线**：30B 参数的 MuseGlimmer 只用 PyTorch 标准模块写、不调用任何模型专属内核，单颗 TPU v7 上 5120 序列的预填充达 2.57 万 token/s、MFU 59%；4 颗 TPU v7 训练 MFU 38%，把 TorchTitan 中为 GPU 写的同款实现稍改搬过来训练 MFU 36%。^[raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx.md]
- **规模**：Qwen3 235B MoE 在 64 颗 TPU v7 上吞吐约为谷歌自家 JAX 方案 MaxText 的 94%；从 1024 颗扩到 8192 颗（一整个 TPU v7 Pod）每 TensorCore 吞吐只降约 6%，MFU 稳定在 27%-29%，各规模损失曲线几乎重合；官方博客另称 Qwen 3 8B 训练 MFU 最高可达 52%。^[raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx.md]
- **推理与案例**：SemiAnalysis InferenceX 的 TPU v7 + vLLM-TorchTPU 在 8K 输入 / 1K 输出下每百万 token 成本 2.01 美分，比 9 月首次亮相的 3.6 美分低约四成；FLUX 训练从开箱即时模式到完整编译优化单步 11.49s → 2.78s；Meta 的 DLRM v2 在 32 颗 TPU v7x 上与谷歌高度优化的 JAX 基线完全持平。^[raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx.md]

产业侧的时间点也值得记：过去一年 Anthropic 签下最多 100 万颗 TPU 的使用协议，Meta 被曝与谷歌达成数十亿美元 TPU 租用合约——硬件已经卖得出去，软件成了最后一块短板，TorchTPU 是谷歌亲手补的这块板。^[raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx.md]

读法跟上文一致：94% 对 MaxText 才是「换框架不换硬件」的真实折扣；2.57 万 token/s 与 MFU 59% 是**无模型专属内核**基线下的后端成熟度，不是 TPU 对 GPU 的硬件对比；4917 个模型 / 97% 数值一致衡量的是算子覆盖广度，不构成收敛等价的保证。

## 深度分析

### 后端之争的本质：编译时机归谁

PyTorch/XLA 的问题不是"慢"，而是把编译时机放在了开发者写代码的位置：基于 Lazy Tensor，用户得在源码里划出可编译区，等于被要求用 JAX 的思维写 PyTorch。这与 PyTorch 的演进方向正面冲突——新能力都先在 eager 落地，再等编译器追。要求生态先为后端让路，生态没有理由答应。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

"原生后端"的准确含义因此是**把图的边界从用户手里收回到框架手里**：TorchTPU 在 dispatcher 层注册 `tpu` 后端、在 aten 层拦截算子，图捕获交给 torch.compile（Dynamo backend `tpu`），eager 仍是默认且可调试的模式。开发者面对的仍是普通 PyTorch——换个 device 就跑，编译是可选的加速层而非入场券。官方对此的表述是与其它后端共享同一个编译器前端 Dynamo：前端归社区，厂商只竞争 lowering 质量。

### 三种 eager：把"图多大"做成运行时旋钮

它没在"逐算子"与"整图编译"之间二选一，而是给了三个形态：Strict Eager（逐算子异步下发，等同 GPU eager）、Synchronous（同步执行，慢但可开边界检查）、DeferAndFuse（把连续 aten 攒块再编译，官方称比 Strict Eager 快 50% 以上，融合还压低 HBM 峰值）。代价是延迟算子要做活性分析判断哪些必须物化、带副作用的算子（集合通信）必须保序。这也解释了 DeferAndFuse 在 FLUX 上只值 1.15×：省的是下发与临时显存，不改算子数值效率。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

比性能更硬的是两处语义冲突。其一是内存模型：PyTorch 的 tensor 是指针加 stride，可共享字节、就地写彼此可见，view 只改元数据；StableHLO 是纯函数式的、值不可变。TorchTPU 用 view 分解与反演桥接，这不是"多支持一个算子"，而是维持一套跨语言内存契约，破了则拼接参数数组这类惯用法直接失效。其二是通信发起方式：JAX 走干净 SPMD，靠 sharding 注解让 XLA 自动插通信并重叠；而 PyTorch 生态里 rank 条件分支与 DTensor 不均匀切分会让各 rank 模块不一致，踩到 sync flag / buffer offset 不匹配。对策是默认把每个集合通信编成独立 skinny 模块保一致，代价是丧失重叠（官方给 30-40% 端到端损失），再用 `@spmd_safe` 或编译期握手合大图换回重叠——**非 SPMD 生态里做分布式编译，正确性默认优先，重叠要用户显式担保才买得到**。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

### 数字怎么读：三组基线，三种失效边界

- 训练侧 3.80× 是逐层加料的累积：11.49s →（DeferAndFuse）9.95 →（SimpleFSDP）6.52 →（SplashAttention）5.55 →（整步图 + 扫描）3.02 → 2.78。基线是"未调优的 eager"，度量的是**后端成熟度的落差**，不是 TPU 对 GPU 的硬件比。
- 唯一真正可比的是 Qwen3 MoE 235B 在 64 卡上对 JAX/MaxText：1828 对 1937 TPS，约 94%，这才是"换框架不换硬件"的真实折扣。
- 扩展性面，v6e-8 到 v6e-256 每样本步时 0.92s→1.00s、每芯片 TPS 走平，证明的是 XLA 把跨 DCN 的 all-reduce 压进了相邻层计算。
- 推理侧以性能/美元计（InferenceX 审计：2.01 美分/百万 token）。

边界：这些配置是每 TensorCore local batch 16、序列 4096 的宽 shape，正好喂饱 MXU。换成 memory-bound 的 decode、小 batch、shape 频繁变化，收益结构完全不同。稀疏负载更保守：DLRM v2 与 JAX 基线持平，是追平而非超越。

### 生态后果与还没解决的部分

地缘意义不在 TPU 而在**后端接口的所有权**：Google 有硬件缺开发者基数，Meta 有生态缺自研加速器，共做一个 backend 让"换加速器"从重写模型变成换 device，框架锁定与硬件锁定同时被削弱。这与 [[entities/pytorch-2-12-release-blog|PyTorch 2.12 发布]] 中 Dynamo 作为统一前端的路线同源。训练侧比 MFU 与扩展效率，推理侧比每美元 token 与 KV 布局——同一个后端、两套优化语汇，正合 [[concepts/inference-optimization|推理优化]] 的"优化层次分离"。

未解清单同样重要：自定义算子仍有 Pallas / Tokamax / Helion 三条路线；FP8/FP4 在途；bounded dynamism 未默认开启；一进程一设备使 RL colocation 无法开箱支持；profiling 工具仍在迁移；`@spmd_safe` 靠用户自判是否真 SPMD，用错会让系统变脆。适合迁移的是"标准架构 + 标准并行 + 标准栈"，深度定制内核或生产级确定性仍须逐案评估。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

## 实践启示

1. **先测"改一个 device 能不能跑通"，再谈性能。** 第一道门槛是算子覆盖与数值等价：eager 前向不崩、FX→StableHLO 发射成功、生成类困惑度相对差 ≤0.10，过了再进调优；引用加速比时把基线与 shape 一起写清。

2. **把编译时延当独立 SLI 监控。** TorchTPU 用三级缓存加 graph key 压它。自建后端应从第一天设计缓存键，并把首次 step 与稳态 step 分开上报。

3. **非 SPMD 代码里，先证一致性再索取重叠。** 跨 rank 图不一致会以死锁形式出现。默认保守（每通信独立模块），只在确无 rank 分支的区段用 `@spmd_safe` 换大图，并保留编译期握手作开关。

4. **sharding 按物理拓扑排序，别按变量名顺手写。** 通信最重的并行维（TP/EP）贴最连续的 ICI 维，DP 放跨 slice 的 DCN。

5. **别把"最高精度"当保险。** TPU 的 MXU 原生是 BF16 输入 + FP32 累加；`high`/`highest` 实际是 3 遍/6 遍分解，代价约 3× 到 6× 算力。只在确认收敛问题的那几层用线程局部精度域。

## 与既有基础设施条目的关系

这条路线对 [[entities/ai-infra-llm-efficient-inference-vllm|vLLM 推理基础设施]] 的意义在于**硬件后端的可替换性**：当 vLLM 能在 TorchTPU 栈上跑，推理优化的工作（batching、KV cache、调度）就可以与加速器解耦，这与 [[concepts/inference-optimization|推理优化]] 中"优化层次分离"的主张一致。对 [[entities/pytorch-2-12-release-blog|PyTorch 2.12 发布]] 这条线，TorchTPU 是 Dynamo 作为统一编译前端策略的延伸——文章明确说不同后端应共享同一编译器前端 Dynamo。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

需要保留的边界：这是厂商工程博客，性能数字绑定 FLUX/TorchTitan 特定配置，且对照基线是"eager 开箱即用"而非 JAX 上的同等调优实现，因此 3.80× 不能读成"TPU vs GPU"的结论；跨加速器迁移的真实成本仍在 sharding/shape 重写与算子覆盖度上。^[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026.md]

→ [[raw/articles/torchtpu-native-pytorch-tpu-backend-google-meta-2026|原文存档（PyTorch 官方博客）]] · [[raw/articles/torchtpu-pytorch-conference-2026-native-tpu-benchmarks-jqzx|原文存档（机器之心，发布数字复盘）]]
