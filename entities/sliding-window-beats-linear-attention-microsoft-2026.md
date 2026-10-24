---
title: "Sliding-window beats linear attention (微软 SWA vs 线性注意力)"
created: 2026-09-02
updated: 2026-10-25
type: entity
tags: [attention, linear-attention, sliding-window, inference, kv-cache, microsoft, transformer, arxiv]
sources: [raw/articles/sliding-window-beats-linear-attention-microsoft-2026]
confidence: 0.85
arxiv: "https://arxiv.org/abs/2608.28444"
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Sliding-window beats linear attention (微软 SWA vs 线性注意力)

微软团队发表论文证明：带 attention sinks 的滑动窗口注意力 (SWA) 在零后训练投入下，性能接近甚至超越需要数亿 token 后训练的线性化方法。

## 核心发现

### 0 Token vs. 千亿 Token

线性注意力方案（LoLCATs、Liger-GLA、QRWKV6、SUPRA 等）通过蒸馏或后训练将现成 Transformer 改造为线性化模型，训练量从 20M 到 100B token 不等。

微软提出的 **SWA(64,4)** — 总窗口 64、保留 4 个 attention sinks — **零 token、零后训练阶段**，在六项知识与推理任务上平均恢复原模型 **99.0%** 性能，与需要 350M–700M token 后训练的 QRWKV6（99.1%）几乎持平。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

MMLU 上 SWA 恢复率 **93.2%**，略高于 QRWKV6 的 92.4%。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

### 实验覆盖

覆盖 Phi、Mistral、Llama、Qwen、QwQ 等多个模型系列，参数规模从 1.3B 到数十 B。11 组比较中 SWA 有 9 组拿到除全注意力外的最高平均成绩。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

Qwen3-8B 上 SWA 平均 71.6，而 Gated DeltaNet、GLA、QRWKV6 经约 100M token 两阶段蒸馏后仍停留在 50 多分。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

### 长上下文优势

SWA 有效感受野通过网络深度逐层扩展（l 层后约 lw）。在 Llama 3.1 8B 的 S-NIAH 测试中：

- 窗口 128 + 上下文 4K：SWA 准确率 17.2 vs LoLCATs 1.6（**10.8 倍差距**）^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]
- BABILong 4K：SWA 15 vs LoLCATs 3^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

### 工程优势

- 窗口 < 512 时，SWA 状态内存与线性注意力相当或更低，吞吐更快^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]
- KV Cache 达窗口上限后保持固定，解码吞吐稳定^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]
- 窗口 64 时速度最快、内存最低^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

### 关键洞察

论文指出过去线性化工作很少直接和带 sinks 的 SWA 比较——普通 SWA（不带 sinks）在窗口滑动后 sink token 丢失会导致灾难性下降，容易低估 SWA 能力。SWA + attention sinks 已成为后续线性化工作难以绕开的 baseline。^[raw/articles/sliding-window-beats-linear-attention-microsoft-2026.md]

## 深度分析

### 被低估的基线：一场关于「对照组」的方法论事故

「线性注意力在长上下文更强」这个结论，至少有一部分是对照组的产物。领域内并非没有和 SWA 比过，而是比的是**不带 attention sinks 的普通 SWA**——一个在窗口开始滑动时就会结构性失效的版本。这不是调参不到位，而是少了一个组件：比较的一方是完整系统，另一方是缺件的系统。更值得追问的是，修好它几乎不花钱——总窗口 64 里只让出 4 个位置给 sink，剩下的照常滑动。也就是说，诚实的基线一直唾手可得。省略它并非算力所迫，而是实验设计里的注意力分配问题；而每一次低估 SWA，都会同步抬高后训练线性化的账面价值。

### attention sinks 修复的到底是什么

softmax 注意力有一个硬约束：每个 query 的可见 key 上概率必须归一到 1。语言模型会把大量「无所谓」的概率质量倾倒到序列开头几个位置上——它们是不承载具体语义的泄压阀。窗口一旦前移，这几个位置滑出可见范围，那份质量就无处可去，只能被强行摊到有语义的 token 上，注意力的形状被改写，输出开始漂移。固定保留 4 个 sink，等于给模型一个稳定的停车位：窗口怎么滑，概率预算的结构不变。这解释了为什么差距在长上下文反而拉大——上下文越长，窗口滑动的次数越多，每一次滑动都是一次注入畸变的机会，误差沿长度累积，而不是被摊薄。

