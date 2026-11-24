---
title: "TPU Megakernels：Inferact 在 Kimi K3 上跑出 700 TPS"
created: 2026-09-26
updated: 2026-11-13
type: entity
tags: [inference-optimization, tpu, megakernel, kimi-k3, speculative-decoding, llm-inference, vmem]
sources: [raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026]
confidence: 0.75
provenance_state: extracted
---

# TPU Megakernels：Inferact 在 Kimi K3 上跑出 700 TPS

Inferact 开源 tpu-megakernels（TPU v7 Ironwood）：Kimi K3 speculative decoding 下 700+ tokens/s（GB200 基线 452）；无 spec decoding 时 batch 1-8 解码吞吐 1.4-2× GB200。核心设计：TPU 大容量片上 VMEM + 顺序编程模型 + 跨层异步权重预取，把端到端解码推向 HBM 带宽理论峰值。 ^[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026.md]

## 深度分析

### Megakernel 消除多 kernel 带宽气泡

传统解码路径每个 step 会依次分发数百个独立 kernel，每个 kernel 大致经历"从 HBM 加载权重/激活 → 计算 → 写回"三段，只有加载阶段真正吃满 HBM 带宽，计算与写回阶段都是带宽空闲的"气泡"。CUDA Graphs、stream 并发、PDL（programmatic dependent launch）等 GPU 机制只能部分恢复重叠，kernel 边界依然存在。Megakernel 直接取消边界：整个解码 step 融进单个程序后，权重加载不再绑定于使用它的 kernel，只要片上存储允许就可以尽早发起，真正的约束变成了缓冲区生命周期与容量本身。目标很纯粹——让 HBM 接口在整个解码 step 中持续忙碌。^[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026.md]

### VMEM 64 MiB vs GPU TMEM 的架构差异

TPU v7 Ironwood 与 GB200 顶层规格相当（ICI 1,200 GB/s vs NVLink 5 1,800 GB/s），但片上存储形态差异巨大：TPU 每个 TensorCore 拥有 64 MiB VMEM，HBM 数据直达一大块 SRAM 池，VREG 直接从 VMEM 取数，且 VMEM 中数据的存亡完全由程序显式管理——不覆盖就一直在。GPU 的 TMEM 总量约 38 MiB，却要摊给 152 个 SM（每 SM 仅 256 KB），加上硬件管理的 L1/L2 缓存层级，几乎没有提前驻留权重层级的余地。这解释了为什么"跨层权重预取"在 TPU 上是自然设计、在 GPU 上却寸步难行：解码速度受限于内存带宽，而能否提前把权重搬进最快存储，取决于最大的一块可编程 SRAM 有多大。^[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026.md]

### 无网格单程序启动与 run_scoped 生命周期控制

虽然 Pallas 也能写成 GPU 风格的网格 kernel，但 TPU 本质上是单线程顺序执行模型——即便启动网格，程序仍是一个接一个跑。作者选择彻底拥抱单线程：megakernel 以无网格单程序方式启动，换来更好的可读性和细粒度控制。VMEM 管理上，run_scoped 原语提供了细粒度生命周期控制：激活等数据全程驻留，MLA 的 KV cache 等短命数据则用自包含的"迷你 kernel"声明所需形状与大小，作用域结束即释放。程序员无需手工追踪任意时刻哪些 VMEM 字节区间可安全读写，只需在 64 MiB 预取预算内平衡激进预取与容量约束。^[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026.md]

### 跨层异步权重预取与编译红利

