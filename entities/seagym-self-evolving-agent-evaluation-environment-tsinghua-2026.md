---
title: "SEAGym: 自进化Agent评测环境 — 清华大学"
created: 2026-07-15
updated: 2026-11-07
type: entity
tags: [harness, self-improving-agent, evaluation, benchmark, tsinghua, agent-evaluation, self-evolving]
sources: [raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# SEAGym: 自进化Agent评测环境 — 清华大学

> **Background**: 本文基于机器之心的报道，系统梳理了清华大学团队提出的 SEAGym 评测环境。该工作与翁荔关于 Harness Engineering for Self-Improvement 的论述相呼应，但聚焦于检测基础设施层面。^[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026.md]

## 核心动机

现有 Agent benchmark 大多面向静态系统：给定一个固定 Agent，在一组独立任务上运行，然后报告最终成功率。这种评测方式无法回答 harness evolution 中更关键的问题：一次更新到底改进了什么？提升是否能迁移到未见任务？是否只是过拟合近期反馈？是否遗忘了旧能力？是否引入了更高成本或运行时不稳定？^[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026.md]

## SEAGym 框架

SEAGym 将自进化 Agent 形式化为一个和强化学习算法训练过程对齐的评测过程。每个 Agent snapshot 表示为 (M, H)，其中 M 是固定的基础模型和不可变运行组件，H 是可更新的 harness state（包括 prompts、memories、skills、tools、middleware、runtime configuration 等）。^[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026.md]

在每一步中，环境采样一批训练任务，Agent 在这些任务上执行，产生轨迹和反馈，然后根据自身的更新规则修改 harness：H' = update(H, trajectories, feedback)。SEAGym 不规定具体的更新算法，只需通过统一的 rollout/update interface 接入。^[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026.md]

## 多视角评测体系

SEAGym 的关键设计是将传统数据 split 与评测 view 区分开来：^[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026.md]

- **Train batch**：提供 Agent 更新所需的轨迹和反馈；
- **Update-validation**：冻结中间 snapshot，观察更新过程是否带来阶段性提升；
- **ID transfer**：测试更新是否能迁移到同分布但未见过的任务；
- **OOD transfer**：测试更新是否能迁移到分布外任务；
- **Replay**：回放旧任务，检查是否出现遗忘或回归；
- **Cost records**：记录 token、工具调用、运行时间和更新成本。

## 关键实验结果

AHE 在 validation、ID 和 OOD 三个视角上都取得了提升（validation 40.0 -> 57.1，ID 40.0 -> 49.1，OOD 22.5 -> 28.8），而 ACE 和 TF-GRPO 的提升更局部化。TF-GRPO 在 OOD 上甚至下降 0.1 个百分点——说明只在 validation 上提升是不够的。^[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026.md]

Batch 多样性 > Batch 大小：10 diverse 任务在 ID 和 OOD 上均达到最佳，20 less diverse 表现更差。在自进化场景中，不是「练得越多就越好」，而是「练得越广越好」。^[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026.md]

## 论文信息

- 论文：SEAGym: An Evaluation Environment for Self-Evolving LLM Agents
- 机构：清华大学
- arXiv：https://arxiv.org/abs/2606.17546
- 代码：https://github.com/antropy-research/SEAGym^[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026.md]

## 深度分析

### 设计本质：从「快照打分」到「过程观测」

传统 benchmark 回答的是「这个 Agent 现在有多强」，SEAGym 回答的是「这个 Agent 是怎样变强的」。它把自进化过程形式化为与强化学习训练对齐的评测过程：环境采样训练任务 → Agent 执行产生轨迹与反馈 → Agent 用自身更新规则修改 harness（H' = update(H, trajectories, feedback)）。关键在于 SEAGym 不规定更新算法，只要求通过统一的 rollout/update interface 接入——这使得 ACE（积累 skillbook）、TF-GRPO（更新 experience/context store）、AHE（直接编辑 prompts/tools/middleware/runtime）这些粒度截然不同的方法能在同一协议下比较。借助 harbor 包管理器（类似 apt-get 安装 benchmark），Terminal-Bench 2.0、HLE 等任务源被统一声明接入，静态 benchmark 由此被组织成自进化过程中的 train batch。

### (M, H) 分解：把「进化」变成可测量的对象

每个 Agent snapshot 被分解为 (M, H)：M 是冻结的基础模型与不可变运行组件，H 是可更新的 harness state。这个分解的方法论价值在于把「自进化」从模糊的整体印象变成明确定义、可 diff 的对象——测量的不再是「模型变强了吗」，而是「harness state 的每一次更新贡献了什么」。当模型权重冻结时，validation/ID/OOD 上的任何变化都可归因于 H 的修改，这为 harness engineering 提供了受控实验的基础。这也解释了 SEAGym 为何要保存每个阶段的 snapshot、trajectory、harness diff 和 metric records：诊断对象正是 H 的演化史。

### 多视角评测 = 自进化的「鉴别诊断」

SEAGym 把传统数据 split 拆成六个评测视角，各自回答一个不同的临床式问题：Update-validation 问「更新有没有阶段性效果」；ID transfer 问「效果能否迁移到同分布未见任务」；OOD transfer 问「能否泛化到分布外」；Replay 问「是否遗忘了旧能力」；Cost records 问「付出了什么代价」。实验证明这些视角不可互相替代：TF-GRPO 在 validation 上提升 6.5 个百分点，OOD 却下降 0.1；一个 snapshot 完全可能因一次错误的 middleware 修改而短暂变强后崩溃。只看最终 leaderboard 分数，这些动态全部不可见——这是从「单一分数」到「演化曲线族」的评测范式转变。

### 结果揭示的自进化规律

三组方法的对比给出两条可复用规律。其一，validation 提升不等于稳定泛化：AHE 在三个视角全面提升（validation 40.0→57.1，ID 40.0→49.1，OOD 22.5→28.8），而 ACE、TF-GRPO 的提升更局部化，说明窄域积累的收益难以外溢。其二，batch 多样性压倒 batch 大小：10 个高多样性任务在 ID 和 OOD 上均优于 20 个低多样性任务——「练得越广越好」而非「练得越多越好」。两者共同指向：自进化的质量取决于更新所接触经验的覆盖面，而非反馈量；这与「数据多样性决定泛化上界」的机器学习直觉在 harness 层面再次成立。

## 实践启示

1. **给自进化系统建立多视角评测，而非单一 leaderboard 分数。** 任何会随使用修改自身 harness（prompts、memory、tools、middleware）的系统，都应同时观测 update-validation、ID transfer、OOD transfer、Replay 与 Cost；TF-GRPO「validation 涨、OOD 跌」表明单一分数会系统性掩盖泛化退化。参见 [[concepts/agent-evaluation-benchmark-frameworks]] 与 [[concepts/eval-surface-rotation]]。
2. **用 (M, H) 分解隔离归因。** 在自家 harness 迭代中冻结基础模型，把所有可变物（prompt、skill 库、memory、工具配置）显式纳入 H 并做版本化 diff，才能把性能变化归因到具体 harness 修改，而不是笼统的「模型变强了」。参见 [[concepts/agent-harness-engineering-paradigm]]。
3. **把 Replay 当作回归测试。** 每次 harness 更新后回放旧任务集，如同软件工程的 regression suite；没有 Replay 视角的「自进化」可能在悄悄遗忘——最终版本整体得分更高，却丢掉了原本能解决的任务。
4. **训练 batch 追求广度而非数量。** 为 skill/memory 积累挑选任务时，优先提高任务多样性（不同工具链、不同失败模式），10 个多样任务胜过 20 个相似任务；「练得越广越好」直接指导经验池的构建策略。
5. **记录成本，防止「涨分但更贵」。** token 消耗、工具调用次数、运行时间与更新成本应与得分一同入账——一次 harness 更新若以双倍成本换取 2 个百分点，在成本视角下可能是负收益。
6. **优先广义 harness 编辑而非窄域经验堆积。** AHE 直接编辑 prompts、tools、middleware 和 runtime 行为，在 validation/ID/OOD 三个视角全面领先只积累 skillbook 或 context store 的方法；[[concepts/ahe-agentic-harness-engineering]] 的路线提示：自进化的杠杆在整条 harness，而不只在 prompt 层。另见 [[concepts/agent-self-improvement-loops]]。


## 相关实体

- [[entities/lilian-weng-harness-engineering-self-improvement|Lilian Weng Harness Engineering for Self-Improvement]]
- [[entities/harness-engineering-self-improvement-survey-lilian-weng|Harness Engineering for Self-Improvement 研究全景]]
- [[entities/agent-self-improvement-six-mechanisms|Agent Self-Improvement 六种机制]]
- [[entities/agentscope-builder-enterprise-self-evolving-agent-harness|AgentScope: 企业级自进化 Agent Harness]]
- [[entities/areal-2-agentic-rl-online-learning-self-evolving|AREAL-2: Agentic RL 在线学习与自进化]]

→ [[raw/articles/seagym-self-evolving-agent-evaluation-environment-tsinghua-2026|原文存档]]
