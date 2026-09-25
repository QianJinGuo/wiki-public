---
title: "HySparse2：MiMo-V3 新架构预览（两级 KV 共享 + token 级稀疏选择）"
created: 2026-09-25
updated: 2026-09-25
type: entity
tags: [mimo, xiaomi, attention, sparse-attention, kvcache, yoco, long-context, agent, inference-optimization, model-architecture]
sources: [raw/articles/hysparse2xiaomi-mimo-v3-新架构预览]
confidence: 0.75
provenance_state: extracted
vxc: 56
---

# HySparse2：MiMo-V3 新架构预览（两级 KV 共享 + token 级稀疏选择）

小米大模型官方（2026-09-24）公开的 MiMo-V3 核心架构 HySparse2，面向**长程多轮 Agent** 场景的注意力架构升级：更少的 Prefill 计算、更小的 KV Cache、更精准的长上下文检索。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## 背景与问题定义

从 MiMo-V2 系列的 Hybrid SWA（Full Attention + Sliding Window Attention 混合）演进而来；第一代 HySparse 用少量 Full Attention 层提供 KV Cache 和重要位置选择结果，供后续 Sparse Attention 层复用，但 Prefill 仍需执行所有层，块级选择在多轮长距离检索上也有精度提升空间。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

Agent 长程多轮任务对注意力架构的三个要求：①高效读入（减少长输入 Prefill 计算）；②节省显存（降低 KV Cache 占用）；③精准检索（从长历史中找出相关证据、跨轮次信息整合）。一次简短的工具调用可能带回整页网页/文件/执行日志，每步都要处理新增输入。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## 两级 KV 共享（借鉴 YOCO）

HySparse2 将模型分成前后两部分：前半 Self-Decoder 采用 Full Attention + SWA 混合；后半 Cross-Decoder 采用 Full Attention + Sparse Attention 混合。KV 共享分两个层次：^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

- **KV Bridging（跨前后两部分）**：后半部分每个 Full Attention 层从前半部分对应 Full Attention 层的输入隐藏状态生成自己的 KV Cache——每个目标层保留独立 K/V 投影，同一份源隐藏状态可构建不同的 KV。后半部分 Full Attention 层的 KV 不必等输入逐层经过它们后才就绪，"KV 更早就绪"。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]
- **KV Reuse（同一 Hybrid Block 内）**：每个 Hybrid Block = 1 层 Full Attention + 随后多层 Sparse Attention。Full Attention 计算时按注意力分数选出重要位置，后续稀疏层直接复用其 KV Cache 和选择索引——保留 HySparse 核心设计（少量全注意力层提供全局信息与选择结果）。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## token 级稀疏选择 + 统一局部访问

HySparse2 在块级选择基础上引入 **token 级稀疏选择**，提升多轮长距离信息检索的精度；并将局部信息访问（SWA 滑窗）统一进同一机制。三个设计目标直指 Agent 场景的本质约束：历史随任务推进不断增长，而关键证据分散在早期轮次的工具返回中。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## 与已有架构的关系

- 相对 **YOCO**：借其"前半 Self-Decoder / 后半 Cross-Decoder"分层思想，但 Cross-Decoder 内部改为 Full+Sparse 混合而非全 Full；
- 相对第一代 **HySparse**：块级选择 → token 级选择；Prefill 不再全层执行；
- 与 DeepSeek V4.1 的 KV Cache 压缩路线（CSA2 三维联合利用）同属 2026 下半年"长程 Agent 注意力架构"竞赛，但各家切分维度不同（见 [[deepseek-v4-flash-pro-通往百万级上下文与万亿参数推理的新纪元-v2|DeepSeek V4 Flash Pro]]）。^[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览.md]

## 定位

这是继 [[mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo-V2.5 推理系统全链路优化（Hybrid SWA）]] 与 MiMo-V2.6 RL 训练之后，MiMo 系列公开的第三代架构路线信号：V2.5 解决"推理系统"效率，V3 的 HySparse2 解决"注意力架构"效率——从系统层下沉到模型结构层服务长程 Agent。

## 深度分析

### 两级 KV 共享的真正意义：把「缓存就绪」从串行依赖中解耦出来

传统 Transformer 中，第 N 层的 KV Cache 必须等输入逐层流经前 N-1 层才能构建——这是一种深度方向上的串行依赖。HySparse2 的 KV Bridging 打破的正是这个依赖：后半部分 Cross-Decoder 的 Full Attention 层保留独立 K/V 投影，但从前半 Self-Decoder 对应层的输入隐藏状态直接生成 KV。同一份源表示被「分叉」成多套不同的 KV，等于把 KV 构建从「逐层累积」改成「前半一次产出、后半并行投影」。这不仅是省计算，更是对推理流水线的结构性改造——Prefill 的关键路径被截短到前半程。

### 局部窗口合并是 Prefill 提前退出的前置条件，而非独立优化

表面看，把 SWA 滑窗合并进稀疏注意力（强制选最近 128 token，再从窗口外选 1,024 个全局 token）只是简化了稀疏层的实现。但原文点出了因果链的深层一环：独立 SWA 分支的 KV Cache 来自该层自己的隐藏状态，它的存在就要求后半部分继续执行前向计算。一旦局部与全局统一读取 Full Attention 层提供的共享 KV，后半部分就不再有「必须自己算一遍」的理由——这正是 Prefill 能在 Self-Decoder 结束后提前退出的逻辑前提。由此得到一个架构判断方法：评估某项优化是否关键，看它解除了哪条依赖链。

