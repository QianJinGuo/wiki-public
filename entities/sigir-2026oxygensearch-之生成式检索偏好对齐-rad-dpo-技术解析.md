---
title: "SIGIR 2026: RAD-DPO 生成式检索偏好对齐"
created: 2026-07-24
updated: 2026-08-01
type: entity
tags: [ai, agent, sigir, retrieval, generative-retrieval, dpo, preference-optimization, e-commerce, jd-tech]
sources: [raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析]
confidence: 0.92
score: 72
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# SIGIR 2026: RAD-DPO 生成式检索偏好对齐

> **v×c score**: 72 | stars=4
> **来源**: https://mp.weixin.qq.com/s/iPaRBCnSPk5kq0h9R6kM5Q
> **发布**: 京东技术 (2026-07-16)

## 摘要

RAD-DPO（Robust Adaptive Denoising Direct Preference Optimization）是京东 OxygenSearch 团队提出的面向生成式检索（Generative Retrieval, GR）中 SID（Semantic ID）结构的偏好对齐方案，被 SIGIR 2026 接收。该方案系统性地解决了标准 DPO 在电商生成式检索场景中的三个核心问题：公共前缀被误伤（Shared Prefixes）、隐式反馈噪声敏感（Pseudo-negatives）、多正例概率挤压（Squeezing Effect）。RAD-DPO 通过三个协同模块——MLGC（Multi-Label Global Contrast, session 级多标签全局对比）、TLGD（Token-Level Gradient Detachment, token 级梯度截断保护公共前缀）、RDRW（Reward-Dependent Reweighting, 动态奖励加权降低伪负例惩罚）——将 DPO 从"简单 pair-wise 序列级偏好优化"提升为"低成本、多样本、token 级、噪声鲁棒、prefix-aware 的结构化偏好学习范式"。配合 Label Packing 高效训练方案，RAD-DPO 在 3000 万数据基准上对比 SFT 和其他 DPO 变体取得全面领先，召回和 MRR 指标在 0.6B 到 8B 模型上持续提升。^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]


## 核心要点

- **三个核心问题**：共享前缀误伤（正负样本共享的类目前缀被 DPO 无差别打压）、伪负例噪声（曝光未点击≠不相关）、多正例概率挤压（打压负样本同时压缩长尾正样本概率）
- **MLGC（多标签全局对比）**：将偏好学习从单个 pair 扩展到 session 级，同时提升所有正样本、对所有负样本做 list-wise 对比
- **TLGD（Token 级梯度截断）**：只切断负样本公共前缀上的梯度，只在正负样本分叉点之后施加偏好惩罚
- **RDRW（动态奖励加权）**：基于模型自身 hidden state 相似度判断正负样本相似性，对疑似伪负例做软惩罚
- **Label Packing 工程优化**：将所有正负候选拼接到同一序列，使用 Block-diagonal Attention Mask，训练吞吐提升 300%、显存降低 50%
- **实验结果**：在 3000 万数据集上全面领先 SFT 和竞品 DPO 变体，0.6B→8B 模型领先优势持续扩大

## 深度分析

### 为什么标准 DPO 在生成式检索中失败

理解 RAD-DPO 的价值，需要先理解生成式检索（GR）与传统 NLP 任务的本质差异。在 GR 中，商品被编码为结构化的 Semantic ID（SID），通常由层次化聚类（如 RQ-VAE）构建——前缀 token 表示粗粒度类目（如"电子产品 > 手机"），后缀 token 区分具体商品 ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:22-26]。

标准 DPO 的 loss 函数对整条序列无差别操作：正样本整条序列被提升，负样本整条序列被打压。但当正负样本共享类目前缀时（比如 iPhone 15 和 iPhone 15 Pro 都是"电子产品 > 手机 > 苹果"），打压负样本的同时也在打压正确的类目前缀——这正是"公共前缀误伤"问题的根源 ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:30-30]。

此外，电商的"未点击"不等同于"不相关"。用户没点击可能是位置偏见、图片问题或已通过其他方式购买。将这种"伪负例"作为强负样本使用，会扭曲模型对商品语义的理解。这与 [[entities/ebay-generative-retrieval-rq-vae-semantic-id-2026-06-30|eBay 生成式检索实践]] 中遇到的噪声问题一致。^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]


### TLGD：结构化输出的梯度保护创新

TLGD（Token-Level Gradient Detachment）是 RAD-DPO 最具原创性的贡献。它的核心思路可以概括为：**"把'整条负样本序列打压'改造成'只在分叉点之后打压'"** ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:93-127]。

具体实现：前向计算时，负样本公共前缀仍然参与 likelihood 计算（保证 prefix 的表示更新）；反向传播时，通过 Stop-Gradient 操作切断公共前缀上的梯度，只让真正区分正负样本的差异后缀参与偏好优化。这个精妙的梯度操作确保了 SID 的层级语义结构不会被 DPO 破坏——粗粒度语义（类目、品牌）受到保护，细粒度语义（具体商品差异）才参与偏好学习。^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]


这个思路与 [[entities/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|RL 对齐算法演进]] 中的"token 级细粒度优化"趋势一致，但 TLGD 的技术路线更直接——不是通过复杂的 reward 设计间接保护前缀，而是从梯度传播机制层面直接切断不必要的梯度。^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]


### RDRW：基于模型表示的自适应降噪

