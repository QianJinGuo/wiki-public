---
title: "gpu kernel"
created: 2026-08-01
updated: 2026-11-21
type: entity
tags: ['raw', 'article']
sources: [raw/articles/gpu-kernel]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# gpu kernel

## 摘要

这篇文章以"两个百万长度向量相加"这一极简 CUDA 程序为线索，完整追踪了一次 kernel 从源码到硬件再返回结果的全链路：`nvcc` 多级编译产出 PTX/SASS，驱动通过 pushbuffer/GPFIFO/doorbell 把启动描述符 QMD 交给 GPU，compute work distributor 把 4096 个 block 铺满 128 个 SM，最终以约五分之四的 DRAM 峰值带宽在 10.78 微秒内完成 ^[raw/articles/gpu-kernel.md:53-60]。文章的独特价值在于把通常被黑盒化的三层——编译器（ptxas 的控制码）、闭源用户态驱动（libcuda）、硬件调度器（scoreboard barrier）——全部拆开给读者看，并展示了用 LD_PRELOAD shim、ioctl 解码等手段窥探闭源驱动的可复现方法 ^[raw/articles/gpu-kernel.md:362-366]。

## 核心要点

- 编译流水线是多编译器接力：cicc（LLVM 系）生成设备无关的 PTX 虚拟 ISA → ptxas 生成特定架构的 SASS → fatbinary 把两者打包成 fatbin 嵌入宿主可执行文件的 `.nv_fatbin` 段，PTX 作为前向兼容的 JIT 回退保留（默认压缩） ^[raw/articles/gpu-kernel.md:64-79,137-159]
- kernel 参数通过 constant bank 0 广播给所有线程：三个指针固定在偏移 0x160/0x168/0x170、n 在 0x178，host stub 打包参数的偏移（0/8/16/24）与 SASS 读取的 bank 偏移一一对应 ^[raw/articles/gpu-kernel.md:132-135,167-180]
- GPU 不接受函数调用：驱动把 QMD（含 grid/block 维度、寄存器需求、SASS 入口、参数 bank 地址）作为 methods 写入 pushbuffer，GPFIFO 环形队列维护 GP_GET/GP_PUT 游标，Turing 起的 GPU 不再侦听游标，必须以一次 MMIO 写 doorbell 寄存器触发 ^[raw/articles/gpu-kernel.md:201-225]
- 驻留 block 数由最紧的资源约束决定：AD102 每 SM 上限 1536 线程/65536 寄存器/100KB shared memory，本例每线程 16 寄存器时寄存器可容纳 16 个 block，但线程容量只允许 6 个 block（48 warp） ^[raw/articles/gpu-kernel.md:241-258]
- warp 调度靠编译期静态控制码而非乱序硬件：每条 128 位 SASS 指令的第二字携带 21 位控制字段（4 位 stall 计数、6 位等待掩码、yield 位、reuse 位），两个 LDG.E 设置同一 scoreboard barrier B2，FADD 等待 B2 清除后才成为 eligible ^[raw/articles/gpu-kernel.md:264-307]
- 内存访问按 warp 合并（coalescing）：32 个线程的连续 4 字节访问被合并为 4 个 32 字节 sector 请求，逐级经过 L1 → 72MB L2 → GDDR6X；本例 12 字节传输只换一次浮点加法，是典型的算术强度瓶颈 ^[raw/articles/gpu-kernel.md:309-345]
- ncu 实测：warps active 82.77%、issue active 仅 5.17%、DRAM 吞吐 79.65%、总时长 10.78 微秒；输出数组 c 全程留在 L2，从未写回 DRAM，回读由 L2 直接服务 ^[raw/articles/gpu-kernel.md:317-345,347-355]
- 一次 launch 的 ioctl 量级惊人：strace 记录到 948 次 ioctl（几乎全是一次性 setup），稳定启动循环远少于此，揭示了驱动初始化与稳态开销的差异 ^[raw/articles/gpu-kernel.md:505-516]

## 深度分析

### 编译器的两级降级：PTX 的"冗长"是有意的

PTX 里形成 `&a[i]` 一个地址竟要三条指令（cvta.to.global + mul.wide + add.s64），看似低效，实则是设备无关性的代价与设计：`cvta.to.global` 把"通用指针"（可能是 global/shared/local 任意窗口）断言为 global 窗口，从而让后端能用更廉价的 `ld.global`；而 ptxas 把十余个虚拟寄存器折叠到 7 个真实寄存器（ncu 报告每线程 16 个含 ABI 保留），并将 mul.wide+add 融合为单条 IMAD.WIDE ^[raw/articles/gpu-kernel.md:100-130]。这解释了 CUDA 编译体系的一个关键分工：PTX 层做语义正确性，架构特定的优化全部下推到 ptxas——因此同一份 PTX 在不同代 GPU 上能生成完全不同的 SASS，fatbin 里附带压缩 PTX 的唯一目的是在未覆盖的架构上 JIT ^[raw/articles/gpu-kernel.md:137-159]。对写 kernel 的人而言，这意味着 PTX 级别的手工优化往往徒劳，真正的杠杆在 SASS 层。

### 启动即数据流编程：QMD 是 CPU 与 GPU 的唯一契约

