---
title: "LLM 记忆容量：ICML 2026 获奖论文揭示每个参数约 3.6 bit"
created: 2026-07-08
updated: 2026-11-13
type: entity
tags: [llm, memorization, scaling-law, icml-2026, capacity, generalization, double-descent, membership-inference, data-privacy]
sources: [raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LLM 记忆容量：ICML 2026 获奖论文揭示每个参数约 3.6 bit

> ICML 2026 杰出论文荣誉提名（Outstanding Paper Honorable Mention）— Meta、DeepMind、Cornell、NVIDIA 联合研究，回答了一个基础问题：语言模型的参数中究竟存储了多少训练数据信息？核心发现：GPT 类 Transformer 的经验记忆容量约为 **每个参数 3.5–3.6 bit**。^[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026.md]

## 方法：信息压缩视角

传统研究从训练数据抽取（extraction）和成员推断（membership inference）入手，但难以区分样本级记忆与泛化能力。该论文转向**信息压缩视角**：^[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026.md]

- 如果有模型作为参考，一个样本可以被压缩成更短编码，就说明模型提供了关于这个样本的部分信息
- 样本编码长度减少多少，对应模型提供了多少样本信息
- 使用算术编码 + 模型似然来近似压缩长度，将 Kolmogorov 复杂度转为可操作估计

这一方法将**非预期记忆**（模型对具体样本保留的信息）与**泛化**（从整体分布中学到的通用规律）分开测量。^[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026.md]


## 关键实验发现

| 发现 | 细节 |
|------|------|
| **容量线性缩放** | 容量与参数量之间近似线性：$\text{Capacity} \approx \alpha \cdot N_{\text{params}}$ |
| **bfloat16 容量** | 平均约 3.51 bit/参数 |
| **fp32 容量** | 平均约 3.83 bit/参数（位宽增加但容量仅小幅上升） |
| **平台效应** | 非预期记忆先随数据量上升，达到容量上限后进入平台 |
| **双下降** | 数据-容量比接近 1 时测试损失先升后降，模型从记忆转向泛化 |

^[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026.md]

实验在 500K 到 1.5B 参数的 GPT 类 Transformer 上完成，使用合成随机数据排除泛化干扰。^[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026.md]


## 记忆 vs 泛化的关系

**核心理念**：在真实文本（FineWeb）实验中，当训练数据规模超过模型容量时，非预期记忆下降，**泛化开始占据更大比例**。模型迫于容量限制在样本之间共享信息以节省容量，进而表现出更强泛化。^[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026.md]

> 背不下去，才会泛化。

## 成员推断与数据安全

论文给出成员推断（Membership Inference）的经验缩放律：^[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026.md]

- 关键变量：**Capacity(θ)/∣D∣** — 模型容量与数据集的比值
- 模型容量越大，平均样本越容易被成员推断区分
- 训练数据越大，单个样本被识别越难（∣D∣ 足够大时 F1 → 0.5）
- 成员推断通常比直接抽取更容易，即使抽取率为 0 仍可能有效

**稀有 token 风险更高**：TF-IDF 分析显示，高 TF-IDF 样本（日文、中文、希伯来文等非英语文本）更容易被记住。稀有、重复、异常的样本需要单独关注数据隐私风险。^[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026.md]


## 局限

- 仅测试了特定 GPT 类架构和训练设置
- 梯度下降不保证全局最优，因此测量更接近容量**下界**而非上界
- 样本细节、通用规律和训练过程统计痕迹在参数中混合，难以完全分离

## 与现有知识体系的关系

- 与 Scaling Laws 直接相关 — 模型容量（而非参数量）可能是预测泛化行为的更根本变量
- 为 Dario Amodei 的 scaling 论述 提供微观理论基础
- 与 [[entities/loop-engineering-feedback-control-system|Loop Engineering]] 中评估器设计的隐含联系：如果模型记忆容量有限且受数据-容量比驱动，那么"模型是否记住了测试数据"就不是一个 binary 问题，而是 continuum 问题
- 在双下降现象上的发现补充了 ML 理论的核心认知

## 深度分析

**1. "3.6 bit/参数"衡量的是训练实际写入的信息，而非数值存储能力。** bfloat16 每个参数有 16 个物理比特，实测容量却只有约 3.51 bit；fp32 位宽翻倍到 32 bit，容量仅升至约 3.83 bit（约 +9%）。参数的可占用信息量远低于其数值位宽，且精度翻倍只带来微小提升——瓶颈不在编码格式，而在梯度下降实际找到的解只覆盖了参数空间表达能力的一小部分。因此 3.6 bit 应读作经验容量的**下界**估计，不宜当作所有架构的物理极限。

**2. Scaling Law 的解释变量可以从参数量 N 替换为容量 C。** 若 C ≈ α·N 在受控设定下近似线性，则常规缩放律可整体平移为以容量为自变量的形式。这不只是记号游戏：不同精度、架构、优化器下相同的 N 未必对应相同的 C，容量才是跨配置可比的"有效自由度"计数。这也给 [[concepts/scaling-laws|Scaling Laws]] 在架构更迭时频繁失灵的现象提供了一个候选解释——它们隐含假设了容量与参数量成正比。结合 [[entities/model-size-scaling-in-2023-2031|Model Size Scaling in 2023-2031]] 的历史外推，未来参数量增长对应的"可存信息量"增长应以约 3.6 bit/参数为锚点折算。

**3. 双下降获得机制性解释：容量耗尽迫使信息共享。** 论文把双下降精确锚定在数据-容量比 C/|D| ≈ 1 附近：|D| < C 时模型逐样本记忆，测试损失随数据增加不降反升（记住了噪声细节）；|D| > C 后参数装不下逐条样本，模型被迫在样本间共享统计结构，测试损失二次下降。在此框架下泛化不是"更好的记忆"，而是容量不足的**必然产物**——"背不下去，才会泛化"由此从格言升级为可检验的机制命题。

**4. 泄露风险的正确度量是比值而非绝对量。** 成员推断有效性由 Capacity(θ)/|D| 驱动：同一个十亿参数模型，训练在 1B token 与 100B token 上泄露风险截然不同；|D| 足够大时 F1 趋近 0.5（随机猜测）。更关键的是两条攻击通道不对称——抽取率降到 0 不代表成员推断失效，后者利用似然信号而非输出复现。这意味着隐私防御与 [[entities/selective-unlearning-amazon-nova-2026|Selective Unlearning]] 类遗忘工作的效果检验，都应放在容量框架下用压缩增益量化，而不是只看输出端能否复现原文。

## 实践启示

- **隐私审计先算 C/|D| 再定级。** 用 参数量 × 3.6 bit 估算模型容量，与训练 token 数求比值；比值接近或超过 1 的模型-数据组合列为高风险，优先做成员推断红队与重复样本检测，而不是对全部模型无差别扫描。
- **稀有样本单独脱敏。** TF-IDF 高的样本（非英语语料、含 PII、重复异常模式）被判为"已记忆"的比例显著更高；在数据配比阶段对稀有 token 密集的文档做降采样或加噪，比全库统一处理性价比更高。
- **评测集污染按 continuum 监控。** "是否记住了测试数据"取决于训练时的数据-容量比：小模型 + 小评测语料组合最危险。可用论文的算术编码压缩增益方法量化单条评测样本被记忆的程度，替代 binary 的命中/不命中判断。
- **预算固定时优先扩数据而非加轮次。** 容量由参数量决定，加 epoch 只会提高记忆占用、不新增容量；当数据规模跨过 C 之后，继续堆数据的收益才从记忆转向泛化——小模型场景尤其应把预算花在数据量上。

---

→ [[raw/articles/llm-memorization-capacity-36-bit-per-parameter-icml2026|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