RDRW 解决的是隐式反馈噪声问题。它的方法论比 TLGD 更难设计：如何判断一个负样本是"真负例"还是"伪负例"？^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]


RAD-DPO 的解法很巧妙：用模型自身生成 EOS token 时的最后一层 hidden state 作为 SID 的序列级表示，计算正负样本之间的余弦相似度 ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:130-138]。如果正负样本在表示空间中非常相似，说明负样本很可能是"伪负例"（语义相关但用户没点击），惩罚应当减轻。

为了避免使用固定阈值（不适应性），论文设计了一个统计 warm-up 阶段：先缓存 4096 个 pair 的相似度，构造基准分布，计算分位点，然后根据分位点动态调整惩罚权重 ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:140-155]。相似度低的负样本保留完整惩罚（更大可能是真负例），相似度高的负样本惩罚降到 0.5（更可能是伪负例）。

这种"用模型当前表征判断样本可信度"的思路，本质上是把偏好学习从"基于固定标签的训练"升级为"基于模型认知状态的自适应训练"。^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]


### Label Packing：将工程效率提升 300% 的关键创新

RAD-DPO 的工程实现面临一个独特挑战：一个 session 级训练样本包含 1 个 prompt + N 个正样本 + M 个负样本。如果复用标准 DPO 框架，prompt 会被重复编码 1+(N+M) 次 ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:190-203]。

在 NLP 任务中这个问题不明显（response 较长，prompt 占比不高），但 GR 场景中 SID 通常只有 3-7 个 token，prompt 可能上百 token——计算量主要在 prompt 上。^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]


Label Packing 的解决方案是将所有 candidate 拼接到同一条 unified sequence，prompt 只计算一次。通过 Block-diagonal Attention Mask 确保不同 candidate 之间互不可见，同时对齐 Position ID 防止模型通过位置差异区分样本 ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:204-228]。这一工程创新使得 RAD-DPO 的训练吞吐与 SFT 处于同一水平，对比 GRPO 等强化学习方案有数量级的优势 —— 这是论文中提到的"全链路算法优化 + 工程效率优化"的双轮驱动策略。

这对于 [[entities/agent-harness-production|生产级 Agent Harness]] 的设计原则——"高效训练是大规模落地的必要条件"——是一个有力的佐证。^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]


### 与 RL 对齐方法论的定位

RAD-DPO 选择 DPO 路线而非 GRPO 或 PPO，有明确的技术考量：DPO 不依赖额外的 Reward Model，实现简单且样本获取成本低 ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:30-30]。但 GRPO（如 OneSearch-V2 的 TPMA-GRPO）对 SID 前缀 token 也有建模，两种路线的差异在于：

- DPO 路线（RAD-DPO）：通过梯度操作保护前缀，保持 DPO 的简洁性
- GRPO 路线：通过分组奖励建模前缀，但需要多个采样的 reward 估算

两种路线各有利弊——DPO 路线训练效率更高，但 reward 信号来自身而非外部评判；GRPO 路线更灵活但训练成本更高。RAD-DPO 的 reference-free 设计（与 SimPO 一致）进一步简化了训练流程 ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md:82-84]。

## 实践启示

1. **结构化输出场景中的 DPO 需要 token 级保护**：如果模型输出具有层级结构（如 SID、代码 AST、JSON schema），标准 DPO 会误伤公共前缀。应该在 token 级别控制梯度传播，保护底层结构不被偏好优化破坏。

2. **隐式反馈信号的噪声处理应基于模型自身表示**：电商"点击/未点击"等隐式信号天然有噪声。RDRW 的方法论——用模型当前表征判断样本可信度——比固定规则过滤更适应训练过程中的表示空间变化。

3. **Session 级对比优于 Pair-wise 对比**：在 multi-label 场景中（一个 query 对应多个正样本），pair-wise 对比会导致长尾正样本被挤压。使用 session 级全局对比，所有正样本都被提升，模型看到的不是"单个正负 pair"而是"正样本集合 vs 负样本集合的整体关系"。

4. **Label Packing 是多候选训练场景的标配工程优化**：任何需要在一个 prompt 下对多个候选 response 计算 log-probability 的训练场景（DPO、对比学习、候选重排），都可通过 Label Packing + Block-diagonal Attention Mask 大幅提升吞吐。

5. **Reference-free 设计降低训练成本**：去掉冻结的 reference model（走 SimPO 路线）在 GR 场景中有效，显存降低近 50%。如果你的场景不需要 KL 散度约束，可以考虑这条更轻量的路线。

6. **从 error bar 中提炼真正有用的 insight**：论文在 0.6B、2B、8B 三个规模上做了实验。模型规模越大，RAD-DPO 领先优势越明显——这说明结构化偏好学习在大模型中释放的潜力大于小模型，可能因为大模型具有更好的 SID 结构表示能力。

## 相关实体

- [[entities/ebay-generative-retrieval-rq-vae-semantic-id-2026-06-30|eBay 生成式检索实践]]
- [[entities/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|RL 对齐算法演进]]
- Agentic 搜索检索
- RLHF/DPO/GRPO 对齐
- [[entities/dream-dense-retrieval-autoregressive-modeling-challengehub-2026|DREAM 密集检索]]
- [[entities/agent-harness-production|生产级 Agent Harness]]

→ [[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析|原文存档]] ^[raw/articles/sigir-2026oxygensearch-之生成式检索偏好对齐-rad-dpo-技术解析.md]
