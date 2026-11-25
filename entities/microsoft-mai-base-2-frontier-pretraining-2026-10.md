---
title: "MAI-Base-2：微软 AI 前沿预训练（80B/2.8T MoE、scaling ladder 与 Muon 规模化）"
created: 2026-10-20
updated: 2026-11-26
type: entity
tags: [pretraining, moe, scaling-laws, muon, microsoft, training-infrastructure, llm]
sources: [raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10]
confidence: 0.9
provenance_state: extracted
---

# MAI-Base-2：微软 AI 前沿预训练

微软 AI（MAI）在 2026-10-15 的 Lab Notes 中披露了 MAI-Base-2 的完整预训练叙事：一个 80B 激活 / 2.8T 总参数的 MoE foundation model，用 16K 张 GB200 训练 38 天（2026 年 8–9 月），平均 MFU 21%、goodput 89%，并在 held-out BPB 上优于全部被测 open-weight 模型，相对最强开源 base model DeepSeek-V4-Pro-Base 有 4–12x 的预训练 FLOPs 效率优势。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

这是少见的、由前沿实验室第一方给出的"预训练配方"级细节：不只是模型规格，还包括超参 scaling law 的解析式、MoE 实现层的 MFU 归因、Muon 在 16K GPU 上的通信优化，以及把 activation gradient clipping 当作数据质量探针的用法。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

## 模型规格（MAI-Base-1 → MAI-Base-2）

| 项目 | MAI-Base-1 | MAI-Base-2 |
|---|---|---|
| 激活 / 总参数 | 35B / 1T | 80B / 2.8T |
| 层数 | 78 | 102 |
| hidden dim | 6656 | 8704 |
| MoE 布局 | dense FFN 与 MoE 交替 | 每层都是 MoE |
| MoE 路由 / expert 数 | 0 shared + 8，512 routed | 1 shared + 7，512 routed |
| Attention（KV / Q heads） | 8/80 global（每 6 层）+ 8/80 local | 2/96 global（每 6 层）+ 16/96 local |
| Embedding | tied | untied |
| 训练 horizon | 30T tokens | 50T tokens |
| Optimizer | AdamW | Muon（非 embedding 2D 参数）+ AdamW |
| FP8 方案 | tensor-wise + delayed scaling E4M3/E5M2 | MXFP8 E4M3 |
| 梯度裁剪 | 全局 weight grad clip | 全局 weight grad + activation grad clip |
| 视觉 | text only | native VLM |

架构上有两个方向性选择值得记录：**每层都上 MoE**（而非交替 dense/MoE），以及 **attention 的 KV head 进一步压缩到 2 个 global**、只保留 16 个 local head —— 前者靠实现效率换回，后者是长上下文下 KV cache 成本的直接让利。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

## Scaling 科学：把超参选择从"启发式"变成"解析式"

MAI 把 MAI-Base-1 → MAI-Base-2 的成功归因于三件事：scaling ladders（考虑重复训练数据）、model-infrastructure co-design、数据规模扩张。其中最有复用价值的是超参 scaling law 的做法：对若干 (model size, TPP) 组合沿搜索网格训练，在预训练评测上拟合 2D 二次函数，取解析最小值得到 $(LR^*, LR^* \times WD^*)$。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

文章给出的拟合结果可直接作为量级参照：

- $LR^* \times WD^* = 17.04 / \text{steps}$
- $\log LR^* = -1.51 + 0.175\log \text{TPP} - 0.573\log \text{steps}$
- $\mathrm{Dropout} = \operatorname{sigmoid}\big(0.526(\log \text{ActiveParams} - 27.2)\big)$

这些 law 先在小规模实验估计，再在中等规模验证，稳定优于旧启发式后用于定义 MAI-Base-2 的最终超参。对做大规模训练的团队，这条"小规模拟合 → 中规模验证 → 全量使用"的流程比公式本身更重要。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

## MoE 的 MFU 归因：24% → 35% 才换来"每层 MoE"

MAI-Base-1 时代，虽然实验显示"每层 MoE"相对交替布局有一致的 EG FLOPs 优势，但 MoE 实现的低效使"每隔一层 MoE"在 EG **时间**上更优。MAI-Base-2 定位了两个低效点：dynamic routing 造成内存碎片、并为极端的 expert 负载不均预留过多内存（这些内存本可用于减少 activation recomputation）；以及增加更细粒度 expert 带来的 per-expert 开销。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

修复后 MoE 层 MFU 从 24% 提升到 35%，才使"每层 MoE"在时间上成立。配套手段是 dropless MoE + 可调、固定大小的内存占用（fixed-size memory），以及降低 per-expert 开销。这条经验的关键点：**架构选择被实现效率反向约束**，先算清 MFU 归因再决定布局。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

