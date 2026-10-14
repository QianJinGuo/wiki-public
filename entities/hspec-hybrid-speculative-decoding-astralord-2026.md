---
title: "H-Spec 混合投机解码 drafter（Mamba-2 + 就地注意力）"
created: 2026-10-13
updated: 2026-10-13
type: entity
tags: [inference-optimization, speculative-decoding, mamba, architecture]
sources: [raw/articles/hspec-hybrid-speculative-decoding-astralord-2026]
confidence: 0.75
---

# H-Spec 混合投机解码 drafter

H-Spec 是一个混合骨干投机解码 drafter，核心思想是把"上下文摘要"与"逐字细节"两个职责拆给两类模块：Mamba-2 线性注意力层提供与上下文长度无关的固定大小状态摘要，注意力层负责无法从有损摘要恢复的精确 token 细节。架构上 4 层 drafter 每层以 Mamba-2 开头、其中 3 层后接注意力；Mamba 模块不扫描上下文——其初始状态 U₀ 由 target 模型 5 个层的末位隐状态融合投影而来，然后只对 block 内位置做一次并行扫描（线性递推允许整块单遍并行），同一 mask token 在不同位置获得不同状态；注意力模块与 target 的 KV 布局匹配，就地读取 target 3 个层的 keys/values（2,048-token 滑窗），不维护自己的缓存；顶层再叠 DSpark 的 Markov head。对 Qwen3-8B，Mamba 状态每层 48 heads × 64 × 4 = 12,288 个数、每轮重建，drafter 在轮与轮之间不保留任何随上下文增长的状态^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md]。

消融实验证明两半都必要：固定架构下去掉 target 的 KV 就地读取使接受长度下降 24-31%，去掉摘要使接受长度下降 7-12%；同参数量下混合骨干比 Mamba-only drafter 高 29-45%，比 attention-only 高 13-15%。与同配方重训的 P-EAGLE、DFlash、DSpark 对比，H-Spec 在 Llama-3.1-8B、Qwen3-4B、Qwen3-8B 上接受长度分别提升 13.3%、5.0%、8.7%，且优势从 1k 到 32k 上下文保持；vLLM 中在最低 KV cache 利用率点达到 5-17% 的峰值吞吐提升，且优势随并发增长。代价是计算图更深——低于约 32 并发请求或 512 输入 token 时混合骨干的起草速度慢于 DFlash，且注意力层必须复制 target 的 KV 布局^[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026.md]。

定位：与 DFlash（投影 target 隐状态为自己 KV，缓存随 token 增长）和 DSpark（Markov head）构成 drafter 家族的第三代——服务引擎已收敛到少数几类 drafter 家族（vLLM 的 AMD GPU 基准恰好比较四类），H-Spec 的贡献是证明 hybrid 目标模型（如 Qwen3.5 类线性注意力 + 少量全注意力）的 drafter 可以同时吃两层红利。

## 关联

- [[concepts/speculative-decoding|投机解码]]
- [[concepts/inference-optimization|推理优化]]
- [[entities/deepseek-dspark-speculative-decoding-2026|DeepSeek DSpark 投机解码]]
- [[entities/lmsys-dflash-speculative-decoding-2026-06|LMSYS DFlash 投机解码]]
- [[entities/approximate-speculative-decoding-asd-relaxed-validation-2026|ASD 近似投机解码]]

→ [[raw/articles/hspec-hybrid-speculative-decoding-astralord-2026|原文存档]]
