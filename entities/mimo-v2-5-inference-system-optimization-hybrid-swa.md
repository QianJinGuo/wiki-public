---
title: "MiMo-V2.5 推理系统全链路优化：Hybrid SWA + MoE + 多模态生产级落地"
type: entity
created: 2026-07-02
updated: 2026-08-14
tags: [mimo, xiaomi, inference-optimization, hybrid-swa, kvcache, speculative-decoding, moe, multimodal, production-engineering, sglang]
source:
author: MiMo Team
vxc: 63
sources:
  - raw/articles/mimo-v2-5-inference-system-optimization-hybrid-swa
  - raw/articles/mimo-v25-pro-ultraspeed-1000tps-xiaomi-2026
---

# MiMo-V2.5 推理系统全链路优化

## 摘要

小米 MiMo-V2.5 系列模型的推理系统全链路优化方案，围绕 Hybrid SWA + MoE + 多模态复合架构，系统性重构了 KVCache 管理、调度策略、Prefill/Decode 链路，实现 API 降价最高 99%。这是业内首篇全面覆盖该组合架构的大规模工程落地方案。^[raw/articles/mimo-v2-5-inference-system-optimization-hybrid-swa.md]

## 核心要点

1. **Hybrid SWA 架构**：70 层 Transformer 中 10 层 Full Attention + 60 层 Sliding Window Attention（窗口 128 token），KVCache 降至 1/7，Prefill 计算量降至 1/7
2. **KVCache 系统重构**：双池分治（Full KV Pool + SWA KV Pool环形缓冲区）、前缀缓存树重构、GCache 三级缓存（GPU→CPU→NVMe SSD）
3. **调度优化**：KVCache 亲和调度（L2 命中率+25%）、计算量感知优先调度（TTFT P90 -30%）、EP 缩减与三级长度分桶
4. **Decode 加速**：MTP 投机解码（前 128 token 加速 2.3×）、SWA 显存扩容（有效容量 +5 倍）
5. **多模态链路并行化**：Encoder 跨请求 Batch、GPU 图片预处理、视频多 chunk 并行（1 小时视频 156s→23s）

## 深度分析

### 1. Hybrid SWA 是生产级 LLM 推理的成本最优解之一

MiMo-V2.5 证明了一个关键假设：在 70 层模型中仅用 14% 的 Full Attention 层即可维持足够质量，同时将 KVCache 和 Prefill 计算量降至 1/7。这意味着 **Attention 的稀疏化不是质量牺牲而是架构设计选择**——Sliding Window Attention 对局部依赖建模能力与 Full Attention 接近，而全局依赖由少量 Full Attention 层承载。这一设计在长序列场景下优势更显著，与当前模型上下文窗口持续扩张的趋势高度契合。^[raw/articles/mimo-v2-5-inference-system-optimization-hybrid-swa.md]

### 2. 前缀缓存树重构是 SWA 工程化的核心挑战

SWA 模式下传统「token 序列相等 → KV 也相等」的缓存假设被打破，因为窗口滑动导致相同前缀在不同请求中的 KVCache 表示不同。MiMo 团队的解决方案——将匹配规则升级为「窗口安全长度」（尾部至少 W 个 token 仍有有效 slot），并让淘汰路径与请求生命周期绑定——是针对 SWA 特性的关键工程创新。线上命中率平均 93% 证明了这种方案的有效性。这一思路对任何采用 SWA 或其变体的推理系统都有普适参考价值。^[raw/articles/mimo-v2-5-inference-system-optimization-hybrid-swa.md]

### 3. GCache 三级缓存的零成本策略值得所有推理系统借鉴

GCache 利用 GPU 机器上已被分配的 CPU 内存和 NVMe SSD，作为 KVCache 的二级/三级存储，实现了「额外存储成本为零」。RDMA 通信实现 170 GB/s 读吞吐和 280μs 延迟，使三级缓存在性能上也可接受。这种「混部存算」的思路——在推理节点上复用已有的 CPU/NVMe 资源作为缓存层——比独立部署缓存集群更经济。与 [[agent-evaluation-systematic-guide-metrics-to-closed-loop|Agent 评测的分层评分引擎]] 的成本分层思路一致。^[raw/articles/mimo-v2-5-inference-system-optimization-hybrid-swa.md]

### 4. 计算量感知调度是对传统负载均衡的范式改进

传统负载均衡考虑的是请求数量均衡，而 MiMo 的「计算量感知优先调度」考虑的是真实计算 token 数量。这一差异在混合 Attention 架构下至关重要——不同请求的 Prefill 计算量可能相差数十倍（取决于输入长度和处理 attention 的层数）。优先处理计算量小的请求再辅以等待时间惩罚避免饥饿，这种「先易后难 + 公平补偿」的策略使 TTFT P90 降低 30%，同时不牺牲吞吐。^[raw/articles/mimo-v2-5-inference-system-optimization-hybrid-swa.md]

### 5. 多模态链路并行化的架构设计原则

MiMo 在多模态优化上的设计体现了一个重要原则——**将瓶颈操作从 CPU/IO 路径迁移至 GPU 计算路径**。图片预处理从 CPU 侧迁移至 GPU 消除大图瓶颈，视频解码多 chunk 多线程并行化，Encoder 支持跨请求 Batch。这背后是对「GPU 利用率高但 CPU/IO 成为瓶颈」现象的工程回应——当模型推理本身已高度优化时，数据预处理和传输的瓶颈就会凸显。随着多模态 Agent 场景增多（输入含图片、音频、视频），这一优化原则将变得越来越关键。^[raw/articles/mimo-v2-5-inference-system-optimization-hybrid-swa.md]

