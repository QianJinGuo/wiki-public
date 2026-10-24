---
title: "天下苦Self-Distillation久矣？1.2万卡时实测：最能打的还是GRPO"
created: 2026-10-25
updated: 2026-10-25
type: entity
tags: [agent, reinforcement-learning, self-distillation, grpo, benchmark, paperweekly]
sources: [raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026]
confidence: 0.7
---

# 天下苦Self-Distillation久矣？1.2万卡时实测：最能打的还是GRPO

> **Background**：本页基于 PaperWeekly 对浙江大学与蚂蚁集团复现研究《More Methods, Better Agents? A Reality Check on Self-Distillation in Agentic Reinforcement Learning》（arXiv 2610.19789）的报道整理，记录把 11 种训练方法放进统一框架重比一次的大规模实测。

## 评测困境与动机

Agentic RL 的奖励天然稀疏：任务经过十几步乃至几十步交互才给出终局成败，该信号并不说明哪些中间决策值得强化。Self-Distillation 在训练期引入技能描述、后续观测、环境验证过的教师续写等信息，为轨迹补上更细粒度的监督，推理时并不需要。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

难点在证据链。90 天内出现了 47 篇 Agent 蒸馏相关论文，平均不到两天一篇；而本次纳入比较的 10 种方法，原始论文中只有 1 篇报告了重复评测，5 篇分析过训练失败，3 篇报告了实测计算开销。许多工作沿用前人基线分数，环境与评测协议却未必一致，涨分有多少来自方法本身难以归因。团队索性不提新算法，把已有方法搬进统一框架重跑。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

## 实验设计（11 方法 × 4 任务 × 3 档模型 × 132 次训练 / 1.2 万 H800 卡时）

实验比较 GRPO 与 10 种自蒸馏方法，模型覆盖 Qwen3.5-2B、4B、9B，任务覆盖 ALFWorld、ScienceWorld、Search-QA 与 WebShop，共 12 组设置。每种方法在每组下各训练 100 步，每 5 步记录一次验证成绩，完成后用 8 个随机种子重复评测，累计 132 次训练。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

环境与评测流程统一，参数参考源论文与公开实现确定。12,124.4 H800 GPU 小时含训练、过程验证与模型保存，尚未计入独立重复评测，算力差距仅是训练下界。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

## 关键发现

第一层是"各有胜场，无人通吃"。2B 的 ALFWorld 上 Skill-SD 成功率达 74.29%，GRPO 仅 28.48%，相差超过 45 个百分点；但换到 WebShop，Skill-SD 在 4B 与 9B 上成功率均为 0%，GRPO 分别为 66.02% 与 70.02%。换模型规模或环境即失效。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

把 12 组设置等权汇总，PCSD 以 54.26 排第一，AgentOPSD 为 51.34，GRPO 以 49.68 位列第三，10 种方法中只有前两者综合均值超过 GRPO。但 PCSD 对 GRPO 只是 6 胜 5 负 1 平，未在任何一组设置中排名第一；ICSD 在 7 组设置中赢过 GRPO，综合均值却只有 44.75，因为它在 4B WebShop 上成功率归零，一次严重失利抵消了其余场景的小幅领先。可见该基线难以超越。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

第二层是最终分掩盖训练过程。按验证曲线下面积（AUC）排名，OCSD 第一而最终成绩只排第四，AgentOPSD 最终第二却 AUC 第九。4B ALFWorld 最典型：AgentOPSD 第 35 步已有 60.0%，随后连续 6 次验证不超过 4.3%，第 100 步回升至 79.3%；同设置下 GRPO+OPSD 未走出低谷，结束时仅 2.1%；OCSD 在 85.0% 峰值后明显回落。崩盘后恢复、持续低迷、后期退步说明：能否保住已获性能是独立于最终分的维度。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

第三层是评测协议本身会移动结论。2B Search-QA 中，OVCSD 的同一个最终模型贪心解码准确率只有 1.34%，改用温度 0.4 重复采样、8 个随机种子平均后达 33.21%。同一批问题换协议相差二十余倍，解码设置不能当无关细节。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

第四层是同等算力下的反转。自蒸馏需教师打分、轨迹采样或优势计算，同样 100 步耗时差很多：120 组对照中 112 组需要更多净训练时间，其中 58 组最终成绩反而更低、7 组持平；领先的 PCSD 与 AgentOPSD 平均耗时分别增加约 17% 与 19%。统一到 GRPO 时间预算后，ALFWorld 上 AgentOPSD 训满 100 步平均验证成功率 68.10%，高出约 10 个百分点，但在 GRPO 预算内只能取得 37.63%；PCSD 在 WebShop 上也出现同类反转。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

