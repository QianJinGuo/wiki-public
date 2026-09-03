---
title: "AutoResearch 四种常见循环设计框架"
created: 2026-08-27
updated: 2026-08-30
type: entity
tags: [autoresearch, agent-loop, research-agent, framework, paperweekly]
sources: [raw/articles/autoresearch怎么设计四种常见循环一套通用框架]
confidence: 0.7
---

# AutoResearch 四种常见循环设计框架

> AutoResearch = 基模 + Agent Loop。当基模固定时，方法循环设计成为竞争的本质。本文是白白小白（PaperWeekly）提出的 AutoResearch 循环设计分类学——四种常见循环范式 + 一套可复用的通用分析框架。^[raw/articles/autoresearch怎么设计四种常见循环一套通用框架.md]

## 四种循环范式

| 循环范式 | 代表系统 | 核心特征 |
|---|---|---|
| 线性循环 Keep-or-Discard | Karpathy autoresearch（2025） | 单线程探索，好则保留坏则回退 |
| 并行分支探索 | — | 同时探索多个方向 |
| 失败经验结构化保存 | — | 记录失败原因避免死循环 |
| 长时程策略 | — | 短时预算易陷入局部最优的替代 |

**线性循环 Keep-or-Discard** 是最简单的设计：每次尝试一个想法，结果更好就保留，否则 git reset 回退。Karpathy 的 autoresearch 仅三个文件，循环逻辑由一个 Markdown 指令（program.md）定义，含「固定 5 分钟时间预算」这一关键约束——它迫使 Agent 思考哪些改动能在极短训练后产生可测量收益，淘汰需要长训练才见效的方案。^[raw/articles/autoresearch怎么设计四种常见循环一套通用框架.md]

该范式的结构性局限：①无法并行探索多个方向；②失败实验经验未结构化保存（可能反复尝试同一 idea 死循环）；③短时间约束容易让框架陷入局部最优。^[raw/articles/autoresearch怎么设计四种常见循环一套通用框架.md]

## 通用分析框架

当出现新的 AutoResearch 方法时，可用该分析框架直接推导其优劣势：从循环拓扑（线性/并行/分层）、失败处理（回退/记录/重试）、时间预算约束、以及人类介入程度四个维度审视。^[raw/articles/autoresearch怎么设计四种常见循环一套通用框架.md]

## 深度分析

### 竞争的本质：从基模转向循环设计

AutoResearch = 基模 + Agent Loop。当基模（底层模型）趋于同质、固定之后，方法循环设计便成为竞争差异化的本质来源——谁能更好地组织探索、反馈与记忆，谁就能在同一基模上取得更好的研究产出。这解释了为什么 Agent Loop 设计 会从一种实现细节上升为核心研究问题。

### 四种范式是搜索拓扑的演化谱系

四种循环并非并列选项，而是沿「搜索拓扑」维度递进演化的谱系：线性 Keep-or-Discard 每次只走一步、要么保留要么 git reset 回退；树搜索维护一棵解树，允许在任意历史节点重新出发，获得线性循环不具备的回溯能力；遗传进化池通过选择 + 突变（由 LLM 完成）维持种群多样性；异步多 Agent 则让多个独立搜索进程通过共享记忆间接协调。拓扑越复杂，单次实验的「失败成本」越低，但系统管理与解释成本越高。

### 反馈信号的粒度决定学习上限

反馈信号决定系统每次实验能学到多少。标量奖励只回答「好了多少」，不解释「为什么失败」，这也是 Karpathy 线性循环可解释性不足的根源；结构化指标提供多维评估；而 GEPA 用文本反馈取代标量奖励，让 LLM 阅读完整执行轨迹来诊断问题、归因原因，从而驱动更具针对性的突变。信息越丰富，下一步决策质量越高，但获取与处理的成本也随之上升——反馈粒度与成本之间存在根本权衡。

### 记忆架构与决策主体是扩展性的两翼

记忆架构决定系统能否从历史中学习：从无记忆、Git 历史、解树、文件系统池到知识图谱，结构化程度逐级提升。CORAL 将 [[concepts/agent-memory-architecture|共享持久记忆]] 拆为 attempts/notes/skills 三个目录，成为多 Agent 无需显式通信即可协调的基石。同时，决策主体从「人类硬编码搜索规则、LLM 仅作算子」逐步让渡给 Agent 自主判断（如 AI Scientist v2 抛弃公式化选择）。搜索能力的提升正沿着「拓扑 → 反馈 → 记忆 → 决策主体」四个维度同步展开。

