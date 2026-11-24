---
title: "Agent 自进化全景图：从 Skill 到模型权重的二维分类（AliExpress 技术）"
created: 2026-10-25
updated: 2026-11-15
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

## 深度分析

### 两个问题撑起一张可扩展的地图

综述的分类力不来自方法清单，而来自两个正交问题：进化什么定对象层级（Agent 系统资产 vs 模型权重），如何进化定手段谱系（批量归纳、群体协同、对抗验证、RL 共进化）。任何新方法都可先问「改的是哪一层、靠什么信号改」再入座，地图不随方法爆炸而失效。Skill 之所以被单列成方法树，在于它可被视为「可训练的文本参数」，[[concepts/skill-engineering-principles|Skill 工程原则]]中的优化纪律可整体移植自深度学习训练。

### 时间尺度不对称决定分层次序

Harness 可在部署期低成本修订、随时回滚，权重一旦更新就不可逆——这条不对称是全文最具工程含义的判断，推出两个推论：其一，非参数层是参数层的试验场，Skill 库里反复验证过的经验才值得固化进权重；其二，过早微调会把不稳定经验固化为难撤销的偏差，故参数路径被明确放在「Harness 经验稳定且通用之后」。正确的进化次序是先外后内，而非两线并行。

### 评估机制内建于方法而非外挂

对照各方法可见同一问题的不同解法——如何可信地判断「变好了」：[[entities/trace2skill-trajectory-distillation-agent-skills|Trace2Skill]] 用批量统计抵消单轨迹偶然性；[[entities/skillopt-microsoft-research-skill-training|SkillOpt]] 引入幅度约束、验证集门控与动量，把 SGD 纪律搬进文本空间；coEvoSkill 让验证器出题并升级难度，以对抗制造评估压力；D2Skill 用技能组与基线组的性能差构造事后效用信号。评估不是流程末端的质检，而是嵌在每个更新决策点上的门控条件。

### GiGPO 把状态重访从缺陷变成信号源

[[concepts/grpo-policy-optimization-2026|GRPO]] 在长序列稀疏奖励下的信用分配难题，被 GiGPO 用环境的一个「缺点」化解：策略不成熟时 Agent 频繁回到相同状态，传统视角视之为探索低效，GiGPO 却离线哈希聚合这些 Anchor State，在同状态的不同动作间做组内归一化，再与轨迹级优势加权叠加，得到步级密集监督——保住无 Critic 的低开销，同时把「探索效率低」变成免费的监督信号源。

## 实践启示

1. **先建评估，再启动进化。** 方向不可控与质量不稳定共享同一根因：缺可信反馈。没有量化评估体系时，据单次轨迹更新会让 Skill 为迎合个别 badcase 而臃肿发散，出现「越优化越差」。
2. **给 Skill 优化引入训练纪律。** 借鉴 SkillOpt：小批量反思、限制单次更新幅度、独立验证集门控、动量保留长期趋势——把文本编辑当作带学习率的参数更新来管理，而非随手改文档。
3. **批量归纳优于单点修补。** 学 Trace2Skill 先并行跑大量轨迹、按成败分类再分层合并补丁，用样本量对冲单轨迹的偶然性，而非看到一条失败就动手改 Skill。
4. **非参数与参数进化分先后。** 在 Skill/Harness 层把经验迭代到稳定且通用，再考虑微调或 RL 内化；与[[entities/self-distillation-agentic-rl-reality-check-grpo-2026|自蒸馏的现实检验]]相互印证——权重路径的价值在跨会话迁移，前提是内化的先验确实可靠。
5. **Agent RL 优先利用状态重访。** 若环境状态可重复访问，优先考虑 GiGPO 式双层优势估计，在不引入 Critic 的前提下获得步级信用分配信号，样本利用率随之提升。
6. **把改进策略本身纳入优化对象。** 参考 HyperAgents 的元进化：Meta Agent 既改 Task Agent 也改自己，持久化记忆、性能追踪等自创的通用改进策略可跨领域迁移，改进能力比单次改进更值钱。

> 综述的真正贡献不是罗列方法，而是给出可继续生长的分类坐标；也正因它是综述而非实证，收益数字横向不可比，[[entities/self-evolving-agents-survey|自进化综述]]类工作都需在具体场景中重新验证。

## 关联

- [[concepts/agent-self-improvement-loops]]：闭环定义与失败模式
- [[concepts/agent-harness-engineering-paradigm]]：Harness 作为运行时控制层
- [[concepts/agent-memory-architecture]]：本篇未展开的 Memory 对象之架构视角
- [[concepts/context-engineering]]：非参数进化的共同预算约束
- 同题异构：[[entities/agent-self-evolution-flywheel-tencent-2026|腾讯自进化飞轮]]、[[entities/agent-self-evolution-four-gear-flywheel-tencent-2026|四齿飞轮]]、[[entities/agent-self-evolution-evaluator-bottleneck|评估器瓶颈]]

→ [[raw/articles/agent-self-evolution-skill-to-weights-taxonomy-aliexpress-2026|原文存档]]
