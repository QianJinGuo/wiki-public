---
title: "MAI-Base-2：微软 AI 前沿预训练（80B/2.8T MoE、scaling ladder 与 Muon 规模化）"
created: 2026-10-20
updated: 2026-10-20
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

→ [[raw/articles/microsoft-mai-base-2-frontier-pretraining-2026-10|原文存档]]
