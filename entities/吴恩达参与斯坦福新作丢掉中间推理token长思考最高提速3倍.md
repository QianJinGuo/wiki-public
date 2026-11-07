---

title: "吴恩达参与斯坦福新作：丢掉中间推理Token，长思考最高提速3倍"
created: 2026-08-30
updated: 2026-11-07
type: entity
tags: ['ai', 'training', 'inference', 'research', 'coding']
sources: [raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 吴恩达参与斯坦福新作：丢掉中间推理Token，长思考最高提速3倍

## 吴恩达参与斯坦福新作：丢掉中间推理Token，长思考最高提速3倍

原创 让你更懂AI的 2026-08-28 21:55 北京

越想越久，不必越算越贵


推理链越来越长，模型未必需要记住全部历史。Prefix Sliding 只保留任务前缀和最近窗口，让长思考不再背着整条推理链前进。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

一条推理链越来越长之后，模型究竟需要记住多少？

按照标准全注意力的做法，答案几乎是全部。

此前完成的计算、中间步骤、不断累积的推理历史，都会继续留在后续注意力范围里。**推理越长，每生成一个新 token 的成本也随之上涨。** ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

问题在于，模型真的还需要反复回看这些已经过去的中间步骤吗？

最近，斯坦福、华盛顿大学等机构的一项联合研究从这里切入，吴恩达、Yejin Choi、Percy Liang 等参与其中。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

团队提出 **Prefix Sliding** ，只长期保留任务前缀和最近的推理窗口，让更早的中间 token 逐步退出后续注意力计算。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

最终，**现有模型无需重新训练，长思考最高约 3 倍提速 。用于强化学习后，单条推理轨迹还能扩展到 10 万 token 以上。** ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

论文标题：

Prefix Sliding for efficient test-time scaling

论文地址：

<https://arxiv.org/abs>/2608.26070

代码地址：

<https://github.com/Muennighoff/prefix-sliding>

注意力为何集中在推理两端

全注意力的代价会随着推理链长度持续上升，但模型对这些历史 token 的使用并不均匀。

团队分析 Qwen3-1.7B 在 AIME25 上的一条推理轨迹后发现，**注意力呈现出明显的“两头高、中间低”** 。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

最前面的 Prefix 和最近生成的一段 token 获得更多关注，中间大量推理 token 的注意力则整体处于低位。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

〓长推理中的注意力主要集中在前缀和最近生成的 token

Prefix 中保存着系统指令、任务描述和工具信息，其中最前面的几个 token，尤其前 4 个 token，还承担 attention sink 的作用。最近生成的 token 则对应模型当前正在处理的问题。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

论文用一个简单算式 ((42 + 84) × 4) - 5 为例：完成 42 + 84 后，后续计算可能只需要这个结果，不再需要保留此前完整的推理过程。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

**这也构成了 Prefix Sliding 的直接动机： 后续计算未必需要始终携带完整的推理历史。** ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

Prefix Sliding如何丢掉中间Token？

Prefix Sliding 固定保留任务前缀，最近 W 个推理 token 构成滑动窗口。随着生成继续，**更早的中间 token 会逐步从后续注意力计算所使用的 KV Cache 中移除。** ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

假设 Prefix 有 100 token、窗口为 4096，即使完整推理已经达到 10 万 token，**后续注意力计算最多只需保留约 4196 token 。** ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

**窗口填满后，单步成本不再随着完整推理链增长。**

〓Prefix 固定保留，最近的推理 token 随窗口向前移动

普通滑动窗口会让最开始的任务信息逐步移出窗口，**Prefix Sliding 则始终保留 Prefix 。** ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

团队最终采用 Continue PE。附录在 AIME25 上的实验显示，它与 Reset PE 表现相近，同时无需反复应用新的位置编码，可以直接复用已有缓存表示，计算更高效。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

团队还针对 NVIDIA Hopper 实现了专用 FlashAttention 内核。

对完全位于 Prefix 和滑动窗口之外的 tile 直接跳过，对与有效区域部分重叠的 tile 做元素级 Mask，从而减少无效加载和计算，**实际速度也接近普通滑动窗口的实现 。** ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

〓窗口填满后，Prefix Sliding 的生成吞吐趋于稳定

无需训练最高提速3倍，RL扩展到10万Token

在无需额外训练的设置下，作者以 Qwen3-1.7B 为主模型，在 AIME25、GPQA 和 MATH500 上进行评测。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

**Prefix Sliding 的优势来自同样思考时间内可以生成更多 token，并非单个 token 本身质量变高。** ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

以 4096 token 窗口为例，Prefix Sliding 在 AIME25、GPQA 和 MATH500 上与全注意力表现接近，但在长序列下吞吐明显更高，128K 时达到 5224 tok/s，而全注意力只有 448 tok/s。 ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]

〓相同思考时间下，Prefix Sl

## 深度分析

### 为什么注意力呈“两头高中间低”分布

论文对 Qwen3-1.7B 在 AIME25 推理轨迹上的注意力分析显示，注意力质量集中在两个极端：前缀端与最近生成端。这不是巧合，而是自回归生成机制的结构性结果：

- **前缀端**：系统指令、任务描述与工具信息定义了“当前在解什么题”，每一步生成都要重新对齐任务约束；最前面的少数 token（尤其前 4 个）还承担 attention sink 作用，softmax 归一化会把稳定的注意力质量沉淀在这里（参见 [[concepts/attention-mechanism|注意力机制]]）。
- **最近端**：模型正在续写自己的最新思路，紧邻上文决定下一个 token 的局部语法与逻辑衔接，注意力天然偏高。
- **中间段**：已完成的子计算（如 42 + 84 = 126）一旦得出结论，后续步骤只引用结论而非推导过程，对中间推导 token 的依赖随距离衰减。

