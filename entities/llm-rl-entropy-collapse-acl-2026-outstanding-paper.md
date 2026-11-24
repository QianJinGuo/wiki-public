---

title: "大模型RL为何越训越窄？ACL 2026杰出论文揭秘熵坍缩真相"
created: 2026-07-24
updated: 2026-11-13
type: entity
tags: [reinforcement-learning, llm, entropy, collapse, acl-2026, training-dynamics, rlvr]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 大模型RL为何越训越窄？ACL 2026杰出论文揭秘熵坍缩真相

> 浙江大学、腾讯等机构的论文 "Rethinking Entropy Interventions in RLVR: An Entropy Change Perspective" 获 ACL 2026 Outstanding Paper Award。该工作从 token-level 熵变出发，重新解释 RLVR 中的熵坍缩现象，并提出熵稳定方法 STEER，在数学推理和代码任务上取得稳定提升。

→ [[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper|原文存档]] ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

## 问题：RLVR 训练中的熵坍缩

RLVR（Reinforcement Learning with Verifiable Rewards）正在成为大模型后训练的关键技术。然而，训练中策略熵（policy entropy）会快速下降，导致模型输出逐渐同质化，采样路径变窄，训练也可能提前停滞。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

对于 GRPO 这类 group-based 算法，问题尤为严重：当同组采样回答越来越相似，组内 advantage 信号失去区分度，训练随之停滞。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

## Token-level 熵变分析

论文将 RLVR 训练视为一棵推理树的动态调整过程。每个 token 选择对应一个分叉节点，token 熵反映该节点的"有效分叉数"。RLVR 训练同时包含两种力量： ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

- **增枝**：奖励信号鼓励新的推理分支生长 → 推动熵上升
- **剪枝**：低奖励轨迹中的分支被压制 → 推动熵下降

健康的训练需要二者保持动态平衡。论文推导了 token-level 熵变的近似表达式，发现熵变方向主要由**优势信号（advantage）**和**token 生成概率**共同决定： ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

| 类型 | 概率 | 优势 | 熵效应 |
|------|------|------|--------|
| 高概率正样本 | 高 | 正 | 局部熵下降（路径强化） |
| 低概率正样本 | 低 | 正 | 局部熵上升（保留探索） |
| 高概率负样本 | 高 | 负 | 局部熵上升（释放概率空间） |
| 低概率负样本 | 低 | 负 | 局部熵下降（剪枝） |

## 现有方法的局限

从 token-level 熵变视角重新审视现有熵干预方法：

- **DAPO / Clip-Higher**：放开 clip-high 让更多低概率正样本参与更新，但调节仍是粗粒度的
- **Positive-Reweighting**：削弱高概率正样本权重、强化低概率正样本，但忽略了负样本中的熵变力量
- **Entropy-Aware Advantage**：高熵 token 并不保证熵变方向，放大降熵方向的 token 反而会加速坍缩

这些方法缺乏对每个 token 熵变方向和幅度的直接刻画，对训练中熵动态的控制仍然是粗粒度的。

## STEER：Token-level 熵变稳定器

论文提出 **STEER（Stabilizing Token-level Entropy-changE via Reweighting）**： ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

- 直接估计每个 token 在当前更新中的熵变
- 对会带来过大熵变化的 token 降低权重（降权而非抹除）
- 大多数 token 权重仍接近 1，学习信号不受大面积削弱

与常见的熵奖励（entropy bonus）不同，STEER 不是粗暴地"把熵拉高"，而是让每一步训练的熵变化保持可控——像一个 token-level 的"限速器"。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

**实验结果**：
- 数学推理：Qwen2.5-Math-7B 平均分从 GRPO 的 44.2 提升至 48.6
- 代码任务：Qwen2.5-Coder-14B 代码编辑 42.6→45.1，LiveCodeBench v5 29.3→31.8
- 在不同模型（Qwen/Llama/Mistral）、算法（GRPO/RLOO/OPO）、规模上均表现稳定 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

## 深度分析

**1. 熵坍缩的分析单位从"训练轮次"降到"单次 token 更新"。** 论文最核心的方法论贡献是把熵坍缩从训练曲线上的宏观现象，拆解为每次参数更新中单个 token 熵变的累积效应——推理树上每个分叉节点的局部调整，最终决定整棵树的展开或收缩。这使熵坍缩从"可观察"变为"可归因、可调控"：现有方法（DAPO/Clip-Higher、Positive-Reweighting 等）在全局阈值或样本权重层面间接施力，而 STEER 直接作用于每个 token 的熵变本身。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

