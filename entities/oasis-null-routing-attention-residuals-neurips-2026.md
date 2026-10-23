---
title: "OASIS：给注意力加显式弃权选项的双层 null 路由（NeurIPS 2026）"
created: 2026-10-23
updated: 2026-10-24
type: entity
tags: [attention-mechanism, transformer, quantization, long-context, softmax, neurips-2026, research]
sources: [raw/articles/oasis-null-routing-attention-residuals-neurips-2026]
confidence: 0.8
provenance_state: extracted
---

# OASIS：给注意力加显式弃权选项的双层 null 路由（NeurIPS 2026）

西北大学、伊利诺伊理工学院、罗格斯大学等机构的 **OASIS** 被 NeurIPS 2026 接收：它针对 Attention Residuals（AttnResidual）结构里"两层 softmax 都强制权重和为 1、没有'这次不选'选项"的缺陷，在 token 层与 depth 层同时引入显式 null 路由，并用 token 层的 null 概率协调 depth 层的分支选择，从而显著缓解 attention sink、激活异常值和低比特量化退化。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

## 背景：AttnResidual 的双重归一化缺口

普通 Transformer 的 self-attention 决定一个 token 读哪些 token；AttnResidual 在此之上增加沿网络深度的选择——同一 token 在当前子层应该使用哪些历史子层输出（depth query 是当前子层的可训练向量，key 为归一化后的历史输出，value 为历史输出本身），把固定权重的残差累加换成可学习加权混合。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

问题在于两处 softmax 都要求候选权重合计为 1：一次读取若只需很小贡献，模型仍要把全部概率分给真实 token；历史分支若没提供多少有效信息，depth 层仍要在真实分支间分配全部质量。论文的注意力诊断显示，这类模型在序列起始 token 上的集中条带比原始 Transformer 更强，并随深度持续出现（attention sink），同时激活异常值显著增大——异常值扩大了数值范围，让有限量化刻度更难兼顾大多数普通数值。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

## 方法：Softmax₁ + token-to-depth null coupling

**两处 Softmax₁**：为真实候选增加一个固定 logit = 0 的 null 候选，真实候选权重合计可小于 1，剩余质量进入 null，从而减小真实内容总贡献；token 层与 depth 层都用 Softmax₁ 的版本称为 **AoS**。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

**跨层协调（coupling）**：两层各自有 null 仍不解决"一个分支的 attention heads 已大量选择 null、depth 路由却继续给它高权重"的不一致。OASIS 在每个 token 位置对一个源分支内各 head 的 null 概率取平均得到分支统计量 ψᵢ，与候选变换分支的平均值比较，用差值调整 depth logit：null 概率高于平均则降低该分支 depth logit，低于平均则提高；初始 embedding 分支没有 token attention 的 null 统计，其 logit 保持不变。调整后的 logits 再经 depth Softmax₁ 得到最终混合权重。OASIS 保留原训练目标、只改路由机制，通过继续训练适配后再做量化评测。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

## 实验结果

在 LLaMA-3.2-1B、Qwen3-0.6B、Phi-4 上用 BookCorpus 与 Wiki40B/en 继续训练，与标准 AttnResidual、OutEffHop、裁剪、门控等方案比较，三个独立随机种子。相对 Vanilla AttnResidual：平均降低激活峰度 95.90%、最大激活幅度 81.75%，降低 W8A8 困惑度 82.00%，W4A4 GSM8K Pass@1 平均相对提升 42.11%（相对比例）。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

组件消融（Qwen3-0.6B，W4A4 GSM8K）显示：只加 token null、只加 depth null、两处独立 null 逐步改善，而 coupling 在两处独立 null 之上再提高 4.67 个百分点；完整 OASIS 相对 Vanilla AttnResidual 提高 13.21 个百分点——直接支持跨层协调的必要性。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

低比特能力保留的对比最能说明问题：Qwen3-0.6B 的 FP16 GSM8K 上两者接近（OASIS 62.68% vs Vanilla 62.58%），但量化到 W4A4 后 OASIS 保留 58.88%，Vanilla 跌到 45.67%；ThePile W8A8 PPL 从 44.52 降至 11.58，W4A4 下 MMLU/TriviaQA/MATH500 分别从 33.21%/12.09%/15.41% 升到 41.00%/16.14%/21.51%。长上下文（12K 训练序列）上，OASIS 相对 Vanilla AttnResidual 与原始 backbone 的差距，在两个模型上分别恢复 77.3%/80.3% 的 RULER 损失与 89.1%/91.8% 的 LongBench 损失。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

注意力统计是全局层面的证据（覆盖所有层、head、query 位置与评测样本）：Qwen3-0.6B 分配给 BOS 的平均质量从 0.4990 降到 0.0013，LLaMA-3.2-1B 从 0.5630 降到 0.0099，说明 sink 缓解不是少数可视化样例的错觉。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

## 判读要点

这篇工作的价值不在"又一种量化技巧"，而在指出一个结构性缺口：**增强表达能力的可学习路由若缺少显式弃权选项，会把容量压力转移到数值分布上**（sink 与离群激活），而后者恰好是低比特量化最脆弱的地方。因此"表达能力—信息路由—低比特鲁棒性"是一条因果链，null 路由是同时作用于三者的同一机制。若要在自研 harness 或自训模型上复用，可迁移的是"两层归一化各给 null + 用细粒度统计量协调粗粒度选择"这一模式，而非具体超参。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md]

## 深度分析

### 双重 softmax 之和为 1：接口缺了"零档位"