## Muon 规模化：16K GPU 上的三处通信优化

MAI 报告 Muon 在放大到 MAI-Base-2 时训练动态更稳定、相对 AdamW 有明显效率收益，代价是要削掉 per-step 开销。针对 16K GPU 规模做了三类优化：多 batch 预取以重叠通信与计算；缓存并复用 resharding metadata，避免重复的索引逻辑（在参数形状高度重复的稀疏 MoE 中收益尤大）；router 等选定权重改为跨 GPU 复制而非 shard，跳过第一阶段通信，改为在 Newton-Schulz 之后做一次 all-gather 取回完整更新。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

> [!NOTE] 与 wiki 中 [[concepts/moe-mixture-of-experts-2025|MoE 架构]]、[[entities/deepseek-moe-parallel-strategy|DeepSeek MoE 并行策略]] 对照看：DeepSeek 系走的是细粒度 expert + 通信/计算重叠的工程路线，MAI 这版强调 optimizer 侧的 metadata 复用与"复制而非分片"，两者都在同一个约束（MoE 参数量爆炸下的通信墙）下做局部最优。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

## Activation gradient clipping：既是稳定器，也是数据质量探针

文章明确说 activation gradient clipping 让优化更稳定，而且**收益随规模增长**（大多数其他改动带来的效率增益在各规模上大致恒定）。更值得注意的是把它当诊断工具：文中给出一个被 activation gradient clipping 标出的畸形训练样本——代码数据里一段 license header 在句中被截断，回溯后发现是预处理流水线的 bug，已修复。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

这是一个可迁移的实践：把训练稳定性指标（梯度范数、裁剪触发率）反过来当作数据管线监控信号，成本远低于单独跑一遍数据质检。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

## 数据与评测口径

数据侧坚持"organic data，不做蒸馏"：所有训练 token 在内部处理，语料相对 MAI-Base-1 在 unique training tokens 上增长约 36%。评测用 bits-per-byte（BPB）以规避 tokenizer 差异；held-out code 用 MAI 自己的 codebase，STEM / Math / Q&A 由外部 vendor 出题以降低训练集污染风险，并对全部评测集做了 decontamination。对比对象是 2026 年 10 月前可获得的 frontier checkpoint，其中 Kimi K3、Qwen3.8-2.4T-A95B、MiMo-V2.6-Pro-RL 只有 post-trained 版本可取，文章明确标注了这一 caveat。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

Figure 1 的 BPB 数值（held-out code / QA / STEM / Math 四组）显示 MAI-Base-2 在四组上均为最低，例如 held-out code 上 0.2203，对比 DeepSeek-V4-Pro-Base 的 0.2370、Kimi K3 的 0.2264。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

## 对 wiki 的定位

这条记录的价值不在"又一个前沿模型发布"，而在三处可复用的工程知识：(1) 超参 scaling law 的拟合与验证流程（含可用的解析式）；(2) MoE 布局选择必须由 MFU 归因驱动，而不是由 FLOPs 优势驱动；(3) 把训练稳定性信号用于数据质量诊断。它与 [[concepts/scaling-laws|Scaling Laws]]、[[concepts/llm-pretraining-vs-sft|预训练 vs SFT]] 两个概念页构成"预训练配方"这一簇的一手材料。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

MAI-Base-2 目前处于 post-training 阶段，MAI-Base-3 的准备工作已开始。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md]

## 深度分析

### 三支柱中，难复制的是什么
scaling science、model-infrastructure co-design、scaling data 三根支柱的可复制性递减：scaling law 与 MFU 归因以流程形式公开，任何能做小规模消融的团队都能照搬；数据纪律（organic-only、in-house 处理全部 token）是难一夜建立的慢变量；最难复制的是 co-design 的组织闭环（建模、数据、infra 须在同一反馈环内迭代）与 16K 张 GB200 的资本门槛。公式可抄，闭环难抄——文章把 "owning the full stack" 放在叙事开头正说明这点。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:25-31,19]

### 超参 law：把耦合超参解析化
网格拟合 (LR, WD) 耦合的要点：单调 LR 时最优 WD 会漂移，一次给出联合最优而非两轮独立扫描。$LR^* \times WD^* = 17.04/\text{steps}$ 以 steps 计价，隐含"每 step 计算量固定"，对 horizon 30T→50T 的迭代尤其方便。dropout law 把正则强度挂到激活参数量而非总参数量上，与 MoE 视角一致：影响训练动态的是被激活的参数。更值得学的元做法：随规模变化的超参也走同一条"小规模拟合 → 中规模验证 → 全量使用"流水线，而非逐个 ablate——把超参选择本身也 scaling 化。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:49-59]

