---
title: "吴恩达参与斯坦福新作：丢掉中间推理Token，长思考最高提速3倍"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: ['ai', 'training', 'inference', 'research', 'coding']
sources: [raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍]
confidence: 0.7
---

# 吴恩达参与斯坦福新作：丢掉中间推理Token，长思考最高提速3倍

# 吴恩达参与斯坦福新作：丢掉中间推理Token，长思考最高提速3倍

原创 让你更懂AI的 2026-08-28 21:55 北京

越想越久，不必越算越贵

## 

推理链越来越长，模型未必需要记住全部历史。Prefix Sliding 只保留任务前缀和最近窗口，让长思考不再背着整条推理链前进。

一条推理链越来越长之后，模型究竟需要记住多少？

按照标准全注意力的做法，答案几乎是全部。

此前完成的计算、中间步骤、不断累积的推理历史，都会继续留在后续注意力范围里。**推理越长，每生成一个新 token 的成本也随之上涨。**

问题在于，模型真的还需要反复回看这些已经过去的中间步骤吗？

最近，斯坦福、华盛顿大学等机构的一项联合研究从这里切入，吴恩达、Yejin Choi、Percy Liang 等参与其中。

团队提出 **Prefix Sliding** ，只长期保留任务前缀和最近的推理窗口，让更早的中间 token 逐步退出后续注意力计算。

最终，**现有模型无需重新训练，长思考最高约 3 倍提速 。用于强化学习后，单条推理轨迹还能扩展到 10 万 token 以上。**

论文标题：

Prefix Sliding for efficient test-time scaling

论文地址：

<https://arxiv.org/abs>/2608.26070

代码地址：

<https://github.com/Muennighoff/prefix-sliding>

注意力为何集中在推理两端

全注意力的代价会随着推理链长度持续上升，但模型对这些历史 token 的使用并不均匀。

团队分析 Qwen3-1.7B 在 AIME25 上的一条推理轨迹后发现，**注意力呈现出明显的“两头高、中间低”** 。

最前面的 Prefix 和最近生成的一段 token 获得更多关注，中间大量推理 token 的注意力则整体处于低位。

〓长推理中的注意力主要集中在前缀和最近生成的 token

Prefix 中保存着系统指令、任务描述和工具信息，其中最前面的几个 token，尤其前 4 个 token，还承担 attention sink 的作用。最近生成的 token 则对应模型当前正在处理的问题。

论文用一个简单算式 ((42 + 84) × 4) - 5 为例：完成 42 + 84 后，后续计算可能只需要这个结果，不再需要保留此前完整的推理过程。

**这也构成了 Prefix Sliding 的直接动机： 后续计算未必需要始终携带完整的推理历史。**

Prefix Sliding如何丢掉中间Token？

Prefix Sliding 固定保留任务前缀，最近 W 个推理 token 构成滑动窗口。随着生成继续，**更早的中间 token 会逐步从后续注意力计算所使用的 KV Cache 中移除。**

假设 Prefix 有 100 token、窗口为 4096，即使完整推理已经达到 10 万 token，**后续注意力计算最多只需保留约 4196 token 。**

**窗口填满后，单步成本不再随着完整推理链增长。**

〓Prefix 固定保留，最近的推理 token 随窗口向前移动

普通滑动窗口会让最开始的任务信息逐步移出窗口，**Prefix Sliding 则始终保留 Prefix 。**

团队最终采用 Continue PE。附录在 AIME25 上的实验显示，它与 Reset PE 表现相近，同时无需反复应用新的位置编码，可以直接复用已有缓存表示，计算更高效。

团队还针对 NVIDIA Hopper 实现了专用 FlashAttention 内核。

对完全位于 Prefix 和滑动窗口之外的 tile 直接跳过，对与有效区域部分重叠的 tile 做元素级 Mask，从而减少无效加载和计算，**实际速度也接近普通滑动窗口的实现 。**

〓窗口填满后，Prefix Sliding 的生成吞吐趋于稳定

无需训练最高提速3倍，RL扩展到10万Token

在无需额外训练的设置下，作者以 Qwen3-1.7B 为主模型，在 AIME25、GPQA 和 MATH500 上进行评测。

**Prefix Sliding 的优势来自同样思考时间内可以生成更多 token，并非单个 token 本身质量变高。**

以 4096 token 窗口为例，Prefix Sliding 在 AIME25、GPQA 和 MATH500 上与全注意力表现接近，但在长序列下吞吐明显更高，128K 时达到 5224 tok/s，而全注意力只有 448 tok/s。

〓相同思考时间下，Prefix Sl

→ [[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍|原文存档]]