softmax 把 logits 映到开单纯形，隐含一条强先验：候选集合总是"配得上"全部概率质量。这条先验在 AttnResidual 里变成结构性错误，因为两层归一化是**复合**的——token 层被迫对无关上下文满额加权，depth 层再被迫在这些已被污染的历史表示之间满额混合，误差相乘而非相加。

论文的机制解释是条件性的但很关键：当值向量满足一定分离条件时，普通 softmax 若要输出足够小的结果，**只能**把大量概率堆到低范数候选上。于是"想读得少一点"这个正常诉求被实现成"倾泻到 BOS"——sink 不是模型学到的语义偏好，而是归一化约束逼出的数值替代策略；峰度 91.71→3.02 量的就是这替代策略的代价。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md] 正确的定位因此是**接口缺陷**，而非校准或精度问题，量化的脆性只是症状最先暴露处。

### null coupling：用便宜信号校准昂贵决策

两处 Softmax₁ 各自解决本地问题，却留下层级不一致：某源分支的 heads 已大面积弃权，depth 路由仍可能因粗粒度判断给它高权重。coupling 把**细粒度、高密度统计量**注入**粗粒度、低密度决策**——每个 token 位置、每个分支内对所有 head 的 null 概率取均值得 ψᵢ，与候选分支均值比较，差值以加性方式调整 depth logit。

三个细节解释了它为何免调参：加性注入保留 depth Softmax₁ 的其余结构，原有路由能力不被破坏；用相对均值而非绝对阈值，对层深、head 数、null 整体漂移不变，等于隐式标准化；初始 embedding 分支无 token attention 故豁免，只在"有证据的分支"上施加校正。

代价是耦合强度被 head 平均抹平：若少数 head 承载全部有效信息、其余大量弃权，均值会误判该分支为低价值而压掉它——在 head 功能高度分化的模型上这一失效模式会放大。

### 量化结果的真实分量

最强证据不是大数字，而是 **FP16 几乎不动、低比特大幅改善**的对照：Qwen3-0.6B 全精度 GSM8K 62.68% vs 62.58%，W4A4 后 58.88% vs 45.67%（保留率 93.9% 对 73.0%）。这证明被消掉的异常激活**本来就不是能力载体**，而是路由缺口的副产品；若峰度下降伴随全精度下滑，那只是把模型变钝。BOS 质量从 0.4990/0.5630 降至 0.0013/0.0099 且覆盖全部层、head、query 与样本，排除 cherry-pick。相反，W8A8 的 PPL 从 44.52 降到 11.58 幅度极大，反过来说明 Vanilla 在 8bit 下就已严重受损，退化并非 4bit 专属。^[raw/articles/oasis-null-routing-attention-residuals-neurips-2026.md] 长上下文侧的 77.3%/80.3% 与 89.1%/91.8% 是**相对差距的百分比**，仍留 20% 上下缺口，不宜读成"已解决"。

### 可能在哪儿失效

一是依赖继续训练适配，无法 post-hoc 套到 frozen checkpoint 或 API 模型；二是 depth null 的语义是"衰减真实分支贡献"而非"保留上一层 hidden state"，极端利用下可能退化为信息瓶颈，而论文未报告 null 利用率随深度的分布；三是只覆盖 0.6B–1B 与 Phi-4，超大模型的 massive activations 集中于少数固定 channel，sink 结构不同，收益未必线性延续；四是 W4A4 采用 OmniQuant，与 OASIS 的交互未拆开，纯 RTN/GPTQ 下是否成立无消融；五是 Softmax₁ 改了分母，标准 fused attention kernel 需改动，且 depth 路由要保留全部历史子层输出，长上下文下显存随深度线性增长。

## 实践启示

1. **量化前先做结构诊断**：逐层测激活峰度与最大幅度；若峰度远高于个位数、首 token 注意力质量超 0.3，问题多半是路由缺零档位，此时换 GPTQ/AWQ 只是换止痛药。对照 [[entities/quantization-techniques|量化技术]]。
2. **最小改动优先**：给每一处 softmax 加 logit=0 的 null 项，成本仅分母多一个 1，不加参数、不改结构。
3. **跨粒度路由用"便宜信号校准贵决策"**：把细粒度层的 null 率、熵、激活幅度以加性方式注入粗粒度选择，配相对均值而非绝对阈值。同一模式可迁移到 MoE 门控（见 [[concepts/moe-mixture-of-experts-2025|混合专家]]）。
4. **head 聚合按结构选**：均值会被大量弃权的 head 稀释；GQA/MQA 或 head 功能分化明显时改用 top-k 均值或熵加权。
5. **量化报告必须给三样**：FP16 基线、低比特绝对分、**保留率**。只报绝对分无法区分"模型本弱"与"量化损伤大"。
6. **结构改动放进继续训练，并与 kernel 同设计**：硬套冻结权重通常只换来更差的注意力分布；先确认 fused attention 支持改过的分母、并预算历史子层输出显存，否则纸面收益会被部署吞吐吃掉。

## 关联

- 结构基础见 [[concepts/attention-mechanism|注意力机制]] 与 [[concepts/transformer-architecture|Transformer 架构]]。
- 低比特部署侧的同类问题可对照 [[concepts/inference-optimization|推理优化]] 与 [[entities/bonsai-image-4b-quantization|Bonsai 4B 量化]]。
- 深度维路由的对照实验见 [[entities/attention-only-transformers-controlled-study-sans-arxiv-2607-18363|Attention-only Transformer 对照研究]]；sink 与上下文的关系见 [[concepts/lost-in-the-middle|Lost in the Middle]]。
- 论文：arXiv 2605.17887；代码 github.com/robinzixuan/OASIS。

→ [[raw/articles/oasis-null-routing-attention-residuals-neurips-2026|原文存档]]
