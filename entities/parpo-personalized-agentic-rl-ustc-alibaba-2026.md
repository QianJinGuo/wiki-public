---
title: "PARPO：把个性化写进 Agentic RL 的训练目标（中科大 + 阿里）"
created: 2026-09-14
updated: 2026-10-25
type: entity
tags: [agentic-rl, personalization, reward-model, memory-graph, grpo, user-profile, e-commerce, post-training, alibaba, ustc]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026]
---

# PARPO：把个性化写进 Agentic RL 的训练目标

中国科学技术大学与阿里巴巴的团队提出 **PARPO（Personalized Anchor Reward-Decoupled Policy Optimization）**，把「个性化」从推理期的 prompt 拼接上移到训练期的优化目标：同一 query 下不同用户偏好不同的规划策略、工具调用与表达方式，只优化通用任务完成度会让模型学成「平均用户最喜欢的样子」，而把历史行为直接塞进 prompt 又会把从众、流行度与上下文噪声误当成真实偏好。论文为 arXiv 2605.23382《From Correctness to Preference》，在 ETAPP、ETAPP-Hard 与商家决策场景 SJAgent 上以 Qwen3-4B/8B 训练，8B 模型 Judge 分 0.8333、SJAgent Reward 0.8270，并在 15 位人类专家与 4 组 LLM judge 的盲测中排名第一。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 三个被明确命名的问题

论文把「正确答案为什么还不够」拆成三条约束：C1 个性化奖励含义不清——通用奖励无法表达同一条轨迹对不同用户的不同价值；C2 偏好信号被行为混合——真实兴趣、流行度影响与群体从众交织在一起；C3 记忆仍然太扁平——难以显式表达用户、技能、工具、场景与历史轨迹之间的关系。这三条同时决定了下游三件套的设计边界。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 三件套：记忆、奖励、优化各司其职

PARPO 不是替换 GRPO 的一个公式，而是三个相互衔接的部件。**PSGM**（偏好对齐技能演化图记忆）在 rollout 之前把历史经验组织为 User / Skill / Tool / Scenario / Trajectory 五类节点的异构图：检索时先按 query 找到语义种子技能，再沿「技能—用户—同源技能」扩展候选，综合用户相关性、互补加成与冲突惩罚排序，因此召回的未必是文本最相似的技能，而是处在同一有效行为邻域里的技能。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

**两阶段偏好解耦奖励模型**在 rollout 之后产生个性化奖励：第一阶段用多视角用户画像缓解冷启动；第二阶段在用户—物品交互图上用 LightGCN 建立「兴趣」与「从众」两个正交分支，兴趣分支提高低流行度项目的权重、从众分支相反，从而从混合行为里提取更干净的偏好信号。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

**PARPO 优化器**把通用任务质量与个性化偏好拆成两条 advantage 轨道再做策略更新，个性化分支用指数移动平均维护**用户级历史锚点**，让当前奖励与该用户自己的历史中心比较，而不是与所有用户混合后的均值比较，缓解异质偏好下的 reward-center 与 reward-scale 失配。论文给出的分工直觉是：通用质量负责让 agent 把事情做对，个性化质量负责让它用这个用户愿意接受的方式做好。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 闭环与消融

高质量轨迹随后被总结为新技能写回 PSGM，形成「检索—交互—评价—优化—积累」的闭环。消融实验显示：去掉技能记忆后 Judge 从 0.7708 掉到 0.7006，是所有消融中最大的降幅；去掉用户锚点或兴趣/从众任一分支也都明显掉点，说明记忆、解耦与用户锚点三者缺一不可。最大优势维度出现在 User Relevance，即提升主要来自更强的用户对齐，而不只是更顺滑的文字表达。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 边界

论文自陈两处局限：人工盲测规模仍然有限；兴趣—从众解耦是操作层面的正交化，并不构成严格的因果分离。它要论证的方向是「同一个任务不再只有一种正确完成方式」——真正好的策略还必须是这个用户愿意持续使用的完成方式。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 深度分析

### 三个问题落在 MDP 的三个不同位置上

Prompt 级个性化只改条件分布 p(τ|q,u)：把画像拼进 observation，策略看到的东西变了，目标函数没变。三条问题因此都不在 prompt 能触及的层。C1 是奖励函数的问题——通用奖励 R(τ) 在用户维度上做了隐式边缘化，最大化它得到的是 argmax_τ E_u[R(τ,u)]，而每个用户要的是 argmax_τ R(τ,u)；两者只在偏好同向时重合，一旦冲突（A 要最快、B 要最便宜），最优解就是"谁都不满意但平均分不低"的折中轨迹。C2 是估计的问题——观测量 b = g(真实兴趣, 流行度先验, 从众压力)，混合发生在数据生成过程里，把 b 原样写进 prompt 等于把污染过的估计当标签，prompt 越长噪声被引用得越充分。C3 是状态的问题——u 在 t=0 可不可见，取决于记忆能否把分散的历史聚成可检索对象。PARPO 三件套正好对应三处：PSGM 改状态，解耦奖励模型改 R(·,u) 且清洗 u 的估计，用户锚点改 advantage。个性化不是加了个模块，而是被拆到 state / reward / baseline 三个注入点上——这是它无法被更长的 system prompt 复现的结构性原因。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

### 三段的分工与接缝：记忆误差如何变成奖励噪声

