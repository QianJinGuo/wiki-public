---
title: "H-Spec 混合投机解码 drafter（Mamba-2 + 就地注意力）"
created: 2026-10-13
updated: 2026-11-22
type: entity
tags: [inference-optimization, speculative-decoding, mamba, architecture]
sources: [raw/articles/hspec-hybrid-speculative-decoding-astralord-2026]
confidence: 0.75
---

# H-Spec 混合投机解码 drafter

H-Spec 是一个混合骨干投机解码 drafter，核心思想是把"上下文摘要"与"逐字细节"两个职责拆给两类模块：Mamba-2 线性注意力层提供与上下文长度无关的固定大小状态摘要，注意力层负责无法从有损摘要恢复的精确 token 细节。架构上 4 层 drafter 每层以 Mamba-2 开头、其中 3 层后接注意力；Mamba 模块不扫描上下文——其初始状态 U₀ 由 target 模型 5 个层的末位隐状态融合投影而来，然后只对 block 内位置做一次并行扫描（线性递推允许整块单遍并行），同一 mask token 在不同位置获得不同状态；注意力模块与 target 的 KV 布局匹配，就地读取 target 3 个层的 keys/values（2,048-token 滑窗），不维护自己的缓存；顶层再叠 DSpark 的 Markov head。对 Qwen3-8B，Mamba 状态每层 48 heads × 64 × 4 = 12,288 个数、每轮重建，drafter 在轮与轮之间不保留任何随上下文增长的状态^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md]。

消融实验证明两半都必要：固定架构下去掉 target 的 KV 就地读取使接受长度下降 24-31%，去掉摘要使接受长度下降 7-12%；同参数量下混合骨干比 Mamba-only drafter 高 29-45%，比 attention-only 高 13-15%。与同配方重训的 P-EAGLE、DFlash、DSpark 对比，H-Spec 在 Llama-3.1-8B、Qwen3-4B、Qwen3-8B 上接受长度分别提升 13.3%、5.0%、8.7%，且优势从 1k 到 32k 上下文保持；vLLM 中在最低 KV cache 利用率点达到 5-17% 的峰值吞吐提升，且优势随并发增长。代价是计算图更深——低于约 32 并发请求或 512 输入 token 时混合骨干的起草速度慢于 DFlash，且注意力层必须复制 target 的 KV 布局^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md]。

定位：与 DFlash（投影 target 隐状态为自己 KV，缓存随 token 增长）和 DSpark（Markov head）构成 drafter 家族的第三代——服务引擎已收敛到少数几类 drafter 家族（vLLM 的 AMD GPU 基准恰好比较四类），H-Spec 的贡献是证明 hybrid 目标模型（如 Qwen3.5 类线性注意力 + 少量全注意力）的 drafter 可以同时吃两层红利。

## 深度分析

### 有损摘要与精确检索：混合骨干的第一性原理

Mamba-2 的线性递推与注意力在"记忆"上的本质差异在于：注意力必须为每个历史 token 保存一对 key/value，内存随上下文线性增长，且每次生成都要读取全部缓存；而 Mamba 层只保留一个大小与上下文长度无关的状态 U，靠可学习的、依赖输入的遗忘因子自己决定保留什么、丢掉什么。这正是 Qwen3.5 这类混合目标模型能把 KV cache 做小的原因——它们大部分层都是这类线性注意力层，只保留少量全注意力层兜底。H-Spec 的设计哲学是把这对矛盾正面接下来：与其天真地复制 target 的架构配方，不如在 drafter 内部同样按"上下文摘要 / 逐字细节"二分职责，让两类模块各司其职。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:10-12]

### 免费午餐的边界：从 target 取输入但不留状态