## 深度分析

### 一、标题是方法论主张：132 次训练买的是方差，不是新算法

把它当成"第 12 种自蒸馏方法"来读是误读。它没有提出新算法，提出的是可比性：同一套环境与评测流程、同一批 100 步预算、每 5 步记一次验证、8 个随机种子重测，把 11 种方法压在 12 组设置上跑满 132 次训练。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:63-67]

这个投入买到的不是精度，是分母。10 篇原论文只有 1 篇报告过重复评测、5 篇分析过训练失败、3 篇报告过实测开销，意味着此前多数"超过基线"的断言连基线自身的方差都没测过——涨分有多少来自方法、多少来自种子与解码，无法拆分。剥出"方法效应"是 [[concepts/evaluation-harness-design]] 里最贵也最少人做的一步。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:55-59]

买不到的同样明确：3 档 Qwen3.5、4 个任务族、每组 100 步。100 步内的排序对"训练到收敛"无话可说，AUC 只是对同一预算内曲线形状的补偿。参数按源论文与公开实现确定、并对部分方法做了适配——适配本身是新变量，实现偏弱会系统性低估该方法，而这一点论文无法自证。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:67]

### 二、教师—学生耦合内生奖励黑客，GRPO 的组内基线绕开的只是退化点

自蒸馏的诱惑在于把监督从终局挪到中间：技能描述、后续观测、经环境验证的教师续写，都为稀疏的终局成败补上更细的梯度。代价是信号来源与策略耦合——教师评的是学生自己采的轨迹，学生优化的又是教师的评分。一旦评分与任务成功解耦，最优策略就变成迎合评分而非完成任务，这是训练期的奖励黑客——[[entities/anthropic-reward-hacking-hacker-opus-hugging-face-2026-09]] 记录的是推理期版本。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:49-53]

GRPO 的组内基线切在另一处：同题采一组 rollout，组内均值当 baseline，advantage 只表达"比同组其他人好多少"。它自适应、无需 critic，且对奖励的常数偏移与整体漂移免疫。整组都崩时优势仍在组内分配，信号不会因绝对分虚高而消失；绝对奖励塑形下，奖励整体虚高会同时抬高 baseline 与策略，梯度被稀释成噪声。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:83-93]

边界要说清：组内基线免疫的是退化（奖励塌成常数、训练停摆），不是作弊。它消除偏移，不消除"奖励本身可被利用"。这解释了数据里最刺眼的一格——Skill-SD 在 2B ALFWorld 拿到 74.29%（GRPO 28.48%），同一方法在 WebShop 的 4B 与 9B 上却是 0%（GRPO 66.02% / 70.02%）。真学到可迁移技能的方法不会这样塌，这更像学到了该任务教师线索的表层特征。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:73-79]

### 三、蒸馏轨迹还是蒸馏奖励：证据只支持前者的一半

10 种方法按作用点拆开只有两类：改奖励与权重的（辅助蒸馏目标、优势重加权、奖励塑形），和改轨迹内容的（环境验证过的教师续写）。两类证据强度不同。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:53]

奖励侧偏弱。PCSD 与 AgentOPSD 只比 GRPO 高 4.58 与 1.66 分，且靠"每步更贵"换来：净训练时间多约 17% 与 19%，把预算对齐后 AgentOPSD 在 ALFWorld 从 68.10% 掉到 37.63%，PCSD 在 WebShop 同样被反超。增益主要存在于额外计算里，而不在"更会做"里——密集奖励塑形的收益被自身开销吃掉。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:125-135]

轨迹侧更站得住，代价也最高。教师续写有效，是因为把监督锚回环境而非模型自评：续写要真的被执行、真的被验证，作弊空间被环境卡住；但它同时需要额外采样与验证，正是最贵的一类。所以证据支持的命题是"用环境验证过的中间信号补 credit assignment"，不是"用模型自产奖励替代环境奖励"。前者是数据增强，后者是把 evaluator 交给被评者——与 [[entities/on-policy-distillation-vs-offline-distillation-loster]] 讨论的在线/离线分野同源：同是在线，锚点在哪决定了它是补监督还是造捷径。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:133-139]

### 四、成本模型：每一点 pass@1 的价格，与算力受限/数据受限两种处境

12,124.4 H800 卡时只含训练、过程验证与模型保存，未计独立重复评测，是下界而非账单。更关键的是分布而非总量：120 组对照里 112 组比 GRPO 更慢，其中 58 组更慢且更差、7 组持平——多数方法在"多花时间"的同时没买到分数，只有少数买到了。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:67]

