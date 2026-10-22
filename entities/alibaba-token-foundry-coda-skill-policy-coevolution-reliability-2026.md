---
title: "阿里 Token Foundry CODA：技能—策略协同进化的「协同进化陷阱」与可靠性评测协议"
created: 2026-10-22
updated: 2026-10-22
type: entity
tags: [agent, skill, self-evolution, co-evolution, evaluation, reliability, post-training, rl]
sources: [raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026]
confidence: 0.8
provenance_state: extracted
---

# 阿里 Token Foundry CODA：技能—策略协同进化的「协同进化陷阱」与可靠性评测协议

> **Background**：本文档基于阿里巴巴 ATH Token Foundry 团队（作者冯钰捷）在微信公众号「阿里技术」发布的原创解读整理，对应论文 arXiv 2610.20845v1。文章报告了一项负面发现及其评测方案：技能—策略协同进化能把任务分数推高，但高分可能绑定在特定技能库版本上，部署后技能一变即失效。

## 核心结论：高分 ≠ 可靠

技能—策略协同进化（skill–policy co-evolution）让 Agent 从任务经验中生成、改进可复用的「技能」，再用技能训练策略模型，模型能力变化又反馈回下一轮技能更新，形成持续改进循环。团队发现：多轮协同进化带来的分数提升，可能同时加深模型对特定技能库的依赖，**使 Agent 在技能持续变化的部署环境中反而更脆弱**——他们把这一现象命名为「协同进化陷阱」。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

量化结果：在所审计的 6 组「方法—环境」设置中，技能内容变化后的**最差成功率比原始搭配低约 5–13 个百分点**，相当于把原始技能库相对「同一模型不使用技能」基线所带来增益的 **52%–93% 抵消掉**。作者特别澄清：被抵消的是技能库带来的增益，而非模型整体成绩下降同样幅度。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

## 为什么高分会被「锁死」在特定技能库上

研究从各协同进化方法的训练目标与技能筛选机制入手，梳理出一个共同倾向：**技能更新围绕当前模型的收益被筛选，模型又不断适应当前技能库**。单轮看这提升了成绩，多轮迭代后两者形成越来越强的相互适配——模型不仅依赖技能提供的知识，还可能对特定版本的**表达方式**产生敏感性。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

论文在理想化的技能筛选与候选覆盖条件下刻画了「原始搭配形成性能峰值」的机制，关键判断是：只优化某一个技能库上的峰值，并不能自动约束技能变化后的性能损失。「更擅长使用共同训练的技能库」与「更可靠地使用持续变化的技能库」是两个必须分别验证的目标。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

这条结论与 [[concepts/skill-engineering-principles|Skill 工程原则]] 和 [[concepts/skill-formal-theory-survey|Skill 形式化理论综述]] 中「技能即外部化上下文、其收益依赖模型—技能匹配」的观察互为印证：技能库不是无状态的知识包，而是与策略共同演化出的耦合体。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

## 五个技能变化探针

为模拟部署后技能的持续演化，团队构建了五个维度的探针：**同义改写**（保留原意只改表达）、**信息缺失**、**错误策略**、**技能库组成变化**、**完全移除技能**。五个维度分别检验：模型能否识别不同表达下的同一经验、能否应对不完整或错误的指导、对技能库结构与外部技能支持有多大依赖。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

其中同义改写最具诊断性——**若仅改表达就带来明显性能损失，说明依赖已超出技能中的有效知识本身**，落到表面形式层面。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

## 协同进化脆弱性曲线

沿训练轨迹同时记录原始成绩与技能变化后的表现，得到的曲线显示：在多个设置下，**原始技能库上的成绩持续提高，对技能变化的敏感性同步增加**。这就是「协同进化陷阱」的可视化形态——上升的成绩曲线掩盖了可靠性下降，团队称之为 Co-evolution Fragility Curve。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

