---
title: "ICML 2026: GPS — 小模型「指挥」大模型RL后训练，最高减少69% Rollout成本"
created: 2026-07-11
updated: 2026-11-26
type: entity
tags: [llm, rl, post-training, prompt-selection, icml, tsinghua, tencent, gps, reinforcement-learning]
sources: [raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# ICML 2026: GPS — 小模型「指挥」大模型RL后训练，最高减少69% Rollout成本

GPS（Generalizable Predictive Prompt Selection）是清华大学与腾讯在 ICML 2026 上提出的方法，通过训练一个小型 Prompt Predictive Model（PPM）预测 prompt 难度，从而在 RL 后训练中减少无效 rollout，降低训练成本。^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md]

## 核心问题

RLVR（Reinforcement Learning with Verifiable Rewards）已经成为提升大模型推理能力的重要后训练路线，但存在一个很现实的问题：每个训练 step 都需要大量 rollout，而这些 rollout 往往需要反复调用大模型生成长答案，计算和显存开销都很大。^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md]

并非所有 prompt 都同样有训练价值——太简单的 prompt 几乎每次都答对，梯度信号弱；太难的 prompt 几乎每次都答错，同样难以提供有效学习信号。真正有价值的往往是中等难度的 prompt。^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md]

## 方法：GPS

GPS 的做法是：先训练一个小型、可泛化的 Prompt Predictive Model（PPM），让它预测不同 prompt 在当前模型下的难度；再根据难度和 batch 多样性选择训练样本，从而减少无效 rollout。^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md]

与已有的方法相比：
- **基于真实评估的方法**（如 Dynamic Sampling/DS）：效果好但额外评估本身非常昂贵
- **基于预测的方法**（如 MoPPS、GRESO）：把每个 prompt 当成独立对象建模，难以跟上模型训练的持续变化
- **GPS**：使用轻量 PPM，利用整个优化历史在 prompt 之间共享信息，用更好的 batch 选择策略提升 RL 后训练效率^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md]

## 实验结果

GPS 在数学推理和逻辑推理任务上带来了明显收益^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md]：

- **训练加速**：相比 Uniform 随机采样，训练步数加速 1.4×–2.0×
- **成本降低**：相比 DS Oracle 基线，最多减少 69% rollout 成本，训练时间减少 28%–47%
- **测试时计算复用**：小预测模型可复用到测试时计算分配中，固定预算下最高提升 3.2%，或性能不下降时节省 36.4% 推理计算

## 关键信息

- 论文：Small Generalizable Prompt Predictive Models Can Steer Efficient RL Post-Training of Large Reasoning Models
- 链接：https://arxiv.org/abs/2602.01970
- 代码：https://github.com/thu-rllab/GPS
- 机构：清华大学自动化系季向阳教授团队 + 腾讯混元

→ [[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent|原文存档]]

---
## 深度分析

### 从「单题记账」到共享历史：difficulty context 的动机

MoPPS、GRESO 等预测类方法把每个 prompt 当独立对象建模：某题被采样得多，难度估计才准；采样得少，信息就很稀疏。更麻烦的是，大模型在训练中不断变化，同一 prompt 的难度也随时间漂移，独立记账很难跟上这种动态^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:113]。GPS 为此引入全局潜变量 difficulty context——一个压缩后的「当前训练状态」，第 t 步由完整优化历史生成，再结合候选 prompt 预测其成功率^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:165-170]。于是即便某个 prompt 很少被采样，也能借相似题的历史表现来估计难度；论文还给出理论分析：当历史信息确实包含额外预测信号时，使用完整优化历史的预测均方误差低于只用 prompt-specific 历史^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:175-180]。

### batch utility：难度收益与 history-anchored diversity

只预测难度还不够，如何选出一整个 batch 同样关键。GPS 用统一的 batch utility 合并两个目标：难度收益偏好成功率接近 0.5 的 prompt——在二值奖励场景中，这类样本更容易产生非零奖励方差，也更可能给 GRPO 等算法提供有效梯度；多样性收益则要求 batch 内 prompt 之间有足够分散度，且与上一轮 batch 之间保持探索性，避免训练信号冗余^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:195-212]。论文把这套设计称为 history-anchored diversity：既要选「难度刚好」的题，也要让训练覆盖更多不同区域，避免模型在局部题型上反复打转^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:217]。

### 一个小模型吃两遍：训练侧采样与测试侧预算分配