**2. 四象限分类揭示负样本同样携带升熵力量。** "优势信号 × 生成概率"的四象限表明：高概率正样本强化已有路径（局部降熵），低概率正样本保留稀有路径（局部升熵），高概率负样本释放概率空间（局部升熵），低概率负样本执行剪枝（局部降熵）。关键在于负样本一侧同样存在升熵/降熵两种方向——Positive-Reweighting 类方法只围绕正样本设计，等于放弃了负样本侧一半的控制自由度；这解释了它们"常有效但难完整控制熵动态"的局限。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

**3. 高熵 token ≠ 探索方向：幅度与方向是两个正交自由度。** Entropy-Aware Advantage 类方法的失效路径极具警示性：高熵只意味着该 token 更新可能带来更大幅度的熵变化，并不保证方向朝向增熵。若高熵 token 恰处在降熵方向，放大其 advantage 反而加速坍缩。这是把"变化幅度"误当"变化方向"的度量错位——任何基于熵代理指标（token 熵、条件熵）设计的干预都应先通过四象限检验其方向假设。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

**4. STEER 的设计哲学是"限速"而非"推高"，本质是 token 级信任域。** 与 entropy bonus 把熵最大化写进目标函数不同，STEER 把熵变化幅度本身作为约束对象：估计每个 token 在当前更新中的熵变，对剧烈者降权，其余权重保持接近 1。这与 PPO 的 clip 机制形成互补——clip 约束策略比率，STEER 约束熵变；即使在几乎不施加 clipping 的极端设置下，策略熵仍被维持在稳定区间，既不坍缩也不过度升熵导致发散。这种"机制先行、干预后置"的思路，比基于特定 benchmark 调参的 trick 更可迁移（Qwen/Llama/Mistral × GRPO/RLOO/OPO 全部稳定有效）。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

**5. 对 GRPO 系算法，熵坍缩是双重打击。** 熵坍缩不仅收窄探索空间，还直接摧毁 group-based 算法的工作前提：同组采样回答趋同后，组内 advantage 信号失去区分度，梯度估计退化，训练提前停滞。这解释了熵稳定干预对 GRPO 系的收益尤其显著，也提示 [[concepts/grpo-policy-optimization-2026|GRPO 策略优化]]的实践中应把组内多样性作为独立于策略熵的监控维度。与 [[entities/llm-rl中的熵-part-1-熵的调控|LLM RL 中的熵调控]]对比，本文的贡献在于把调控粒度从"训练步级"细化到"token 级"。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

## 实践启示

- **监控 token 级熵变分布，而非只看平均策略熵。** 平均策略熵是滞后指标；在训练循环中记录每步熵变的分布统计（如 |ΔH| 的分位数），能在坍缩发生前捕捉前兆。论文的 token-level 熵变近似表达式可直接作为插桩依据。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]
- **设计熵干预前，先用四象限检查表定位覆盖范围。** 问一句"这个干预改变了哪一类 token 的熵变贡献"：调 clip-high 阈值主要作用于低概率正样本，positive reweighting 作用于正样本两侧——单象限干预注定片面。四象限（优势信号 × 生成概率）可作为任何熵干预方案的完备性检查。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]
- **出现熵坍缩时，优先 token 级降权而非直接加 entropy bonus。** 熵正则会盲目推高熵，过大的熵增同样使训练发散。STEER 已开源（github.com/zz-haooo/STEER），可作为 GRPO/RLVR 训练管线的即插即用组件；与 [[entities/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|2026 年 LLM RL 方法总结]]中的其他熵干预方案对比选型。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]
- **把组内 advantage 方差作为 GRPO 的健康指标与干预触发器。** 组内回答趋同导致 advantage 信号失去区分度，是熵坍缩的下游症状且更早可观测；当组内方差跌破阈值时触发干预（如降权或增采样温度），比等策略熵跌破阈值更及时。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]
- **RLVR 后训练选型时关注机制级泛化证据。** STEER 在六个数学基准、代码编辑与 LiveCodeBench 上稳定提升，且跨模型家族、跨算法、跨规模验证——选型时应优先此类给出"为何有效"机制解释并在多设置下消融的方法，而非仅报告单基准 SOTA 的工作。相关背景见 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 与 [[entities/llm-post-training-full-guide|LLM 后训练全景]]。 ^[raw/articles/llm-rl-entropy-collapse-acl-2026-outstanding-paper.md]

## 相关实体

- [[entities/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|2026 年面向 LLM 的 RL 方法总结]] — LLM 强化学习方法全景
- [[entities/llm-rl中的熵-part-1-熵的调控|LLM RL中的熵 part 1: 熵的调控]] — 熵调控方法
- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 六框架实践地图]] — Agentic RL 训练实践
- [[entities/llm-post-training-full-guide|LLM Post-Training 全景指南]] — LLM 后训练综述