异步 DMA 是 megakernel 得以成立的关键：make_async_copy(...).start() 指定预取发起时机，.wait() 指定阻塞等待点。Kimi K3 的实现把 92 层 MoE 全部合并进一次 Pallas 调用，按层索引在 KDA 与 MLA 之间条件切换，层循环就在 kernel 内部，因此异步拷贝可以天然跨越层边界——第 N 层还在跑 MoE 时，第 N+1 层的注意力投影权重已在搬运途中。在 JAX 中管理异步依赖一向别扭（XLA 会把操作链当整体调度、随意重排或回收缓冲），但在单 kernel 内部不存在这个问题：拷贝的 start/wait 不会跨出 XLA 可自由调度的边界。另一个意外收获是编译速度：作者手工一次性付清了 XLA 每次编译都要重新发掘的排序与重叠成本，数百上千个 XLA op 的大模型常需 30+ 分钟编译，而整个 megakernel 从零编译不到 90 秒，大幅加快了开发迭代周期。^[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026.md]

### 投机解码接受长度经济学与解码型服务经济性

投机解码的收益是接受长度的直接函数：DSpark 投机解码每次 kernel 启动评估 1 个 anchor + 7 个候选续写，草稿模型平均每步接受 3-6 个 token，解码步长约 8.5 ms，由此对单用户跑出 700+ TPS。经济含义在于：解码是内存带宽受限的，一次 step 搬运权重的成本与产出 1 个还是 6 个 token 几乎无关，所以接受长度越长，单位 token 的带宽成本被摊得越薄——megakernel 把 step 本身压到接近带宽峰值后，投机解码的杠杆被进一步放大。基线对比同样说明这一点：无投机解码时 batch 1-8 吞吐已达 GB200 的 1.4-2×（batch 1 近乎翻倍），加上接受长度 6 的投机解码即从 452 TPS 拉开到 700+。对小批量、解码主导的服务场景（如单用户 agentic 应用），TPU megakernel + 投机解码的组合正在重新定义单请求时延的性价比上限；而精度验证（GPQA-Diamond 0.944、GSM8K 0.972）表明激进优化没有牺牲正确性。^[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026.md]

## 实践启示

1. **解码瓶颈优化先看片上可编程 SRAM 的容量与控制权**：TPU VMEM 64 MiB/核且生命周期完全由程序管理，GPU TMEM 38 MiB 摊到 152 个 SM 几乎无法预取权重——选硬件时，解码型负载应优先评估"最大可显式管理的片上存储"这一维度，而非只比带宽数字。
2. **把跨层调度从编译器手里拿回来**：XLA 这类自动调度未必能发现最优的跨层流水线（存储复用、传输时机、同步需联合决策）；当融合策略受制于编译器时，用 Pallas 这类内核语言直接表达调度决策、只把指令级调度留给编译器，是一条已被验证的路径。
3. **异步拷贝放在单 kernel 内部做，不要跨 kernel 拼**：在单 kernel 内 start/wait 的异步 DMA 不会触碰 XLA 的重排/回收边界，JAX 异步依赖管理的老问题随之消失——这是 megakernel 相对多 kernel + stream 方案在工程可控性上的实质优势。
4. **投机解码 + 接近带宽峰值的 step 是小批量服务最优解**：解码步成本由权重搬运主导，与产出 token 数无关，接受长度 3-6 即可把单用户 TPS 从 452 推到 700+；部署时重点调优草稿模型使平均接受长度最大化。
5. **批大小上升会暴露新的瓶颈类型**：batch 1→8 就让残差流向量算术（attention residual、RMSNorm）成为瓶颈，需要张量并行叠加残差流序列并行来拆分向量运算——扩展并发时别只盯着矩阵乘与专家并行，要对向量算术和跨设备通信单独做并行化设计。
6. **拓扑选择是 TPU 特有的额外搜索维度**：NVLink 域内每张卡等距，只需决定用几张卡；TPU 还要选拓扑（本项目绑定 2×2×4、16 卡），换拓扑意味着重写集合通信——规划 TPU 集群时把拓扑形状当作与并行策略同等重要的一等决策。

## 相关链接

- [[concepts/inference-optimization|推理优化]]
- [[entities/cohere-megakernel-serving-engine-north-mini-code|Cohere North megakernel serving]]

→ [[raw/articles/tpu-megakernels-inferact-kimi-k3-700tps-2026|原文存档]]