折成单位价格就很冷静：PCSD 多 17% 时间换 4.58 分，约每分 3.7% 额外训练时间；AgentOPSD 多 19% 换 1.66 分，约每分 11.4%。这还是 100 步、小模型、四任务的价格，换到长程任务上，轨迹采样与验证开销只会更陡。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:127]

于是"最强方法"取决于约束落在哪一侧。算力受限（固定 wall-clock、按迭代节奏上线）：判据是同等时间内最后一次验证成绩，GRPO 赢，且不靠技巧。数据受限（算力充裕而高质量交互任务稀缺）：自蒸馏本质是用计算换监督密度，额外开销换来的每步信息量才可能划算。论文真正的贡献是把这条约束显式化，而不是给出排名——100 步预算下取"曲线尾部"天然惩罚见效晚的方法，那是预算不匹配，不是方法更差。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:129-139]

### 五、网格怎么读，以及什么样的后续实验才算能证伪

12 组网格不均匀。翻转集中在动作空间形态：ALFWorld 是文本 household 指令，动作可枚举、技能描述能覆盖状态空间，Skill-SD 因此大幅领先；WebShop 是长列表检索加属性匹配，商品空间无法被技能描述覆盖，同一方法直接归零。平坦区在 ScienceWorld 与 Search-QA——多数方法差距落在噪声内，这里选型该看成本而非分数。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:73-81]

排名本身也不稳。PCSD 对 GRPO 6 胜 5 负 1 平，却从未在任何单组设置里排第一；ICSD 在 7 组赢过 GRPO，均值只有 44.75，因为一次 4B WebShop 归零抵消了其余所有小胜。均值排名与逐组胜率不一致，说明"综合第一"是等权 12 组的产物，按业务真实任务分布重新加权名次就会变。2B Search-QA 上 OVCSD 贪心 1.34%、温度 0.4 八种子平均 33.21%，同模型同题差二十余倍——这正是 [[concepts/eval-surface-rotation]] 说的：评测面一转，结论跟着转。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:113-117]

要让这篇被推翻，后续实验至少需要四件事：主表用同等 wall-clock 而非同等步数（论文只做了补充分析）；至少一个未参与调参的 held-out 任务族；8 种子的标准差与置信区间而非只有均值；实现强度对照——原作者代码与复现实现各跑一次，把适配成本量化。若同等 wall-clock 加 held-out 族上自蒸馏仍领先，结论被推翻；若领先只在训练族内保持、到 held-out 消失，那是过拟合到 benchmark 而非方法无效，两种失败模式必须分开归因。公开 11 种方法的代码是这篇最耐久的产出：它把"方法主张"变成"可被重跑的主张"，比任何一个 54.26 都更重要。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md:141-147]

## 对 Agentic RL 工程实践的启示

一线选型不该只看"涨分"。至少同时看跨任务与跨规模的稳定性、训练曲线形状、每单位提升的算力代价：前两项决定能否上线，第三项决定值不值得上线。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

其次是可复现性的最低报告标准。10 篇原论文中只有 1 篇报告重复评测、3 篇报告计算开销，仅报最终分几乎不足以支撑方法主张，重复种子评测、训练失败案例、净训练耗时与解码协议都应写入报告。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

最后是校准 GRPO 的定位：它不是"待改进的弱基线"，而是需按算力预算对齐后才谈得上超越的强基线。自蒸馏的价值更可能落在特定任务形态而非通用替代，公开代码的长期价值可能高于单点结论。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

## 关联

算法谱系上，本文的 GRPO 属于 [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]] 梳理的组相对策略优化分支，奖励来自可验证信号，与 [[concepts/rlvr-reinforcement-learning-verified-reasoning]] 同源。自蒸馏在此用于密集化奖励与轨迹监督，与 [[concepts/reinforcement-fine-tuning-rft]] 互补，血统延续 [[concepts/model-distillation-compression]] 的教师—学生结构，教师信号改为环境验证。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

同类在线自蒸馏可对照 [[entities/unievo-vl-onpolicy-self-distillation]] 与 [[entities/d-opsd-diffusion-llm-on-policy-self-distillation]]，它们是这一族方法在视觉语言与扩散语言模型方向的延伸；本篇价值在于提供了统一口径的横向标尺。^[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026.md]

→ [[raw/articles/self-distillation-agentic-rl-reality-check-grpo-2026|原文存档]]