## 实践启示

1. **从线性循环起步，用固定短时间预算约束 Agent。** Karpathy 仅三个文件、一个 Markdown 指令即可落地；「固定 5 分钟」的约束迫使 Agent 只保留能在极短训练后产生可测量收益的改动，天然淘汰需要长训练才见效的方案。小规模实验先从它开始。

2. **需要并行探索或回溯时，升级到树搜索并用 UCB 平衡探索与利用。** 树搜索把解空间限制在一条线性路径上，可在任意历史节点重新出发；但贪婪策略会让早期优秀 Draft 的子树「饿死」其他方向，UCB 公式通过给访问次数少的节点「好奇心加分」维持探索。

3. **用文本反馈补充甚至取代标量奖励。** 标量指标无法解释「为什么失败」。像 GEPA 那样先对候选做 rollout、记录完整执行轨迹，再让 LLM 阅读轨迹诊断归因，能显著提升突变方向的质量。

4. **为失败与经验构建结构化记忆，避免死循环。** 线性循环的一大缺陷是失败经验未结构化保存、可能反复尝试同一 idea。参照 CORAL 将记忆拆为 attempts（评估记录）/notes（观察反思）/skills（可复用流程），按需读取以避免上下文过载。

5. **多 Agent 场景用共享持久记忆协调，而非显式通信协议。** CORAL 让每个 Agent 在独立 git worktree 中运行、通过共享记忆间接协调——一个 Agent 的发现自然影响其他 Agent 的后续搜索，避免了显式通信带来的耦合成本。

6. **用四维框架评估任何新 AutoResearch 方法。** 面对新方法时，直接从搜索拓扑（线性/树/池/异步）、反馈信号（标量/结构化/文本）、记忆架构、决策主体四个维度解构，即可快速推导其优劣势，而不必逐行读代码。

## 关系与对比

- [[entities/autoresearch-ai-scientific-discovery-l0-l4-challengehub|AutoResearch L0-L4]] 从科学发现能力分级切入，本文从循环拓扑切入——视角互补
- [[entities/autoresearch-feedback-loop-self-improving-agents-introspection|自改进 Agent 反馈循环]] 聚焦内省反馈循环，本文覆盖更宽的四种范式
- [[entities/autoresearch-multi-agent-software|AutoResearch 多智能体软件]] 强调多 Agent 协作，本文侧重单循环设计拓扑
- [[entities/autoresearch-eval-agent-failure-meta-cognitive-loop-2026|评估与失败元认知循环]] 与「失败经验结构化保存」范式直接相关

## 补遗（2026-08-29）：失败处理的第五种范式——编辑计划而非重新规划

本文通用分析框架给「失败处理」列了回退 / 记录 / 重试三种。Polaris 的 Voyage 内核提供了第五种：**失败时不重生成计划，而是对现有计划发增量 plan edit**（add/update/obsolete），带硬不变量——每次 edit ≤8 个新步骤、只准动未完成步骤、不得在当前执行点之前插入、superseded 步骤标 obsolete 而不删除（历史完整）；plan edit 按**错误签名**限连续 2 次（换了错误即重置），防止同因循环 ^[raw/articles/polaris-voyage-research-platform-2026.md]。

配套的预算哲学补进「时间预算约束」维度：**限额花在进展上，不是重试次数上**——实验修复循环不设重试计数器，改用墙钟 phase 预算（时钟排除排队/审批/等待时间）；token budget 耗尽时 wrap-up 步骤仍可跑，其余 pending 标 obsolete ^[raw/articles/polaris-voyage-research-platform-2026.md]。与四种循环范式的关系：plan edit 正交于搜索拓扑——线性 keep-or-discard、树搜索、进化池改变的是「往哪搜」，plan edit 改变的是「搜不动时怎么改地图」。综合来源见 [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 涌现观点·观点三]]。

→ [[raw/articles/autoresearch怎么设计四种常见循环一套通用框架|原文存档]]