PPM 最有意思的性质是跨阶段复用。训练侧，它让 GPS 免掉 DS 类方法「对候选 prompt 额外做真实 rollout」的昂贵环节——DS 在 step 维度是强基线，但 rollout 成本显著更高，GPS 相比 DS 最多减少 69% rollout，同时保持相当甚至更好的最终效果^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:108,343-348]。验证横跨两类任务：数学用 DeepScaler 40.3k 竞赛级问题训练、MATH500/OlympiadBench/AIME24 等评测，另以 MMLU-Pro、GPQA-diamond 测 OOD 泛化；逻辑用 Countdown-34 的 20k 子集训练、在更难的 CD4 上评测^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:264-269]。收益在两边一致：相比 Uniform，数学任务平均提升 1.6–1.9 点，逻辑任务平均提升 4.1–5.7 点；Qwen3-4B/8B 上 CD4 准确率 57.2/59.4 均高于 DS，训练时间 3.4h/5.0h 低于 DS 的 4.9h/6.9h^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:406-436]。测试侧，同一 PPM 给 Best-of-N 采样分配预算：容易题与近乎不可解的题少分配，「有挑战但仍可能解出来」的题多分配^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:232-242]；且在多数未见测试集上，预测难度与经验成功率仍保持统计显著相关，固定预算下最高提升 3.2%，或性能无损时最多节省 36.4% 计算预算^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:446-461]。

### 预测质量、消融与算法兼容性

PPM 收敛很快：只需很少训练步数就能建立有效的 Spearman 排序相关，并随优化历史积累继续提升^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:299-304]。算法兼容性上，GPS 不只适用于 GRPO，也能与 PPO、Reinforce++ 结合——这对 DS 类方法构成结构性差异：PPO 可能是单响应生成设置，依赖「每 prompt 多条回答」的真实评估难以适配，GPS 靠预测模型则天然兼容^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:484-489]。消融确认三个组件各管一件事：去掉 history-anchored diversity 性能明显下降，只去掉 inter-step exploration 也有一定下降，把生成式 PPM 换成不含潜变量 z 的确定性版本同样变差；对应回设计动机——共享历史管泛化、batch 多样性管去冗余、difficulty context 管刻画不断变化的模型状态^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:494-499]。

## 实践启示

1. 把「中等难度」当成 RLVR 数据筛选的第一原则：太简单的题梯度信号弱，太难的题几乎没有有效学习信号，「有时对有时错」的样本才值得练；二值奖励下可直接用「成功率接近 0.5」作量化代理^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:88]。
2. 成本账要落在 rollout 维度而非 step 维度：DS 的强是 step 维度的强，摊到 rollout 上反而更贵；GPS 在 DSR-1.5B/7B 上训练时间 16h/49h，低于 DS Oracle 的 30h/77h，ID 平均准确率持平甚至更高^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:386-391]。
3. 用 ESR 当 batch 质量的直接观测指标：ESR 指 batch 中能产生非零奖励方差的 prompt 比例，越高说明有效训练信号越多；GPS 的 ESR 明显高于 Uniform 与 MoPPS，自建数据选择管线值得引入同款监控^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:312-317]。
4. 训练阶段学到的资产优先考虑跨阶段复用：PPM 训练后直接迁移到测试时 Best-of-N 预算分配，同一份难度判断在训练侧省 rollout、在推理侧省采样预算^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:456-461]。
5. 把 prompt 选择当成独立工程环节，而非 RL 算法的附属品：这篇工作的价值不在更复杂的更新算法，而在把「哪些题值得练」单独拎出来；能产生有效奖励差异的中等难度 prompt 才是核心资产，后训练的下一步可能不是堆 GPU，而是先判断哪些计算值得花^[raw/articles/icml-2026-gps-prompt-predictive-model-rl-post-training-tsinghua-tencent.md:514-529]。
6. 放回 wiki 知识网：GPS 是 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 数据侧的效率路线，与 [[concepts/grpo-policy-optimization-2026|GRPO]] 的难度/方差动机互补；数据侧同类工作见 [[entities/dataflex-rl-pku-rl-data-strategy-framework|DataFlex-RL]]、[[entities/good-qc-for-rl-data|RL 数据质检]] 与 [[entities/tsinghua-popo-group-prioritized-off-policy-optimization-rlvr|清华 PoPO]]，测试时预算分配参见 [[entities/gka-variable-test-time-compute-2026|GKA 变量测试时算力]] 与 [[concepts/inference-optimization|推理成本优化]]。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