### MFU 归因的顺序：先证 FLOPs，再修时间
"每层 MoE" 的 EG FLOPs 优势早已成立，却被实现低效吃掉：dynamic routing 的内存碎片与极端负载预留挤占了本可减少 activation recomputation 的显存，更细粒度 expert 又叠加 per-expert 开销。修复后 MFU 24%→35%，布局选择才翻转。方法论顺序：先证 FLOPs 方向，再归因时间损耗，修复后才解锁布局决策——看修复后的时间，不看修复前的 FLOPs。对照 [[entities/ultaep-moe-expert-load-balancing-300us|UltaEP MoE 负载均衡]]：极端负载在那里被当作独立问题用调度解决，MAI 则从内存占用结构上消除预留，两条路线都承认"极端负载的尾部成本是 MoE MFU 的主要杀手"。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:65-75]

### Muon：收益在 dynamics，代价在 per-step
Muon 的收益侧是动态更稳定 + 相对 AdamW 的大幅效率增益；代价侧是 per-step 开销，且随集群规模线性放大——16K GPU 上任何同步等待都被复制 16000 倍。三处优化各消一类开销：多 batch 预取消通信-计算串行；缓存 resharding metadata 消重复索引逻辑（稀疏 MoE 的同形状参数复用率极高）；router 等选定权重复制而非分片，跳过第一阶段通信，Newton-Schulz 后一次 all-gather。与 [[entities/deepseek-moe-parallel-strategy|DeepSeek MoE 并行策略]] 对照：DeepSeek 从并行切分入手，MAI 从 optimizer 状态分布入手，殊途同归于"通信墙下的局部最优"。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:79-87]

### 数据纪律与评测口径互为表里
organic-only 与 BPB、vendor 出题、decontamination 互为证据：不用蒸馏数据，就无需为评测污染辩护；BPB 规避 tokenizer 差异，为"4–12x FLOPs 效率"的跨家族对比提供可比基准。对 Kimi K3 等对手只有 post-trained checkpoint 可取这一 caveat 的显式声明，是对比诚实度的信号——效率归因链（归于预训练本身而非 post-training 泄漏）依赖这些口径全部成立。对照 [[entities/deepseek-v4-training-methodology|DeepSeek V4 训练方法论]]（organic 与蒸馏两条路线的正面分叉）与 [[concepts/llm-pretraining-vs-sft|预训练 vs SFT]]，这条分叉决定了"预训练配方"类知识能否被独立验证。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:33-39,97]

### activation gradient clipping 作为廉价探针
其收益随规模增长（大多数改动的效率增益各规模恒定），是少数"越大规模越值得引入"的候选。当诊断工具的用法更值得记录：一个被 clipping 标出的畸形样本——代码数据里 license header 在句中被截断——回溯定位到预处理流水线 bug。机制是梯度范数对分布外 token 序列敏感，clipping 触发率天然是数据管线的无标注质检信号，成本远低于单独跑数据审计；实践中可接入告警面。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:89-93]

### 本文没有说的
空白即信息：没有数据配比、没有 BPB 之外的 capability 评测、post-training 只说 "undergoing"。这篇"配方级"披露覆盖的是 pre-training 的 infrastructure 侧（scaling ladder、MFU 归因、Muon 通信、clipping），modeling 侧的杠杆（数据 mixture）仍封闭——应把它定位为"前沿实验室愿意公开的部分"的样本，而非预训练配方本身。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:19,99-107]

## 实践启示

1. 对若干 (model size, TPP) 组合网格训练，拟合 2D 二次函数取 (LR*, WD*) 解析最小值；小规模拟合→中规模验证→全量使用。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:49-59]
2. 改 MoE 布局前先做 MFU 归因：dynamic routing 内存碎片、极端负载预留、per-expert 开销；dropless + fixed-size memory 是已验证修复。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:65-75]
3. Muon 上 16K GPU 三件事：多 batch 预取重叠通信；缓存 resharding metadata；router 等权重复制而非分片，Newton-Schulz 后一次 all-gather。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:79-87]
4. 把 activation gradient clipping 触发当数据管线告警：定位出的畸形样本可回溯预处理 bug；其收益随规模增长，优先引入此类改动、淘汰收益恒定者。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:89-93]
5. 评测口径先于对比实验：BPB 规避 tokenizer 差异，vendor 出题 + decontamination 降低污染质疑；对手 checkpoint 形态差异显式声明。^[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10.md:33-39]

关联：[[concepts/scaling-laws|Scaling Laws]]（解析式与验证流程一手材料）、[[entities/deepseek-v4-training-methodology|DeepSeek V4 训练方法论]]（organic 与蒸馏路线对照）、[[entities/ultaep-moe-expert-load-balancing-300us|UltaEP MoE 负载均衡]]（极端负载的另一解法）、[[entities/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|Scaling Domain Data Repetition]]（ladder 重复数据处理）。

→ [[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10|原文存档]]
