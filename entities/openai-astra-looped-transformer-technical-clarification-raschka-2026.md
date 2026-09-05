---
title: "OpenAI Astra 与循环 Transformer（looped transformer）技术辨析"
created: 2026-09-05
updated: 2026-09-06
type: entity
tags: [model-architecture, transformer, looped-transformer, recurrent-depth, nanbeige, compute-redundancy, mesh, spiralformer, memory-buffer, multi-scale]
sources:
  - raw/articles/openai-astra-looped-transformer-debunk-raschka-2026
  - raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06
confidence: 0.7
---

# OpenAI Astra 与循环 Transformer（looped transformer）技术辨析

> Sebastian Raschka 针对「OpenAI Astra 是 looped/recurrent-depth transformer」的市场 hype 做的技术澄清：所谓 looped transformer 只是一个极小的架构改动（复用层堆叠），不应作为 Astra 的宣传卖点。^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]

## 核心机制：层堆叠复用

Looped transformer 的关键是**复用 transformer block 的层堆叠**来增加容量而不加参数。以开源模型 Nanbeige 4.2 为例（28T tokens 从头预训练）：把同一个 22 层堆叠（=一个 transformer block）复用**两次**而非一次，等价于把 22 层架构「扩展」到 44 层，但**不复制权重**。^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]

这带来三方面的权衡：

- **存储/RAM 不变**：因为组件复用，模型体积与单遍架构相同（忽略 embedding 与输出层）
- **计算成本接近翻倍**：文本要经过接近两倍的层数
- **token 效率约 75%**：Nanbeige 4.2 技术报告发现两遍是最优权衡；更多遍收益甚微但训练显著变慢变贵

## 历史脉络

这一想法并非 Nanbeige 首创，可追溯到 NeurIPS 论文「Mixture-of-Recursions: Learning dynamic recursive depths for adaptive token-level computation」。该论文的机制更精巧——加入一个**学习到的 router**，决定每个 token 经过一遍、两遍还是更多遍，让容易的 token 提前退出、困难的 token 获得更多计算。^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]

## 对「Astra 隐藏 CoT」说法的纠正

媒体声称循环架构「以模糊部分或全部推理（chain-of-thought）的方式工作」，Raschka 认为此说法不一定成立：^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]

- **复用层本身不会抑制可见的 CoT**——它和下普通 transformer 层一样，在发射下一个 token 前于 hidden states 增加计算
- 唯一合理直觉解释是：如果模型用更多递归遍数，可能**生成更少的中间推理 token**，于是更多计算发生在无法被读取为文本的 latent activations 中
- 但这与单纯扩模型规模（如 GPT 5.6 Luna → GPT 5.6 Sol）得到的效果相同

## 结论

Astra 可能确实是好模型，但「looped transformer 方面」只是一个微小的架构 tweak，不应成为宣传卖点。真正有技术含量的是 **Mixture-of-Recursions 的 token 级自适应路由**，而非简单的层复用。^[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026.md]

## 相关

- [[concepts/agent-harness-engineering-paradigm|Harness Engineering]]
- [[entities/agent-architecture-harness-new-backend|Agent 架构新后端]]

→ [[raw/articles/openai-astra-looped-transformer-debunk-raschka-2026|原文存档]]

## 2026-09-06 SUPP：循环 Transformer 的「计算冗余」病理与两面修复（阿里 MeSH + SpiralFormer）

> 量子位《GPT-6 带火循环 Transformer，阿里早已布局》报道阿里及合作高校在循环 Transformer 高效化上的两篇顶会论文（MeSH ICLR 2026 / SpiralFormer EMNLP 2026），与本文档早先的 Astra 技术辨析互补——Raschka 回答「looped transformer 是什么」，本文补充「为什么裸循环会空转、怎么修」。 ^[raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06.md]

### 三个「摸鱼证据」：循环空转的实证诊断

对循环模型每轮 hidden state 变化的内部检查发现：① 第一个 core loop 承担绝大部分更新，后续循环的状态变化迅速缩小（柱子变矮是计算增量变小，不是算力减少）；② CKA 分析显示相邻轮次表示高度接近（同一组网络转了一圈又一圈吐的东西差不多）；③ 奇异值谱显示表示逐渐挤进低维空间（表达越来越单一）。三个证据共同指向：第一个 core loop 猛干、后面几个纯陪跑。 ^[raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06.md]

病理诊断为两个：**计算无分化**（同一组网络反复使用、缺少轮次信息，难以在不同阶段形成分工）与**信息过载**（hidden state 同时承担原始输入、前几轮结果与当前计算，长期记忆与临时加工互相挤压）。 ^[raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06.md]

### MeSH：Memory Buffer + 动态读写路由器

MeSH（ICLR 2026，arXiv 2510.07739，2025-10 首版）引入一个可动态存取的 Memory Buffer（多个记忆槽保存原始输入与循环积累信息），配 Write Router（决定新一轮结果按什么权重分摊到各槽）与 Read Router（下一轮开始前按权重读取槽位重组 hidden state），每轮循环、每个 token 各有不同读写权重——新信息各存多少、历史信息各取多少由模型自己学习。计算形式与 Hyper-Connections/mHC 同族（多路状态 + 可学习映射组合），MeSH 把它用在循环之间。^[raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06.md]

Pythia-1.4B 级实验：权重共享减约 33% 非嵌入参数，零样本下游平均准确率反而从普通 Transformer 的 49.50% 提高到 50.56%（+1.06pp）；新增路由器参数仅占非嵌入参数约 0.005%，额外计算开销约 0.014%。 ^[raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06.md]

### SpiralFormer：每轮看不同尺度的世界

SpiralFormer（EMNLP 2026，arXiv 2602.11698）进一步把每轮循环的序列长度 seq_len 变成可调的：每轮循环前把相邻 token 聚合成更粗表示（1/8 → 1/4 → 1/2 → 完整序列，从粗到细），算完再分配回原 token 位置，写回结果做右移避免泄露未来信息（受 Coconut 隐式思考启发）。Pythia-1.4B 级实验：参数基本相同，计算量从 14.08T 降至 13.13T FLOPs，5-shot 下游平均准确率从 51.93% 提高到 54.37%。 ^[raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06.md]

Attention probing 验证「先全局后局部」分工确实被学到：早期低分辨率循环注意力分布更广，后期高分辨率循环集中到少数关键位置、局部注意力占比上升——对照普通 LoopedFormer 趋势更弱且不稳定，说明多分辨率设计是关键驱动因素而非循环自动涌现。循环层占比实验呈 U 形曲线，30%–40% 达到最佳。 ^[raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06.md]

### 与 Raschka 辨析的关系

Raschka 澄清 looped transformer 只是层堆叠复用的微小架构改动（Nanbeige 4.2 两遍最优、token 效率约 75%）；MeSH/SpiralFormer 则说明裸循环的瓶颈不在「复用层数」而在「每轮算得值不值」——MeSH 管每轮「拿什么算」（信息分工），SpiralFormer 管「以什么粒度算」（尺度分工）。两者合起来回答了循环架构从学术路线走向下一代高效 LLM 候选的卡点是否可拆解的问题。 ^[raw/articles/mesh-spiralformer-recurrent-transformer-ali-qbitai-2026-09-06.md]