实验覆盖 **SkillRL、ReSkill、Evolving-RL、ARISE 四种方法**与 **ALFWorld、WebShop、ScienceWorld、Mind2Web、AMC23 五个基准**，组合成 6 组设置。其中 ARISE 是阿里同期在自进化路线上的另一项工作（rubric-grounded 的迭代自进化 RL），可对照阅读 → [[raw/articles/arise-rl-rubric-self-evolution-alibaba-xhs-2026|ARISE-RL 原始笔记]]。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

## CODA 协议：从峰值成绩到性能下限

**CODA（CO-evolution Drift Audit，协同进化漂移审计）** 在原有成功率之外引入两个指标，并与技能变化下的最差成功率共同构成评测三角：^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

| 指标 | 度量对象 | 回答的问题 |
|------|----------|------------|
| 一致率 CR（Consistency Rate） | 技能含义不变、表达改变时的**逐任务**成功/失败一致性 | 具体任务结果是否稳定（而非仅平均分接近） |
| 抗干扰率 RR（Resistance Rate） | 面对不完整或错误技能时的任务保持能力 | 原本不依赖技能也能完成的任务是否被干扰 |
| 最差成功率 | 所测试技能变化中的性能下限 | 技能一变，至少还能保留多少表现 |

三者分别补充**稳定性、抗干扰能力、性能下限**的信息。作者明确定位 CODA 为「发现并衡量问题的初步评测方案」，而非结论性基准：受控探针让高分背后的脆弱性变得可观察，同时表明现有方法在应对技能变化上仍有较大提升空间。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

## 对 Agent 工程的启示

**把技能库当作需要版本化与兼容性测试的依赖，而不是静态提示词集合。** 一旦技能更新频率高于模型重训频率（生产环境几乎必然如此），评测就必须覆盖「技能变更后」的分布，而不仅是训练时的固定搭配。这与 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] 和 [[concepts/evaluation-harness-design|评测 Harness 设计]] 强调的「评测集要覆盖部署分布漂移」同向，但把漂移源具体化为技能文本的表层变化。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

**对同义改写鲁棒是低成本的高价值验收项。** 工程上不需要复杂基准就能做第一道体检：把技能文档做语义等价重写，观察逐任务结果是否翻转；翻转比例高即说明模型学到了表面形式而非可迁移策略。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

**与安全侧的协同进化研究构成一对镜像。** SHE/SafeEvolve 关心「Harness–Policy 协同进化能否持续修复安全失败」→ [[entities/she-safeevolve-agent-safety-harness-policy-coevolution|SHE 与 SafeEvolve]]；CODA 关心「协同进化产生的高分能否在技能漂移后存活」。前者问进化是否有效，后者问进化的收益是否可携带，两者共同指向 [[concepts/harness-engineering-paradigm-shift|Harness 工程范式]] 里「评测必须与运行时变化对齐」的同一命题。^[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026.md]

## 关联

- [[concepts/skill-engineering-principles|Skill 工程原则]] — 技能设计层原则，本文补充「技能版本漂移下的可靠性」维度
- [[concepts/skill-formal-theory-survey|Skill 形式化理论综述]] — 技能的形式化视角
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] — 评测集覆盖与漂移
- [[concepts/evaluation-harness-design|评测 Harness 设计]] — 评测执行层设计
- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 可验证奖励强化学习]] — 协同进化训练侧背景
- [[concepts/harness-engineering-paradigm-shift|Harness 工程范式]] — 评测与运行时对齐
- [[entities/she-safeevolve-agent-safety-harness-policy-coevolution|SHE 与 SafeEvolve：Harness–Policy 协同进化]] — 安全侧的镜像工作
- [[raw/articles/arise-rl-rubric-self-evolution-alibaba-xhs-2026|ARISE-RL 原始笔记]] — CODA 实验覆盖的自进化方法之一
- 论文：arXiv 2610.20845v1

→ [[raw/articles/alibaba-token-foundry-coda-skill-policy-coevolution-reliability-2026|原文存档]]
