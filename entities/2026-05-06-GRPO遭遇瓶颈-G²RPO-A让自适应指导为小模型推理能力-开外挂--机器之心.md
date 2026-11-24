---

title: "GRPO遭遇瓶颈？G²RPO-A让自适应指导为小模型推理能力「开外挂」"
created: 2026-07-01
updated: 2026-11-14
type: entity
tags: [ai, training]
sources: [raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# GRPO遭遇瓶颈？G²RPO-A让自适应指导为小模型推理能力「开外挂」

## 摘要

小规模语言模型（SLMs）在 RLVR 后训练中长期吃不到有效奖励：GRPO 在 7B+ 模型上效果显著，落到 1.7B 级别却几乎不再涨分。香港中文大学（深圳）T-Lab 唐晓莹教授携课题组博士毕业生郭永新、邓文博（合作者还包括 Zhenglin Cheng，单位涉及淘天集团与西湖大学）提出 G²RPO-A（Guided Group Relative Policy Optimization with Adaptive Guidance），已被 ACL 2026 主会议接收：在 roll-out 部分轨迹中注入高质量思维轨迹，并按训练状态动态调整指导强度，缓解小模型的稀疏奖励困境。^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md] 在 Qwen、Llama、DeepSeek 等家族的实验中，Qwen3-1.7B 在 MATH500 上从 50.96 提升到 67.21，HumanEval 上从 46.08 提升到 75.93，显著优于 vanilla GRPO。论文链接：https://arxiv.org/abs/2508.13023 ；代码：https://github.com/T-Lab-CUHKSZ/G2RPO-A 。

## 核心要点

- **问题定位**：GRPO 等强化学习算法在 1.7B 及以下的小模型上性能提升微乎其微，根源不是「模型太小」这一句话，而是稀疏奖励——小模型很难在 roll-out 中采到高质量思考链，大部分候选拿不到正向奖励。^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]
- **方法本质**：G²RPO-A 不是把标准答案直接喂给模型，而是在生成 roll-out 的过程中注入部分高质量 thinking trajectory，把分布往高奖励区域推。
- **两个组件**：指导机制（Guidance Mechanism，注入部分思维轨迹引导生成）+ 自适应调整（Adaptive Adjustment，动态调整指导长度与指导比例）。
- **naive 指导的陷阱**：固定长度的简单指导只能短暂抬高 reward，且 advantage 标准差极低，训练效率并没有真正提升——「更容易采到高奖励」不等于「更健康的优化信号」。
- **自适应规则**：以最近若干 step 的奖励为窗口，奖励走高则缩短 guidance 让模型自主推理，奖励走弱则拉长 guidance 降低难度。
- **代表性结果**：Qwen3-1.7B-Base 在 MATH500 从 50.96 → 67.21、GPQA 从 27.45 → 32.35；Qwen3-0.6B 在 HumanEval 从 32.32 → 44.96；Qwen3-1.7B 的 Code-Avg 63.95 高于 GRPO 的 60.40。
- **经验规律**：代码任务通常需要更高 guidance ratio，小模型比大模型更依赖 guidance——这正是作者放弃固定超参、转向自适应的直接动机。

## 深度分析

### 为什么 GRPO 在小模型上失效

GRPO 依赖组内相对比较产生 advantage：一个 prompt 采样一组候选，用组内奖励的相对差当作优化信号。这套机制隐含一个前提——组里至少能采到几个有区分度的奖励。大模型能力足够，复杂推理任务的 roll-out 中总有部分候选命中正向奖励，信号天然存在。而 1.7B 级模型面对同样任务时，大部分 roll-out 根本生成不出可用的思考链，Qwen3-1.7B 在代码任务上的奖励热力图呈现出极度稀疏的分布：几乎全组都是零奖励，组内没有差异，advantage 接近于零，策略梯度失去方向。研究团队把这种状态比作「新手司机开手动挡」——引擎（模型）再努力，没有正确引导就完成不了复杂驾驶（推理）。这与 [[concepts/grpo-policy-optimization-2026|GRPO 策略优化]] 的组相对基线设计直接相关，也和 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 场景下奖励可验证但难采样的普遍矛盾一致。

### G²RPO-A 的自适应指导机制