### 「免费」的边界：0 token 不等于 0 代价

99.0% 对 99.1% 是同类项比较，但成本一侧极度不对称：一边零权重改动、零蒸馏阶段，另一边是 350M–700M token 的两阶段训练。这里的「免费」有明确边界。其一，SWA 维持的仍是一份**有界** KV cache，不是线性注意力那种压缩后的循环状态——状态大小是窗口的函数，窗口必须选，而且选大了账就变：SWA-512 的状态内存已经高于线性注意力。其二，99.0% 是恢复率不是持平，MMLU 上那 7 个百分点是实打实缺的。对检索密集型任务，每层 60 个 token 的局部历史加上约 lw 的有效感受野是另一种能力等级——BABILong 4K 上 SWA 15 分而全注意力 60 分，天花板依然在头顶。它买到的是一个更好的默认选项，不是普适替代品。

### 工程上它赢在可预测性，而非峰值指标

分页 KV cache 遇上固定窗口，block table 的大小可以被静态推导出来，不再随请求长度漂移；准入控制、batch 组装、显存规划都能在请求开始前算完，吞吐曲线也从「超过约 1K 后明显下滑」变成一条平线。kernel 层面同样省事：滑动窗口掩码是一条固定带宽，不需要循环状态的扫描逻辑，也没有分块并行的特殊形式，能直接复用现有 flash-attention 类实现；而那 4 个固定 sink 天然是一段稳定前缀，可以直接吃 prefix cache。[[entities/baidu-unlimited-ocr-sliding-window-attention|百度 Unlimited OCR 的 R-SWA]] 是同一逻辑的极端版：参考 token 全程可见，输出侧只留最近 128 个 token，KV cache 变成常量。需要提醒的是，论文那组测速是 4 层模型、batch size 1、单卡，它证明的是状态内存与单请求吞吐，不是服务系统指标——[[concepts/inference-optimization|推理系统优化]] 里真正决定收益的往往是 p99 尾部，而有界缓存恰好拆掉了尾部最主要的驱动因素。

### 可迁移的判断规则

接受任何架构主张之前，先审计两件事：基线的调参预算是否对等，以及基线的必需组件是否齐全。一个从对照组身上摘掉廉价标准件的比较，不构成保守比较，它是优待比较。推论一：看到「X 打败 Y」，先问 Y 被允许拥有什么。推论二：新颖性主张衰减最快的地方，正是领域共享基线定义最模糊的地方——找到真结果最便宜的办法，往往是把那个显而易见的基线认真重跑一遍。

## 实践启示

1. **把 SWA + sinks 写进 baseline 表**：评估任何线性化或稀疏化方案时，至少保留 4 个 sink 位置。不带 sinks 的 SWA 不是弱基线，是坏掉的基线，会让整张表系统性偏向线性化一侧。
2. **在真实服务的最长上下文上测**：不要用短上下文外推。BABILong 上 LoLCATs 短上下文略占优、2K 后被反超、4K 时是 15 对 3——交叉点就是结论翻转的地方。
3. **把 KV 内存预算当一阶约束**：先定窗口 w，再谈注意力变体。窗口小于 512 时 SWA 的状态内存与线性注意力相当或更低且更快；越过这条线账要重算，SWA-512 已经更贵。
4. **优先拿不用重训的收益**：SWA(64,4) 零 token 换回 99.0%，和 350M–700M token 的 QRWKV6 的 99.1% 持平。把后训练预算留给它拿不到的那 1%，以及检索密集型能力。
5. **在 serving 侧按「固定前缀 + 滑动块」来设计**：4 个 sink 走 prefix cache，滑动区走分页 block，调度器按窗口静态规划显存，省掉按请求增长做的动态预留。
6. **报数字前先对齐测速条件**：4 层、batch 1、单卡的结果只说明状态内存和单请求吞吐。生产结论要在自己的并发与 SLO 下重测，尤其是高并发时的尾延迟。

## 与 [[entities/mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo v2.5 混合 SWA]] 的关系

MiMo 的混合 SWA 架构也采用滑动窗口注意力作为推理优化手段。本文从理论上验证了 SWA 作为线性注意力替代方案的有效性，为 MiMo 等采用 SWA 的模型提供了理论支撑。

→ [[raw/articles/sliding-window-beats-linear-attention-microsoft-2026|原文存档]]
