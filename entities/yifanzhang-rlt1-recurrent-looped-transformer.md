---
title: "Recurrent Looped Transformer (RLT-1)"
created: 2026-09-22
updated: 2026-10-06
type: entity
tags: [architecture, transformer, efficiency]
sources: [raw/articles/yifanzhang-rlt1-recurrent-looped-transformer]
confidence: 0.7
---

# Recurrent Looped Transformer (RLT-1)

## 摘要

RLT-1 通过 gated cross-token recurrence 将上一 token 最终 decoder state 注入当前 token decoder input，计算深度随序列长度扩展；parity/permutation-tracking 任务多 seed 量化验证（与 Raschka 对 Astra looped 的 debunk 形成对照）^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]

## 核心要点

- v*c=49（value=7, confidence=7, stars=3），newsletter ingest 2026-09-22
- 详细分析见原文存档

## 深度分析

### 与 "looped transformer"（层复用）的本质区别

社区常把 RLT-1 这类设计与 Nanbeige 4.2 / OpenAI Astra 口中的 "looped transformer" 混为一谈，但两者机制不同。Nanbeige 式 looped transformer 是**层复用**：把同一个 22 层 stack 跑两遍，等效加深模型但不增加参数，代价是约 2x compute，训练侧只保留了约 75% 的 token efficiency——本质上只是权重共享式的深度扩展^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]。RLT-1 则是 **token 间递归**：通过 gated merge 将上一 token 的最终 decoder state 注入当前 token 的 decoder input，形成沿序列延伸的递归路径；层复用只是它的可选权重共享手段，而非定义性特征^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。混淆两者会低估 RLT-1：它改变的是信息在 token 维度的流动拓扑，而不只是加深单次前向。

### 深度随序列增长的计算路径

RLT-1 最有辨识度的性质：per-token 的 decoder block 评估次数固定为 L_D，但经 t 个 token 后计算路径已累积穿越 t × L_D 个 decoder block——等效深度随序列线性增长，而 per-token 推理成本保持 O(1)（SWA 窗口固定、全局 encoder KV memory 限制在当前 prefix）^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。这提供了一条与 "把模型做大" 不同的扩容轴：Raschka 指出 recurrent/looped 方法若减少中间 reasoning token 的显式生成，效果上等价于把更多计算放进 latent 状态，与直接 scale 模型尺寸并无本质区别^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]。RLT-1 把这条 latent 计算路径显式制度化为递归状态 s_t，且实验证明它对特定算法任务确实有效。

### 实验证据：parity 与 permutation tracking 的量化优势

108 个 run（5 个八层 split × 6 任务 × 3 seeds，2000 optimizer steps，宽度 512）给出了一组少见的严格多 seed 对照^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。两个最醒目的结果：

- **Parity**：step 500 时 6+2 split 达 99.44±0.98%，同期 Transformer 8 仅 48.48±0.53%；step 2000 时 4+4 至 7+1 全部 split 在三个 seed 上均达 100%，Transformer 8 为 94.84±3.43%；256 bit 长度上 5+3 与 7+1 仍保持 100%^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。
- **Swaps permutation tracking（长度 512）**：4+4 split 达 55.70±25.78% final-state accuracy（prefix-token accuracy 91.16±6.09%），Transformer 8 仅 0.85±0.30%——从几乎完全失败到可用的质变^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。

解释很自然：这两类任务本质是**状态追踪**——答案依赖把大量逐步变换压缩进一个隐状态，递归 state 正好提供了显式载体，而标准 Transformer 只能靠 attention 逐步回溯前缀。这与 Mixture-of-Recursions 的 per-token 自适应深度同属一个动机家族（让难 token 获得更多计算），只是 RLT-1 是无条件均匀递归而非 router 动态分配^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]。

### 增益的边界：任务依赖、split 依赖与训练代价

同一组实验也诚实地划出了边界。加法（teacher-forced）在 7+1 split 上 9 位操作数时为 68.05±3.64%，但 32 位时所有模型都跌到 14.89–16.84%——递归并未带来长度泛化；flat mod-5 对初始化极敏感（5+3 达 94.18% 但 SD 高达 7.29，Transformer 8 为 64.02±37.64%）^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。encoder/decoder 深度配比（4+4 到 8+0）本身也是超参，任务间排序并不一致。架构代价明确：递归依赖使训练与 prompt prefill 的 decoder 更新必须串行，冲击并行度；还引入 TBPTT（本实验设 128）等训练复杂度^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。家族内部对照说明反馈是关键变量：去掉反馈路径的 RLT-0 每 split 少约 787,968 个参数；RLT-2 改为 chunk 粒度反馈，B=1 时严格退化回 RLT-1^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。

## 实践启示

1. **别把 "looped" 当一个词用**：评估或复现时先区分 layer-reuse（Nanbeige/Astra 式）与 token-recurrence（RLT-1 式）——前者是参数效率 trade-off，后者改变序列维度的计算拓扑，两者结论互不可迁移^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]。
2. **按任务类型选架构**：RLT-1 在状态追踪类任务（parity、swaps）上是质变级提升，在加法长度泛化上无优势；workload 以算法状态压缩为主时值得投入，以知识/语言能力为主时收益不明确^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。
3. **把深度配比当作一级超参**：4+4 到 8+0 的 split 显著影响结果且任务间排序不稳定；mod-5 上 37.64% 的 SD 提示单 seed 结论不可信，至少跑 3 seeds^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。
4. **预算里算上串行化代价**：token 间递归使 prefill 无法完全并行，长 prompt 场景是实打实的 latency 税；RLT-2 的 chunk 粒度反馈（B>1）是恢复部分并行度的现成折中^[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer.md]。
5. **对 "隐藏 CoT" 叙事保持怀疑**：recurrent/looped 深度本身不会隐去可见的 chain-of-thought，只是把更多计算搬进 latent 状态；任何 "无法解释的推理" 宣传都需与普通 scale-up 区分开验证^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]。
6. **对照阅读两个证据源**：Raschka 的 debunk（层复用 = tiny tweak）与本实验的正面结果（token 递归 = 状态追踪质变）并不矛盾——它们说的不是同一机制；引用任一方前先确认所指架构。

相关：[[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026]]

→ [[raw/articles/yifanzhang-rlt1-recurrent-looped-transformer|原文存档]]
