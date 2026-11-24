---
title: "Cohere Megakernel Decode Serving Engine (North Mini Code)"
created: 2026-09-10
updated: 2026-11-21
type: entity
tags: [inference-optimization, kernel-engineering, megakernel, serving, vllm, cuda]
sources: [raw/articles/cohere-megakernel-serving-engine-north-mini-code]
confidence: 0.75
---

# Cohere Megakernel Decode Serving Engine

Cohere（2026-09-08，Xiaochun Tong/Conway Zhu/Donglu Wang）发布围绕 decode megakernel 构建的完整 serving engine，服务其 North Mini Code（30B 模型、每 token 3.3B 活跃参数、BF16 下每 decode step 需流式搬运 6.6GB 权重）。核心论点：低 batch decode 本质是 memory-bound，正确的问题不是 flops 而是 memory bandwidth 利用率。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md]

## 性能数据

H100（3.35 TB/s HBM）上 Speed-of-Light ≈ 470 tok/s；vLLM 服务该模型为 185 tok/s（仅 39% SoL）。Megakernel 在 batch=1 达 292 tok/s（62% SoL），比 vLLM 快 1.58×，端到端快 1.25-1.41×，优势跨 batch size 与 256K context 保持，无精度损失。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md]

## 架构设计

传统 serving 把一次 forward pass 拆成上百个小 kernel，GPU 在 launch 间隙空等。Megakernel = 单个 persistent kernel 跑完整 forward pass：每 SM 一个 threadblock 常驻整个 decode step；数据依赖不靠 kernel 边界编码，而用 global memory 中的显式计数器（task 完成时递增、需要输入时自旋等待）；block 从 host 准备的 task list 取活。这是首个支持 continuous batching、paged attention、ragged sequence length 与 OpenAI-compatible endpoint（含 tool calling）的 megakernel serving 系统。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md]

## 可迁移性

作者强调 megakernel 比名声上更容易写：单个 CUDA 文件，无编译器、无新编程范式——普通 tiled GEMM + 普通 paged attention 重构进单一 calling convention 即可，并附 kernel 移植 recipe。代码开源在 GitHub。该工程模式（memory-bound 分析 → task-list 调度 → persistent kernel）可迁移到其他 decode 场景，与 vLLM 等框架的 kernel-launch 模型形成对照。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md]

## 深度分析

### 加速来源：三种被 kernel 边界隐藏的带宽浪费

Megakernel 的收益不止省 launch 开销。文章把归因按影响排序：除每 step 只付一次的 launch/sync 成本外，更关键的是三点。其一 wave quantization——200 个 tile 对 132 个 SM 会取整成两波，kernel 越小浪费越大，tile 总数受矩阵形状约束、无法靠均匀切分修复；megakernel 没有可取整的边界，就绪 tile 直接落到空闲 SM。其二假依赖——kernel 边界是全 grid barrier，最慢的 SM 决定所有人的节奏，而实际依赖是局部的：某 KV group 的 O-proj 只需等该 group 的 attention 输出，某 expert 的 down-proj 只需等它自己的 up-proj。其三权重预取——权重与当前激活无关，可提前流式搬运；router 与 QKV 甚至在上一层 O-proj 尾部就开抢带宽。North Mini Code 的 parallel transformer layer（attention 与 MoE 共享同一归一化输入）让 backfill 更激进。kernel-per-op 引擎因此闲置了约 61% 的 HBM 带宽。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:54-58] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:59-64] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:70-77]

### 统一 ABI 是"可手写"的关键

文章最有信息量的工程结论：megakernel 不需要编译器或新范式。16 个 opcode 覆盖整个 decode step，全部遵守同一调用约定——固定的 12-warp threadblock 形状（controller/producer/storer 各 1 warp + 8 个 consumer warp）、32 个 int32 的 task descriptor（含等待与完成信号字段）、以及"完成即递增、需要即自旋"的全局内存计数器协议。所有 GEMM opcode 共享同一 warp-specialized pipeline body，加新 GEMM 任务只是填 per-op 细节的小改动而非新 kernel。代价集中在 barrier 记账：计数 barrier 只数到达次数、不校验是谁到达，一个 opcode 多到/少到一次会让 warp 漂移到不同任务、在很久之后才 deadlock，需要显式不变量与仔细测试。依赖机制本身是 O(1) 的——任务等一个计数而非特定上游，与 fan-in/fan-out 无关，但必须配 fence 才能避免跨 SM 的静默数据竞争。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:101-106] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:108-131] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:297-299] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:302-318]

### 调度是 kernel 之后的第二性能旋钮