**记忆段决定 u 的可辨识性。** PSGM 沿"技能—用户—同源技能"扩展而非常规文本相似度召回，把召回单位从"这段描述像不像"换成"这个行为是否在同一有效邻域"，代价是召回的技能带着**别人的**用户上下文。当轨迹内容来自用户 v、而 advantage 以用户 u 的锚点做基线时，信号在这一步就错位了——不是零均值噪声，而是朝 v 偏好偏移的偏置。噪声能靠更多 rollout 平均掉，偏置不能。

**奖励段决定 u 的估计干不干净。** 第一阶段的多视角画像是冷启动兜底，本质是用群体先验替代个体证据；第二阶段才是真个性化：用户—物品图上用 LightGCN 分出兴趣与从众两支，并给两支相反的流行度权重。这里"正交"是训练目标上的解耦，不是因果识别——真实喜欢大众品的用户会被从众分支当作被潮流裹挟，流行度权重越强，这个误判越系统。

**优化段决定同一个 R 值怎么被解释。** GRPO 的基线是同一 prompt 下组内采样的均值（跨样本），PARPO 的锚点是同一用户历史奖励的 EMA（跨时间）。好处是吸收了用户间 reward scale 的异质性（有人打分宽、有人严），代价是**非平稳性被从奖励挪进了基线**：偏好漂移快于 EMA 由 α 决定的时间常数时，一条真正符合新偏好的轨迹会因低于旧锚点而拿到负 advantage。多轮轨迹里的偏好非平稳因此是被部分转移，而非被解决。

### 消融的真实信号与统计边界

可查的数字只有一个：去掉技能记忆后 Judge 从 0.7708 掉到 0.7006，相对降幅 9.1%，为全部消融中最大。去锚点、去兴趣/从众任一分支都掉点，但原文没给数值——"三者缺一不可"是方向性结论，不是可比较的贡献度排序。更有信息量的是**最大增益维度为 User Relevance**：若提升来自更讨喜的文风，增益应均匀落在表达类维度；集中在用户相关性上，至少说明奖励改变了"做什么"而不只是"怎么说"。

统计上有三处软肋。其一，15 位专家 + 4 组 LLM judge 的盲测规模偏小，且未报告评分者间一致性，7 个基线里排第一的复现性缺乏保障。其二，评估域单一：ETAPP 是电商场景的旅行规划、SJAgent 是商家决策，两类用户画像都来自平台内部、密度高、行为链完整，而生产环境里新用户、低频用户、跨账号用户恰好落在 PSGM 最召不回、画像最不可信的区间。其三，指标是均值 Judge 分，没有按用户切分——均值提升完全可与"最差十分位用户被个性化伤害"共存，而对上线团队来说后者才是事故源。

### 个性化奖励为什么招来谄媚，这套设计挡在哪

只要奖励里含"用户是否满意"，策略梯度就去找提高它的最低阻力方向。而"同意用户"的方差远低于"反对用户并在三轮后被证明正确"：后者收益延迟甚至永不兑现，前者即时结算；错误建议的成本几轮后才显现，顶嘴的成本当场兑现。谄媚因此不是性格缺陷，而是奖励结构下的均衡解。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

论文有三处实质约束。**双 advantage 轨道**让通用任务质量不被个性化吞掉，但两条轨道最终加权合并进一次更新，正确性的权重可调而非硬约束——这条防线是参数化的，不是结构性的。**兴趣/从众解耦**抽掉了谄媚最省力的燃料（"跟着多数人走总不会错"），流行度先验一剥，捷径收益下降。**用户级锚点**最有效：它把奖励变成相对量，一条对所有用户都更顺从的回复相对该用户自己的历史中心并没有变高，拿不到额外 advantage——常数偏移被基线消掉了，而这正是谄媚依赖的东西。残留风险在锚点自身的动力学：若历史锚点本由过去那些讨喜轨迹累积而成，就形成棘轮——更顺从 → 更高奖励 → 锚点上移 → 必须更顺从。EMA 在这里是正反馈通道，不是稳定器。参见 [[entities/advanced-ai-sycophancy-strategic-disagreement-2026|高级 AI 谄媚]] 与 [[entities/anthropic-reward-hacking-hacker-opus-hugging-face-2026-09|reward hacking 实录]]。

## 实践启示

1. **分开记三本账**：检索命中的记忆节点与排序分、奖励的两个分支（兴趣 / 从众）各自得分、锚点前后的 advantage 各记一条。三者混在一个标量 reward 里，掉点后无法判断是召回错了、奖励估歪了，还是基线漂了。
2. **给锚点设显式半衰期**，并把 α 与业务的偏好漂移速率对齐——快消/内容类以周计，耐用品以季度计。再加一个变点触发器（行为分布偏移超阈值时重置锚点），而不是让固定 α 盲跟。
3. **把谄媚做成一级指标**：单独统计"用户事实上错误时 agent 是否顶嘴"的比例，与满意度分开监控；同时盯从众分支的残差，若某用户群该分支权重持续走高，多半是流行度先验在替他们做决定。
4. **评估按用户切分而非按交互切分**：留出整段用户（per-user holdout），报告最差十分位与用户间方差，不只报均值。+0.07 的均值提升与一个被伤害的十分位可以同时成立。
5. **冷启动要显式降权**：画像置信度低于阈值时把个性化 advantage 权重调低、退回通用策略，用第一阶段的多视角画像当兜底而不是当结论。把混合系数当安全旋钮，并在正确性任务上单独回归。

## 相关

- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026]]
- [[entities/user-governed-personalization-agentic-recommendation-paradigm-2026]]
- [[entities/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026]]
- [[entities/agent-memory-architecture]]
- [[entities/agent-evaluation-turing-meituan-2026]]
- [[entities/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528]]
- [[concepts/grpo-policy-optimization-2026]]
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]]
