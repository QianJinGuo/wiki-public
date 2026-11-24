---
title: "Dust：Q Labs 用零阶优化在 Transformer 预训练上正面对标反向传播"
created: 2026-10-26
updated: 2026-11-09
type: entity
tags: [pretraining, training, zeroth-order, evolution-strategy, transformer, scaling, research]
sources: [raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026]
confidence: 0.8
provenance_state: extracted
---

# Dust：Q Labs 用零阶优化在 Transformer 预训练上正面对标反向传播

小型研究实验室 Q Labs 提出名为 **Dust** 的零阶（zeroth-order）优化方法，声称是第一个能在 Transformer 预训练任务上与反向传播正面竞争的零阶方法。它的意义不在于"现在就能替代 BP"，而在于把"训练必须处处可微"这条长期默认前提，重新变成一个可以被实验检验的工程选择。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

## 动机：反向传播是一种"低算力时代的好归纳偏置"

反向传播要求整个网络处处可微，这一约束把不可微模块、离散计算和外部程序都挡在训练回路之外。Q Labs 把这条论证接到 Sutton 的「苦涩的教训」上：随着可用算力增长，依赖精巧归纳偏置的方法最终会输给通用的、吃算力的暴力搜索——AlphaGo Zero 用人类棋谱上手更快，但纯自我对弈在充足算力下反超，是他们最常用的类比。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

他们进一步推测，梯度方法没有充分探索 loss landscape，这可能解释为什么当代神经网络需要海量数据才能泛化；而一个以搜索为基础的、更灵活的信用分配算法，或许是通往更好泛化的关键一步。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

## 方法：把扰动从权重空间搬到激活空间

传统进化策略（ES）需要为每个种群成员复制一份略有差异的权重并各自前向，种群越大成本越高；EGGROLL 用低秩扰动降本，但仍需为每个种群成员分配独立序列位置。Dust 换了一个落点：不修改权重，而是在每个线性层输出上加高斯噪声，且**按 token 独立生成扰动**——对 $y_t = Wx_t$ 变为 $y_t \rightarrow y_t + \sigma a_t$。这样每个 token 相当于一次"虚拟实验个体"，一次前向传播就能沿 token 维度并行评估数千个虚拟种群成员，这是它能把种群规模做大的关键。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

信用分配带衰减系数：$r_t$ 是第 t 个 token 的扰动奖励，$c_s$ 是第 s 个 token 的损失改善程度，$\gamma$ 控制未来 token 的贡献权重——某个 token 的扰动若让后续预测也更准，可获额外奖励。随后把多次采样的噪声按奖励加权平均，估计线性层输出端的误差信号，再与该层输入做外积，得到权重更新的梯度估计。与 BP 的差别在于：BP 用链式法则逐层回传，Dust 用前向计算中的扰动奖励估计更新方向。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

同时扰动多层多 token 会互相污染信用信号，因此 Dust 做了三项工程处理：（1）按层类型分组扰动并缓存未扰动的中间结果，减少重复前向；（2）注意力内部的 Q/K/V/Gate 不用 token 损失直接判优劣，而按扰动引起的注意力输出变化与已估计的输出误差信号分配奖励；（3）语言模型输出头直接在缓存的 logits 上扰动，只重算相关交叉熵损失，以低代价扩大种群。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

## 结果：采样效率、规模与"梯度方向收敛"

在 FineWeb 上训练一系列 Transformer，与 BP、EGGROLL 对比：随着种群规模扩大，Dust 训练效果持续改善，部分训练预算下甚至超过 BP；对 EGGROLL 的优势更明显——EGGROLL 用 16384 个种群成员仍不及只用 64 次扰动采样的 Dust，相差 256 倍，说明扰动从权重空间移到激活空间确实提升了采样效率。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

规模上，参数量从 200 万扩到 2.43 亿（约 120 倍）后 Dust 仍能有效训练；在较大种群下，3800 万与 2.43 亿参数模型的测试损失反而低于 200 万参数模型，且大模型从增加采样次数中获益更多。把 Dust 估计的梯度与 BP 梯度对比，采样规模越大两者方向越接近——即使训练已久，这一趋势仍成立。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

## 边界：复现后的冷静结论

作者自己讲得很直白：Dust 目前仍远比 BP 贵，要再便宜几个数量级才有资格谈"替代"；实验最大只到 2.43 亿参数、2000 万 token 的训练预算，部分"优于 BP"的结果来自曲线拟合推测而非真实训练。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

SOTAAZ Lab 的第三方复现提供了更冷静的角度：在论文设定的一轮 SGD 比较下 Dust 基本可复现，但与 BP 差距很小，并未显示出论文强调的明显优势；一旦给 BP 换上更合适的优化器，或允许其在同一数据上多训几轮，BP 就明显领先，且计算成本远低于 Dust。换言之，Dust 当前的价值是**证明零阶方法可用于 Transformer 预训练**，而不是替代反向传播。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md]

## 可迁移的工程启示