### 消融实验保留「强制局部窗口」，说明稀疏架构仍有精度取舍

HySparse2 并没有激进到完全抛弃局部性假设。原文明确提到消融实验显示强制局部窗口在多项长文任务上「保持竞争力，部分指标也存在取舍」——即纯全局稀疏选择并非处处占优，近距离上下文的稳定可见性仍是长文建模的兜底。这与 [[concepts/attention-mechanism|注意力机制]] 研究中「全局检索 + 局部精读」的混合范式一致：token 级选择解决「远处选得更细」，固定局部窗口保证「近处始终可见」，两者是互补而非替代关系。

### 放在 2026 下半年长程 Agent 架构竞赛里看：切分维度不同，收敛方向相同

DeepSeek V4.1 走 KV Cache 压缩（CSA2 三维联合利用），MiMo-V3 走 KV 共享 + token 级稀疏选择，路线细节不同，但收敛点一致：都把「长程多轮 Agent 的 KV 成本与检索精度」当作注意力架构的第一性约束。结合 [[entities/recent-developments-in-llm-architectures-kv-sharing-mhc-and-compressed-attention|LLM 架构近期进展（KV 共享与压缩注意力）]] 可以看到，KV 共享（YOCO 式分层）与 KV 压缩正成为两大主流方向，而 MiMo 的选择给出的答案是：与其压缩已有 KV，不如让 KV 更早就绪且被更少层重复构建。另一个值得注意的信号是，MiMo 把这项工作与 [[entities/mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo-V2.5 推理系统优化]]、[[entities/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026|MiMo-V2.6 RL 训练]] 刻画成一条「系统层 → 模型层」演进链：V2.5 优化推理系统，V3 直接改模型结构，优化空间从软件层下沉到架构层。

## 实践启示

1. **评估 Agent 场景的模型时，把 Prefill 计算量和 KV Cache 字节数作为一等指标。** HySparse2 在百万 token 下将 Prefill 计算量降到 Hybrid SWA 的 1/5、KV Cache 从 12GB 降到 2.7GB（相对一代 HySparse 分别为 1/3 与 6.7GB→2.7GB）。对多轮工具调用型应用，这两项直接决定每轮等待时间和可保留的历史长度，比单纯的 decode 吞吐更能反映实际体验。
2. **Prefill–Decode 分离部署的收益可以因架构设计而放大。** 49 层模型中 Prefill 只需执行前 25 层 Self-Decoder（含桥接 KV 投影，其中仅一层 Full Attention），Prefill 节点无需部署完整网络、权重存储接近减半。做推理集群容量规划时，应先确认所用模型是否支持类似的 Prefill 提前退出，而不是按全模型等比估算 Prefill 节点成本。
3. **稀疏选择粒度从块级到 token 级的升级，对多轮检索类 Agent 任务价值最大。** RULER-v2、MRCR-v2、GraphWalks 的提升说明：当关键证据分散在不同轮次、不同片段中时，块级选择会把注意力预算浪费在无关的相邻内容上。如果你的 Agent 需要跨轮次精准引用早期工具返回，token 级稀疏架构值得优先关注（参见 [[concepts/inference-optimization|推理优化]]）。
4. **改造注意力架构时，先画出 KV 构建的依赖图，再决定动哪里。** HySparse2 的两项关键改动——KV Bridging 与局部窗口合并——本质都是在解除「后半部分必须逐层执行才能拿到 KV」的串行依赖。这个分析方法可迁移：任何推理优化，先识别哪条依赖链决定了关键路径。
5. **长上下文成本与检索精度不是零和博弈，但需要局部性兜底。** HySparse2 用更少的 KV 和更短的 Prefill 同时换来了更高的 MRCR-v2/RULER-v2 分数（相对一代在各报告长度平均分别 +11.30 / +19.81 个百分点），证明「省」与「准」可以兼得；其代价是保留强制局部窗口这一保守设计。自行设计稀疏注意力时，不要轻易去掉局部可见性保障。
6. **架构竞争正从「参数规模」转向「Agent 工作负载的注意力经济学」。** MiMo 与 DeepSeek 等厂商 2026 年的动作共同指向：为长程 Agent 场景定制注意力结构（参见 [[entities/deepseek-v4-flash-pro-通往百万级上下文与万亿参数推理的新纪元-v2|DeepSeek V4 Flash Pro]]、[[concepts/context-window-economics|上下文窗口经济学]]）。选型时，可把「官方是否公开长上下文与 Agent 轨迹评测（如 AgentPPL、LongPPL）」作为衡量模型 Agent 就绪度的信号。

## 相关

- [[mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo-V2.5 推理系统全链路优化]] — 前代 Hybrid SWA 架构与推理系统
- [[concepts/attention-mechanism|注意力机制]] — 稀疏/滑动窗口注意力的基础概念
- [[entities/agent-memory-architecture|Agent 记忆架构]] — 长程 Agent 的另一个互补维度（KV 检索 vs 记忆层）

→ [[raw/articles/hysparse2xiaomi-mimo-v3-新架构预览|原文存档]]