GPU 不像 CPU 那样有调用栈和入口跳转——它只是消费驱动写入宿主内存的命令流 ^[raw/articles/gpu-kernel.md:199-213]。文章把启动机制还原为三个数据结构：pushbuffer（methods 的存放地）、GPFIFO（指向 pushbuffer span 的环形指针队列）、USERD（GP_GET/GP_PUT 双游标）。QMD 作为 256 字节的启动描述符内联流入 pushbuffer（dump 显示 66 词 burst，word 12/18 恰是 4096 和 256），它把"代码在哪、参数在哪、怎么并行、完成如何通知"四件事一次性编码 ^[raw/articles/gpu-kernel.md:217-225,471-479]。值得注意的演化细节：老 GPU 会 snoop USERD 游标，Turing 之后主机引擎不再轮询，驱动必须对映射到进程地址空间的 doorbell 寄存器写 work-submit token——一次 MMIO 写就是整个启动的扳机 ^[raw/articles/gpu-kernel.md:221-227]。这也解释了为什么 kernel launch 是异步的：cuLaunchKernel 在 doorbell 响起的瞬间就返回了。

### 延迟隐藏的极简主义：把调度从硅片挪进指令流

文章对 GPU 与 CPU 调度哲学的对比是全文最深刻的洞察：CPU 用 reorder buffer、寄存器重命名等大量硅片在单线程内动态发现并行性，GPU 则认定"并行本来就够多"，把一切可预测的时序交给编译器——每条 SASS 指令第二字里 ptxas 写入的 21 位控制字段，包含固定延迟指令的精确 stall 周期数、可变延迟操作（LDG/MUFU）依赖的六个 scoreboard barrier 编号、以及调度优先级让渡的 yield 位 ^[raw/articles/gpu-kernel.md:266-278,520-530]。本例中两个 LDG.E 复用同一 barrier B2，FADD 挂在 B2 上直到两个加载都返回；而 FADD→STG 之间因为浮点加法延迟固定，只用 stall=5 而无 barrier ^[raw/articles/gpu-kernel.md:293-307]。这套机制的硬件成本几乎为零——每个周期调度器只需读一个六位掩码和小计数器——代价是控制码布局只能靠 Citadel 微基准论文和 maxas 社区逆向工程得知，且 nvdisasm 默认隐藏它们。理解控制码是手工调优 SASS（如 maxas 式 GEMM）的前提。

### 算术强度决定一切：vadd 为什么"只能"跑这么快

ncu 的三个数字共同讲了一个故事：warps active 82.77% 说明占用率健康，issue active 5.17% 说明计算单元几乎全程闲置，DRAM 79.65% 说明总线才是瓶颈 ^[raw/articles/gpu-kernel.md:317-345]。根因是算术强度——每 12 字节传输（两次 4 字节加载 + 一次 4 字节存储）只换一次 FADD，10.78 微秒的时长本质上就是 DRAM 喂 8.4 MB 读数据的时间（约 780 GB/s，五分之四峰值） ^[raw/articles/gpu-kernel.md:343-345]。更有趣的是写入路径的"意外优化"：4 MB 输出恰好装进 72 MB L2，STG.E 从不落到 DRAM，随后的 D2H 拷贝直接由脏 L2 服务，省掉了整个 DRAM 回写往返 ^[raw/articles/gpu-kernel.md:315,345,353]。这提醒我们：性能模型必须考虑 L2 容量与工作集的相对大小——同样的 kernel 在输出超过 L2 时会多出一段写回时间。这类访存主导的 kernel 优化空间不在指令层面（如 [[entities/deepseek-v4-triton-fp4-optimization|Triton FP4 优化]] 与 [[entities/triton-l2缓存命中优化矩阵乘法fp16int8详解及性能测试|L2 缓存命中率优化]] 所示），而在数据布局与融合策略上——把多个访存密集操作融合进一个 kernel 以摊薄传输成本，正是 [[entities/fable-5-cuda-super-kernel-187x-speedup-2026|fable-5 CUDA 超级内核]] 一类工作的核心思路。

## 实践启示

1. **调优前先算 occupancy 的约束方程**：驻留 block 数 = min(寄存器约束, 线程约束, shared memory 约束)。本例寄存器允许 16 block 但线程上限只给 6——先找出最紧的瓶颈维度再决定是否值得减寄存器/改 block size ^[raw/articles/gpu-kernel.md:255-258]。
2. **判断 kernel 是访存型还是计算型，用三个 ncu 指标足矣**：issue active 高 = 计算瓶颈，DRAM throughput 高 = 访存瓶颈，warps active 低 = 占用率不足。vadd 的 5.17%/79.65% 组合是教科书式的 memory-bound。
3. **PTX 层的优化是幻觉，SASS 才是真的**：想控制指令发射顺序，要读 `cuobjdump -sass` 的控制码（stall/barrier/yield），而非盯着 PTX 源码；nvdisasm 会隐藏控制位，需看原始 hex ^[raw/articles/gpu-kernel.md:274-280]。
4. **C 大小的执行细节影响性能模型**：coalescing 让 warp 的 32 次访问合并为 4 个 sector，非连续访问会成倍放大 DRAM 事务；而输出恰好落在 L2 的 kernel 会"免费"省去写回——benchmark 时要意识到这可能是工作集的巧合而非设计 ^[raw/articles/gpu-kernel.md:309-315,345]。
5. **闭源驱动并非不可窥探**：LD_PRELOAD 拦 mmap 抓 pushbuffer、strace 解码 ioctl（magic 0x46、NV_ESC_RM_ALLOC/CONTROL）、小型 GPU kernel 读设备内存，三者组合足以还原启动全流程——这是研究任何闭源用户态驱动可复用的方法论 ^[raw/articles/gpu-kernel.md:368-371,505-516,482-501]。
6. **异步性是语义而非实现细节**：launch 在 doorbell 响起即返回，完成通知靠 QMD 携带的 completion semaphore；默认流上的 cudaMemcpy 会排队等待，用 pinned memory + cudaMemcpyAsync 才能让 host 真正跑在 GPU 前面 ^[raw/articles/gpu-kernel.md:227,347-355]。

→ [[raw/articles/gpu-kernel|原文存档]]