调度采用"静态 wave 顺序 + 动态 work stealing"混合。8K 均匀路由消融只改 wave 顺序：调优序 291 tok/s，interleaved 282（-3%），attention-first 236（-19%）——所有顺序都产出正确结果，细粒度 barrier 只保证正确性，吞吐差异全部来自调度。反直觉的是依赖亲和放置（把任务与它的 producer 放同一 SM 以复用缓存）反而慢 1-2%，最终版本用朴素 round-robin。早期为 dense model 写的拓扑感知贪心与数百候选暴力搜索调度器曾带来约 10% 增益，但 MoE 动态 routing 使其不可迁移。动态侧只覆盖 host 无法预知的负载不均：ATTN_DRAIN/MOE_*_DRAIN claimer 任务从共享队列原子取活，claim 数控制并行度、队列反映实时请求，静态 schedule 因此保持稳定、无需随 batch 变化重建。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:325-343] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:383-386] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:396]

### 真实流量放大优势；server 边界决定收益上限

均匀路由下 megakernel 增益反而更小：BS=8 时真实 expert 分布 1.32×、均匀路由 1.14×。原因是真实请求倾向命中相同 expert，活跃 expert 集稀疏、MoE 总工作量减少、pipeline bubble 占比升高——而 bubble 恰是 megakernel 能回收的部分，均匀路由反而低估真实收益。另一面，静态 task list 把 tile 数与 barrier 索引烘焙进 schedule，Python 控制面必须先 park C++ decode 线程才能改 batch——prefill 暂停 decode、batch 上限 8 均为实现边界而非设计限制。端到端 1.25-1.41× 低于 decode-only 的 1.58×，正源于 prefill 仍用普通 PyTorch kernel 且与 decode 互斥、以及 serving 中 batch 大小持续波动。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:459-461] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:398-399] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:437-448] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:466]

## 实践启示

1. 低 batch decode 优化先问带宽利用率而非 flops：对每 token 3.3B 活跃参数的 BF16 模型，SoL 可直接由 (权重字节 + KV 字节)/HBM 带宽估算（North Mini Code 在 H100 上约 470 tok/s），用它和现有引擎实测吞吐的比值定位优化空间。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:36]
2. 迁移该模式时从已达标的 kernel 起步：megakernel 只消除 op 之间的 glue，不会让慢 kernel 变快；先让 GEMM/attention 单独比肩 cuBLAS/FA3，再套 ABI（8 consumer warp + ≤3 producer/storer warp，拆分 producer/consumer 函数以生效 per-warpgroup 寄存器上限）。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:302-308]
3. 权重预取是近乎免费的带宽：凡被 barrier 阻塞的任务仍可先行搬运字节——把 prefetch 放在跨 SM wait 之前即可；作者放弃 shared-memory paging 方案，因其簿记复杂、持续引入 bug 且开销超过收益，改用任务列表内的 multi-stage pipeline 复用。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:74-77] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:88] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:89-98]
4. MoE serving 基准测试要用真实 expert 分布：均匀路由会系统性低估 megakernel 类 bubble 回收型优化的收益（1.14× vs 1.32×），评估此类优化时应区分两种路由设定。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:459-461]
5. 静态调度值得调但不值得"聪明"：wave 顺序一个决定可差 19%，是 kernel 本体之外的另一性能旋钮；但依赖亲和放置与复杂调度器都输给 round-robin + 调优 wave 序——先调简单调度，复杂方案留给 Blackwell/NVFP4 等新场景。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:383-386] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:396] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:544]
6. 持久 kernel 的代价是宿主侧调度冻结：task list 预先烘焙意味着 batch 变更必须 park/resume；mixed prefill-decode 等动态场景需要调度器支持运行时重组，这是把该模式扩展到 prefill 与多卡（引入跨设备 collective 同步边界）的主要工程缺口。^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:398-399] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:545] ^[raw/articles/cohere-megakernel-serving-engine-north-mini-code.md:547]

## 关联

- 推理优化对比基线：[[entities/vllm|vLLM]]、[[entities/ai-infra-llm-efficient-inference-vllm|AI Infra LLM 高效推理]]
- 同源模型：[[entities/cohere-north-mini-code-agentic-coding-model|Cohere North Mini Code]]
- 异构对照：[[entities/tpu-megakernels-inferact-kimi-k3-700tps-2026|TPU Megakernels (InferAct/Kimi K3)]]
- 效率叙事对照：[[entities/magic-10x-more-efficient-pretraining|Magic 10x 高效预训练]]

→ [[raw/articles/cohere-megakernel-serving-engine-north-mini-code|原文存档]]
