---
title: "Agent 自进化全景图：从 Skill 到模型权重的二维分类（AliExpress 技术）"
created: 2026-10-25
updated: 2026-10-25
type: entity
tags: [agent, self-evolution, skill, harness, memory, taxonomy]
sources: [raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026]
confidence: 0.7
reviewed: 2026-10-25
review_verdict: keep
review_category: tech
---

# Agent 自进化全景图：从 Skill 到模型权重的二维分类（AliExpress 技术）

> **Background**：AliExpress 技术朱灵子 2026-09-10 的综述，用「进化什么 × 如何进化」两问，把 Trace2Skill、SkillOpt、SkillRL、GiGPO、AgentEvolver、OPSD 等方法排进同一张地图。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

## 什么是自进化

起点是对「一次训练、静态部署」的否定：模型不会因用得久就变强。自进化指靠交互、反馈与自我重构获得能力增长的范式，能力来源从数据与算力换成交互与反馈。三条判定须同时成立：动态适应（闭环在运行期转动）、多维对象（prompt、记忆、工具、架构）、自主自驱（自己探索、反思、评估）。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

## 进化什么：Agent 系统 vs 模型权重

第一层 Agent 系统进化改模型外部资产：Skill、Harness、Memory、AFlow、Environment，即改即生效、可回滚，受上下文预算约束。第二层模型参数进化把经验写进权重：微调与 RL、自我归因、自蒸馏，最持久、可跨会话迁移，但不可逆。两层有先后——参数路径被放在「Harness 经验稳定且通用之后」，因内化先验可免去反复支付编排开销，并迁移到上下文覆盖不到的任务与用户。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

## 如何进化

**Skill 轨迹生成（Trace2Skill）** 走「先观察、后总结」：用初始 Skill 跑大量轨迹并按成败分类，让分析子智能体并行给每条轨迹产出补丁，再分层合并去冲突。意图是避开单点更新的方向偏差；文章报的吞吐是 122B 模型生成 200 条 50 轮以上轨迹不到两小时。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

**群体进化与协同**。EvoSkill 拆出执行者、提案者、搭建者三个子智能体，提案者负责在失败时定位根因、成功时总结模式。SkillOpt 把 Skill 文本当「可训练的外部参数」：优化器小批量反思后产出增删替操作，配幅度限制、验证集门控与动量更新，等于文本空间 SGD。coEvoSkill 让验证器出题并升级难度，环境只给黑盒 pass/fail，把单边自省变成对抗性共同进化。SkillRL 让 RL 与 Skill 库互为燃料，成功轨迹蒸馏为策略、失败转为避坑指南。D2Skill 补上估值层：双粒度技能加事后效用信号，按效用检索与剪枝。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

**Harness 与元进化**。Harness 是围绕模型的运行时控制层，管上下文、工具、权限、执行、反馈与记忆，关键是时间尺度不对称：它可在部署期被低成本修订，权重不能。由此区分两件事——单任务里 loop 只需成功一次，学习却要跨窗口累积，harness 决定 loop 能保留什么。EvoAgentX 用 TextGrad 改 prompt、AFlow 搜拓扑、MIPRO 调工具与参数，三线并行逐代进化。再往上是元进化：HyperAgents 的 Meta Agent 能改 Task Agent 也能改自己，使改进策略本身成为优化对象。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

**参数侧三路**。反馈进化最直接：邮件场景用人工修正回复做 GRPO 验证奖励后训练，或用业务标注做对抗奖励的 RL 后训练。自我归因以 AgentEvolver 为代表：自我提问自造任务、自我导航用 ReMe 经验池复用、自我归因用 ADCA-GRPO 做轨迹级信用分配——LLM 回溯轨迹给每步打 GOOD/BAD 标签，量化为 ±1 过程奖励再与结果奖励融合。自蒸馏以 OPSD 为代表：同一模型分饰学生与教师（教师多看答案），最小化两者 JSD，梯度只回传学生。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

**支撑算法 GiGPO** 被 SkillRL 与 D2Skill 共用：保留 GRPO 无 Critic 的特性，靠环境「状态重访」先验把相同状态下的不同动作聚成组，在组内归一化折扣奖励，与轨迹级优势加权叠加后代入 clipped PPO，把「探索效率低」转成密集监督。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

## 与既有自进化框架的关系

与腾讯两篇是同一问题的不同切面。[[entities/agent-self-evolution-four-gear-flywheel-tencent-2026|四齿飞轮]] 答流程——一轮进化怎么跑完；本篇答对象与手段——可改哪些层、每层有哪些已验证方法。层级划分一致，但本篇把 Skill 单列成方法树，四齿飞轮把它并入 Harness 齿。[[entities/agent-self-evolution-flywheel-tencent-2026|腾讯飞轮]] 偏闭环动力学，[[entities/agent-self-evolution-evaluator-bottleneck|评估器瓶颈]] 指最脆一环是评估可信度——本篇为其举证：Trace2Skill 靠批量统计抵消单点偏差，SkillOpt 用验证集门控。验证内建于机制而非外挂，故缺量化评估时自进化会退化为盲改。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

## 适用边界

文章自认「可进化」到「有效进化」差距很大：方向不可控，单轮轨迹带偶然性，据单次更新会让 Skill 为迎合个别 badcase 而臃肿发散；质量不稳定，缺量化体系时无法区分真提升与噪声。两者指向同一前提：先有可信的反馈与评估体系。另有三点保留：它是综述而非实证，收益数字横向不可比；Memory 与 Environment 只列对象未展开；参数侧门槛是 Harness 经验已稳定，过早微调会把不稳定经验固化成难回滚的偏差。^[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026.md]

## 关联

- [[concepts/agent-self-improvement-loops]]：闭环定义与失败模式
- [[concepts/agent-harness-engineering-paradigm]]：Harness 作为运行时控制层
- [[concepts/agent-memory-architecture]]：本篇未展开的 Memory 对象之架构视角
- [[concepts/context-engineering]]：非参数进化的共同预算约束
- 同题异构：[[entities/agent-self-evolution-flywheel-tencent-2026|腾讯自进化飞轮]]、[[entities/agent-self-evolution-four-gear-flywheel-tencent-2026|四齿飞轮]]、[[entities/agent-self-evolution-evaluator-bottleneck|评估器瓶颈]]

→ [[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026|原文存档]]