“两头高、中间低”意味着丢弃中间 KV 造成的理论信息损失接近注意力权重的下界，这正是 Prefix Sliding “敢丢”的实证前提——若中间注意力不低，丢弃会直接表现为准确率下降。类似现象在 [[entities/attention-collapse-context-management|注意力坍缩与上下文管理]] 中也有平行讨论。

### Prefix Sliding 与 KV cache/滑动窗口类方法的本质区别

与 Last-k、Summary、普通滑动窗口逐项对比，差别不在“是否丢 token”，而在**对“什么必须被永久记住”的显式建模**：

- 普通滑动窗口把前缀与工作记忆混为一谈，最先滑出的恰恰是最不该丢的任务信息（含 attention sink）；
- Last-k 保留前缀但每步重复处理保留 token，计算被浪费；
- Summary 需要额外生成摘要并引入超参，真实样例中模型甚至会忽略摘要里的已有推导重新求解；
- Prefix Sliding 把上下文拆成“任务定义（前缀，永久保留）+ 工作记忆（最近 W 个 token，滑动）”，注意力集合恒为 Prefix ∪ 窗口，成本有界且信息不丢位。

KV cache 层面它不做压缩或量化，只是让低注意力的 key/value 不再参与后续计算，因此在 [[concepts/inference-optimization|推理优化]] 谱系中属于注意力稀疏化路线，与 [[entities/deepseek-v41-flash-kv-cache-compression-limit-agiyanxishe-2026-09-22|KV cache 压缩]] 类方法的作用层次不同。

### 无需重训练为何可行——机制层面解释

- **利用的是预训练模型的既有性质**。“两头高中间低”是预训练后模型自发形成的注意力模式，Prefix Sliding 只移除注意力权重接近零的 KV，对模型实际计算的函数扰动极小，不需要任何权重更新。
- **Continue PE 保持位置连续**。保留 token 不重新赋位置编码，RoPE 的相对距离关系不变，query-key 交互几何与全注意力一致，可直接复用已有缓存表示——这既是它比 Reset PE 高效的原因，也是“免训练即插即用”的关键。
- **attention sink 未被破坏**。前缀永久保留使承担 sink 作用的头部 token 始终在上下文中，softmax 注意力分配不会因 sink 消失而重新分布。

其风险边界同样来自机制：当模型真的在依赖窗口外 token 时（LiveCodeBench 中用数千 token 注释“离线思考”再回到代码），丢弃就会显现为质量损失，此时需要 16384 窗口才能追平全注意力。

### RL 后 10万+ token 推理轨迹意味着什么

同等内存预算下全注意力 rollout 上限为 8192 token，Prefix Sliding 将其推至 104K 且训练奖励更高。这不只是“训得更长”：

- **探索视野被拉长**。rollout 长度是策略探索空间的上限之一，10 万 token 轨迹意味着模型可以学到需要超长推导的策略，而不必在 8K 处截断。
- **截断反向传播在经验上够用**。跨多层理论感受野为 W×L，但受信息瓶颈限制实际有效感受野约 1.5×W；只对最后一个窗口算 loss、前面给 4 倍窗口上下文，KL 偏差已与 8 倍基本持平——与 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 中 credit assignment 本就局部化的直觉一致。
- **边界仍然清晰**。作者明确表示更大规模、更长推理链的表现未验证，7B 异步 RL 也只是小规模实验。

真正的信号是：长思考的瓶颈从“训练侧装不下”转为“上下文管理策略怎么定”，这与 [[entities/sliding-window-beats-linear-attention-microsoft-2026|滑动窗口优于线性注意力]] 的架构层结论互为印证。

## 实践启示

1. **先做注意力画像，再决定买卡还是改算法**。长 CoT 成本高企时，先检查任务上注意力是否同样“两头高”：若是，4096 窗口的 Prefix Sliding 在 AIME25/GPQA/MATH500 上与全注意力持平、128K 吞吐高约 11.7 倍，比换更大的卡便宜得多。
2. **把“任务定义”固化进前缀，把“工作记忆”留给窗口**。系统提示、题目描述、工具 schema 应稳定且靠前——这与 [[concepts/context-engineering|上下文工程]] 中最大化 prompt cache 命中是同一件事的两面，运行中途频繁改写前缀等于主动制造缓存失效。
3. **窗口大小按任务类型设定，不要套用默认值**。数学类 4096 足够；代码类至少 16384（注释式思考会跨窗口丢失）；平均生成长度仅约 2086 token 的短任务几乎没有收益——先用平均生成长度判断是否值得引入。
4. **长链 RL 采用 4 倍窗口的截断反向传播**。实证上 4 倍窗口的 KL 偏差已接近 8 倍，训练端只需最后约 8K token，不必为整条 10 万 token 轨迹做完整反向传播。
5. **多轮与 Agent 场景暂缓接入**。一次超大工具输出可能占满整个窗口；后续用户指令该并入前缀还是允许滑出，论文没有定论——当前把它当作单轮长思考技术更稳妥。
6. **Prefill 开销不会消失**。Prefix Sliding 不降低超长前缀在 prefill 阶段的 KV 开销，超长前缀场景仍需配合缓存复用或分块 prefill。

→ [[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍|原文存档]] ^[raw/articles/吴恩达参与斯坦福新作丢掉中间推理token长思考最高提速3倍.md]