## 实践启示

1. **Attention 架构选择是推理成本的根本杠杆**：在模型设计阶段，对 Attention 比例的取舍直接影响部署成本。MiMo 的 Hybrid SWA 实践证明 10% 的全注意力层就足以维持质量。模型团队应在训练前就评估推理成本，而非先训后优化。

2. **前缀缓存在 SWA 架构下需要重新设计**：传统基于精确 token 匹配的缓存策略在 SWA 下失效。如果团队部署的模型使用 SWA 或变体，应参考 MiMo 的「窗口安全长度」方案重新设计缓存逻辑。

3. **推理优化要系统性考虑 KVCache、调度、Decode 三个维度**：单一维度的优化效果有限，MiMo 证明三维联动（KVCache 亲和调度 + 双池分治 + 投机解码）的协同效果远超各维度之和。建议推理系统团队以「全链路」视角而非「单点优化」视角制定优化计划。

4. **多模态 Agent 场景的瓶颈正在从模型推理转移到数据处理**：随着推理效率提升，图片/视频预处理和传输将逐渐成为延迟的主要贡献者。将预处理迁移到 GPU 并支持跨请求 Batch 是当前最有效的缓解策略。

5. **开源回馈是检验工程质量的试金石**：MiMo 将部分优化以 PR 形式回馈 SGLang 开源社区。回馈开源的过程强制团队将临时方案标准化为通用设计，本身就是工程自检。

## 相关实体

- [[areal-2-agentic-rl-online-learning-self-evolving|AReaL 2.0 在线 RL 系统]] — 推理基础设施的另一个维度（RL 训练方）
- [[agent-evaluation-systematic-guide-metrics-to-closed-loop|Agent 评测体系化指南]] — 分层效率设计的评估维度
- [[insight-agent-trustworthy-reasoning-guandata|洞察 Agent 可信推理链路]] — 企业级推理应用场景
- [[codex-agentsmd-project-instructions-rookie|Codex Agent 项目配置]] — Agent 推理客户端实践

## 第 2 来源 — MiMo-V2.5-Pro-UltraSpeed：将 1T 参数模型推向 1000 TPS（2026-08-14 MERGE）

> v×c=56 (v=7 c=8 s=4) | 小米大模型 (2026-06-09) | MiMo × TileRT 联合发布的 UltraSpeed 模式技术详解，与第 1 来源（推理系统全链路优化）同品牌同系列演进

**核心增量：在 1T 参数旗舰模型上首次突破 1000 tokens/s 输出速度**，通过 FP4 QAT + DFlash 投机解码 + TileRT 超低延迟推理系统三层联动实现。^[raw/articles/mimo-v25-pro-ultraspeed-1000tps-xiaomi-2026.md]

**互补角度 5 条**：

1. **FP4 选择性量化（MXFP4 QAT）**：1T 尺度下 FP8/INT8 带来恐怖显存占用和内存带宽压力，但「一刀切」FP4 会在复杂推理、逻辑代码上精度退化。针对 MoE 架构特性（Expert 占参数绝大部分且对量化精度容忍度最高），只对 MoE Expert 做 FP4 QAT 量化，其他模块保留原精度——大幅缩减模型体积、榨干硬件带宽的同时整体能力基本持平。^[raw/articles/mimo-v25-pro-ultraspeed-1000tps-xiaomi-2026.md]
2. **DFlash 投机解码**：与第 1 来源的 MTP 投机解码（前 128 token 加速 2.3×）互补的另一种解码加速路径，进一步压缩单 token 生成延迟。^[raw/articles/mimo-v25-pro-ultraspeed-1000tps-xiaomi-2026.md]
3. **TileRT 常驻内核引擎（Persistent Engine Kernel）**：1000 tokens/s 高频状态下单算子生命周期压缩至微秒级，传统推理系统「算子边界」成为核心瓶颈（每次算子启动、硬件同步、全局内存往返都在微秒尺度打断执行流）。TileRT 摒弃逐算子启动模式，让整个计算流水线常驻 GPU 内部持续流转，获得全链路持续预取能力——当前 Tile 仍在 Tensor Core 计算时，后续数据已沿存储架构提前流动，实现数据搬运与计算的极致重叠。这是「消灭算子边界」的范式级执行模型变革，可迁移到任何超低延迟推理场景。^[raw/articles/mimo-v25-pro-ultraspeed-1000tps-xiaomi-2026.md]
4. **算法重构（千亿/万亿模型带宽卸荷）与推理系统（TileRT）的分工配合**：MiMo 算法侧重构为模型卸下带宽枷锁，TileRT 推理系统侧将通用 GPU 物理潜能压榨到微秒级极限——与第 1 来源的「KVCache 亲和调度 + 双池分治 + 投机解码三维联动」构成同一优化哲学的延续。^[raw/articles/mimo-v25-pro-ultraspeed-1000tps-xiaomi-2026.md]
5. **发布形态印证生产化路径**：从 06-09 发布 UltraSpeed 模式到第 1 来源的 API 降价 99%，MiMo-V2.5 系列的推理优化从「技术能力展示」走向「成本结构重构」，验证了推理优化最终要落回商业价值。^[raw/articles/mimo-v25-pro-ultraspeed-1000tps-xiaomi-2026.md]

→ [[raw/articles/mimo-v25-pro-ultraspeed-1000tps-xiaomi-2026|第 2 来源原文存档]]

→ [[raw/articles/mimo-v2-5-inference-system-optimization-hybrid-swa|原文存档]]
