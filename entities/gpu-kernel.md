---
title: "gpu kernel"
created: 2026-08-01
updated: 2026-09-07
type: entity
tags: ['raw', 'article']
sources: [raw/articles/gpu-kernel]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/gpu-kernel.md|原文存档]]

sha256: 76b09750ef9b2c7a5beadae8e6d99e219066a794c91a2497fa28161585315668 ^[raw/articles/gpu-kernel.md]

## 摘要

这是一篇深度技术长文，以一个在 RTX 4090 上把两个百万长度向量相加的极简 CUDA 程序为例，完整追踪一次 kernel 从源码到硬件的执行全链路 ^[raw/articles/gpu-kernel.md]。编译阶段，nvcc 驱动 cicc 把设备代码编译成与设备无关的 PTX 虚拟 ISA，再由 ptxas 编译成特定架构的 SASS，fatbinary 把两者打包进 ELF 格式的可执行文件（PTX 作为前向兼容的 JIT 回退） ^[raw/articles/gpu-kernel.md]。启动阶段，主机端把参数打包进 QMD（Queue Meta Data）描述符，通过 pushbuffer/GPFIFO 通道写入 GPU 方法流，最后以一次 MMIO 写操作敲响 doorbell 寄存器触发计算 ^[raw/articles/gpu-kernel.md]。执行阶段，compute work distributor 把 4096 个 block 分配到 128 个 SM 上，每个 SM 借助 ptxas 写入指令的静态 stall 计数、yield 提示和 6 个 scoreboard barrier 实现接近零硬件开销的延迟隐藏，最终 DRAM 以约五分之四的峰值带宽在 10.78 微秒内完成计算 ^[raw/articles/gpu-kernel.md]。文中还给出了用 LD_PRELOAD shim 拦截 mmap、解析 pushbuffer 命令流、解码 ioctl 等窥探闭源驱动内部机制的方法 ^[raw/articles/gpu-kernel.md]。

## 关键要点

- 编译流水线：cicc（LLVM 系）生成 PTX（无限多类型化虚拟寄存器的设备无关 ISA）→ ptxas 生成 SASS 并把十余个虚拟寄存器折叠到 7 个真实寄存器、将 mul.wide+add 融合为 IMAD.WIDE → fatbinary 将 cubin 与 PTX 打包为 fatbin 嵌入 .nv_fatbin 段
- 启动机制：kernel 参数存放在 constant bank 0（偏移 0x160/0x168/0x170 为三个指针，0x178 为 n），由 QMD 携带；现代 GPU（Turing 起）不再侦听 GP_PUT 游标，必须由驱动向 doorbell 寄存器写 work-submit token
- 资源约束决定驻留 block 数：AD102 每个 SM 上限 1536 线程、65536 个 32 位寄存器、100 KB shared memory；本例每线程 16 寄存器、每 block 256 线程，线程容量是更紧的瓶颈，每个 SM 驻留 6 个 block（48 warp）
- warp 调度靠编译期控制码：每条 128 位 SASS 指令第二字携带 21 位控制字段（4 位 stall 计数、6 位等待掩码、yield 位、reuse 位），两个 LDG.E 设置同一 scoreboard barrier B2，FADD 等待 B2 清除后发射
- ncu 实测：warps active 82.77%、issue active 仅 5.17%、DRAM 吞吐 79.65%、总时长 10.78 微秒；输出数组 c 因留在 72 MB L2 中而从未写回 DRAM，回读直接由 L2 服务

## 来源

- 原文：[[raw/articles/gpu-kernel.md|gpu kernel]]