G²RPO-A 的介入点在 roll-out 阶段：每一步训练把候选分成 guided 和 unguided 两组，对 guided 组注入部分高质量思维轨迹作为前缀引导，让小模型「接着聪明的话往下说」，从而更容易采到高奖励候选。关键在于两点设计。其一，指导只覆盖部分轨迹和部分候选，unguided 组仍然保留模型的自主探索分布，避免优化目标被外部轨迹完全接管。其二，指导强度不是固定超参：论文根据当前奖励与历史奖励的比值动态调整后续 guidance length，形式上接近「m=min(T,k) 的历史窗口内，奖励走高则缩短 ℓₖ，奖励走弱则拉长」。直观理解是「智能变速」：模型最近学得顺，就撤掉脚手架让它自己走；模型最近学不动，就多扶一段降低难度。这一思路与 [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演进]] 中「训练信号要随策略状态自适应」的方向呼应。

### 为什么简单指导行不通

工作里最有分析价值的部分是 naive guidance 的对照实验。在基于 Math-220K 子集的训练动态分析中，固定长度的指导确实能在早期短暂抬高 reward（图 1 中 Qwen2.5-Math-7B 在 s1K-1.1 上的表现即是如此），但很快与 vanilla GRPO 殊途同归。深层原因是 advantage 标准差被压得极低：当大家都容易被引导到高奖励区，组内候选的奖励趋同，区分度消失，梯度信号变弱，训练效率反而受损。这个发现把「给小模型加提示」从直觉上的好事变成一个需要精细权衡的信号工程问题——指导的价值不在于抬高绝对奖励，而在于在不牺牲组内方差的前提下把奖励分布搬进可优化区间。这与 [[entities/llm-rl-entropy-collapse-acl-2026-outstanding-paper|LLM RL 熵坍缩问题]] 关注的「探索性信号耗尽」是同一类困境的不同侧面。

### 实验结果与边界

主实验在统一设置下对比 Base、vanilla GRPO 与 SFT。数学侧：Qwen3-1.7B-Base 在 MATH500 从 50.96 → 67.21、GPQA 从 27.45 → 32.35，Qwen3-8B-Base 在 MATH500 从 71.32 → 82.08；更强数学设置下 Qwen3-1.7B 在 AIME24/AIME25 达到 63.33/53.33，高于 GRPO 的 56.67/50.00。代码侧：Qwen3-0.6B 在 HumanEval 从 32.32 → 44.96、LiveCodeBench 从 17.07 → 23.14；Qwen3-1.7B 在 HumanEval 从 46.08 → 75.93。需要如实记录的反例是 Qwen3-1.7B 在 LiveCodeBench 上 SFT 略高，但 Code-Avg 综合对比中 G²RPO-A 以 63.95 仍高于 GRPO 的 60.40 与 Clip-Higher 的 60.19——整体占优而非单项碾压。作者也坦承两个边界：验证集中于数学与代码，跨模态场景未检验；guidance ratio α 仍依赖经验搜索，离完全自适应还差一步。

## 实践启示

1. **1.7B 级模型别裸跑 GRPO**：奖励稀疏会让组相对信号失效；若必须用小模型做 RLVR，先看奖励热力图确认高奖励候选密度，再考虑引入外部指导。^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]
2. **给指导前先看 advantage 方差**：任何抬高奖励的干预都可能压扁组内区分度；监控 advantage 标准差应与监控 reward 并重，方差塌了说明优化信号在丢失。
3. **指导强度做成函数而非超参**：按近期奖励趋势动态调整（涨则撤、跌则扶），比网格搜索一个固定 guidance length/ratio 更贴合训练过程的状态变化。
4. **代码任务要多给指导**：配置分析显示代码任务通常需要更高 guidance ratio，且小模型比大模型更依赖 guidance；按任务域与模型规模分别整定，不要一套参数通吃。
5. **unguided 组不可省**：保留一部分无指导 roll-out 是维持自主探索分布、防止策略被外部轨迹绑架的结构性保障。
6. **参考实现**：论文与代码仓库已公开（github.com/T-Lab-CUHKSZ/G2RPO-A），可对照 [[entities/self-distillation-agentic-rl-reality-check-grpo-2026|GRPO 自蒸馏实践检查]] 与 [[entities/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct|350M 模型 GRPO 结构化输出微调]] 交叉验证小模型 RL 训练管线。

## 关联

- 概念：[[concepts/grpo-policy-optimization-2026|GRPO 策略优化]]、[[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]]、[[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演进]]、[[concepts/harness-engineering-framework|Harness Engineering]]
- 实体：[[entities/overcoming-reward-signal-challenges-verifiable-rewards-based-reinforcement-learn|可验证奖励的信号挑战]]、[[entities/llm-rl-entropy-collapse-acl-2026-outstanding-paper|熵坍缩]]、[[entities/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心|强化学习的进化（PPO→MaxRL）]]

→ [[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心|原文存档]]