1. **把"可微性"当成本项而非公理**。当算力便宜到一定程度，训练回路的准入门槛从"可微"降到"可评估"——这为不可微模块、离散程序、外部工具进入预训练回路打开了理论上的门。
2. **扰动的落点决定采样效率**。同样的 ES 思想，落在权重空间是 16384 个成员打不过 64 次采样；落在激活空间并借 token 维度并行，种群规模才成为可调旋钮。选择"在哪里注入随机性"比"用什么搜索算法"更影响成本。
3. **信用分配是零阶方法的真瓶颈**。token 间存在因果依赖，噪声互相污染，所以需要按层分组、缓存中间结果、对注意力组件改用输出误差信号——这些工程细节决定了方法能否 scale，而不是论文里的主公式。
4. **"曲线拟合优于基线"必须单独标注**。第三方复现显示：换优化器/多训几轮后基线反超，说明跨方法比较的预算对齐方式能直接翻转结论；读这类论文先看比较协议，再看数字。

## 深度分析

### 为什么激活空间扰动能逼近 BP 梯度方向：同一个量的两种估计

Dust 与 BP 的「梯度方向收敛」不是巧合，而是同一个量在两种估计方式下的一致性。线性层的 BP 梯度在数学上就是「输出端误差信号 × 层输入」的外积；Dust 不走链式法则，而是给该层输出注入高斯噪声、按每个 token 的损失变化给噪声打分，再把噪声按奖励加权平均——这正是对同一误差信号的蒙特卡洛估计。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:93-101] 采样越多方差越低，估计方向自然向 BP 收敛，且训练已久仍成立，这是估计一致性的经验印证而非神秘现象。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:149-151] 真正非平凡的是落点选择：扰动落在紧邻损失的残差流上，信号路径短、每样本方差低——相当于用架构先验给随机采样做了预处理。

### 256 倍差距与规模不塌缩的真正含义

EGGROLL 用 16384 个种群成员仍不及 64 次采样的 Dust，说明梯度估计的方差由扰动落点决定：权重空间的扰动要穿过整段非线性下游才影响损失（信号弱、方差高），激活空间的扰动立即耦合到损失。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:137-139] 参数量扩约 120 倍而不塌缩、大模型从更多采样中获益更多，挑战了「维度灾难杀死零阶方法」的直觉。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:141-145] 但要诚实读取机制：真正随规模扩张的是 token 级虚拟种群——采样轴是序列长度而非参数维度^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:35]；而「部分预算下超过 BP」的关键证据来自曲线拟合外推，尚未在真实训练中兑现。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:155-157]

### 零阶方法何时真正有竞争力

全可微的监督预训练恰是 BP 协同设计积累最深的领域：第三方复现显示，一换更合适的优化器或多训几轮，BP 就明显领先，且计算成本远低于 Dust。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:41] 零阶方法的主场在梯度不存在之处——奖励模型或可验证器打分的 RL 微调（对 reward 无法反传）、黑箱模拟器与硬件在环、离散或程序化模块。Dust 的信用分配——按扰动奖励加权、γ 衰减让后续 token 的改善也记功^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:87-91]——与 [[concepts/grpo-policy-optimization-2026|GRPO]] 一脉的「采样、按优势打分、加权平均」同属一个估计器家族，只是「动作」的角色换成了激活扰动。这条研究线的现实终点更可能出现在 reward-only 的后训练回路，而非预训练的替代。

### 计算经济学：冷水与热度都需折价

复现的冷水是预算对齐方式直接翻转结论；社区的热度同样要折价解读：Noise speedrun 十天内把训练用时从约 1.76 小时压到 1.6 秒（约 4000 倍），但这是达到规定验证损失（约 4.2）的用时而非训练到收敛，比较起点是早期 Dust+Adam 而非 BP，且作者自己承认部分加速技巧无法扩展到更大规模。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:177-181] 被两边都忽略的变量是内存形态：BP 需要为反向传播存储中间激活，Dust 的前向-only 回路配合缓存复用（论文已在注意力组件与输出头上这样做^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:113-117]），在显存受限或训练/推理共用硬件的场景可能有不同的成本曲线——这是可检验的猜想，不是结论。

### 什么会证伪这条路

三个可观察的失败信号：(1) 梯度方向相似度在 1B+ 参数规模处平台化、不再随采样规模上升——「方向收敛」的规模叙事断裂；(2) γ 衰减信用分配无法处理长上下文——早期 token 的噪声污染数千个下游 token 的归因——虚拟种群被锁死在短序列；(3) 匹配预算、匹配优化器的对比下 BP 在更大规模上一致胜出——「苦涩的教训」式论证失去经验锚点，Dust 退回为「可微性是一种可选项」的存在性证明。^[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026.md:45-47]

## 关联

- [[concepts/llm-pretraining-vs-sft|LLM 预训练与 SFT]] — 本文讨论的是预训练阶段的优化器替代路径，而不是后训练
- [[concepts/scaling-laws|Scaling Laws]] — Dust 的核心论据是"算力充裕时通用搜索胜过精巧偏置"，与 scaling 的取舍同源
- [[entities/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|领域数据重复与预训练]] — 同属预训练数据/预算侧的实证研究

→ [[raw/articles/dust-zeroth-order-pretraining-transformer-qlabs-2026|原文存档]]