H-Spec 对 target 的利用方式有两个反直觉之处。其一，Mamba 模块完全不扫描上下文：五个 target 层的末位隐状态被融合投影成初始状态 U₀，之后只对 block 内位置做一次并行扫描——线性递推允许整块单遍并行，同一个 mask token 在不同位置自然获得不同状态，无需任何特殊处理。其二，注意力模块不建自己的缓存，而是匹配 target 的 KV 布局，通过 2,048-token 滑窗就地读取三个 target 层的 keys/values，补上有损摘要无法恢复的 token 级细节。整个 drafter 每轮从零重建：Qwen3-8B 上每层 Mamba 状态是 48 heads × 64 × 4 = 12,288 个数，轮与轮之间不保留任何随上下文增长的状态，也没有任何需要跨轮注入的缓存。顶层的 DSpark Markov head 则是第三处对现有家族成果的直接复用。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:13]

### 消融数字揭示的不对称依赖

把两组消融放在一起看，依赖是不对称的：固定架构下去掉 target KV 就地读取使接受长度下降 24-31%，而去掉摘要只下降 7-12%——逐字细节通道的价值约为摘要通道的 2-4 倍。同参数量对比进一步说明收益来自"混合"本身而非容量堆叠：混合骨干比 Mamba-only drafter 高 29-45%，比 attention-only 高 13-15%。但代价同样来自混合：更深的计算图使混合骨干在低于约 32 并发请求或 512 输入 token 时，起草速度反而慢于 DFlash。三项收益与一项代价指向同一个结论——混合架构的优势依赖服务负载达到一定规模才兑现。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:15]

### drafter 家族的收敛与 H-Spec 的卡位

服务引擎已收敛到少数几类 drafter 家族——vLLM 的 AMD GPU 基准恰好比较四类——这意味着 drafter 设计正从自由探索进入家族内竞争。DFlash 把 target 隐状态投影为自己的 KV，缓存随 token 增长；DSpark 的 Markov head 只做局部外推；H-Spec 则占据了一个此前空缺的位置：当目标模型本身是混合骨干（如 Qwen3.5 类线性注意力为主、少量全注意力）时，hybrid drafter 可以同时吃两层红利——Mamba 侧匹配 target 的线性注意力行为，注意力侧直接复用 target 已有的 KV 缓存而非另建一份。在 Llama-3.1-8B、Qwen3-4B、Qwen3-8B 上接受长度分别提升 13.3%、5.0%、8.7%，且优势从 1k 到 32k 上下文保持不变，也印证了摘要通道与上下文长度解耦的设计意图。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:14-16]

## 实践启示

1. 接受长度对两条通道的依赖不对称：KV 就地读取损失 24-31%，摘要只损失 7-12%。为混合 drafter 做工程优化时，优先保障 KV 布局兼容与滑窗读取路径的正确性，摘要是其次。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:15]
2. 部署前先看负载画像：低于约 32 并发请求或 512 输入 token 时，混合骨干起草速度慢于 DFlash——小流量、短上下文服务选 DFlash 类更划算，高并发长上下文才是 H-Spec 的主场。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:15]
3. vLLM 中 5-17% 的峰值吞吐提升出现在 KV cache 利用率最低的点，且优势随并发增长——把 hybrid drafter 安排在 KV cache 压力最大的高峰时段，收益最大。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:15]
4. drafter 无状态化（每轮重建固定大小的 Mamba 状态，不保留随上下文增长的东西）显著简化内存管理：没有缓存要回收、没有状态要跨轮注入，对多租户、多请求并发的服务场景尤为友好。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:13]
5. 注意力层必须复制 target 的 KV 布局是硬工程约束：目标模型换代或 KV 布局改动时 drafter 需要同步适配，选型时应评估目标模型的迭代节奏与兼容成本。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:15]
6. 面向 Qwen3.5 类混合目标模型的时代，"线性注意力摘要 + target 就地 KV"的混合 drafter 模式值得作为下一代 drafter 的默认候选来评估，而不是默认沿用单一骨干配方。^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md:12-13]

## 关联

- [[concepts/speculative-decoding|投机解码]]
- [[concepts/inference-optimization|推理优化]]
- [[entities/deepseek-dspark-speculative-decoding-2026|DeepSeek DSpark 投机解码]]
- [[entities/lmsys-dflash-speculative-decoding-2026-06|LMSYS DFlash 投机解码]]
- [[entities/approximate-speculative-decoding-asd-relaxed-validation-2026|ASD 近似投机解码]]

→ [[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026|原文存档]]
