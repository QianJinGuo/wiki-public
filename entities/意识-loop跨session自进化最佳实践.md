---
title: "意识×Loop：AGENTS.md/MEMORY.md/USER.md 三层文件驱动的跨 Session Loop 自进化"
type: entity
created: 2026-07-10
updated: 2026-09-13
tags: [agent, loop-engineering, harness-engineering, memory, self-evolution, agents-md, alibaba]
rating: v9c8
sources:
  - raw/articles/意识-loop跨session自进化最佳实践
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 意识×Loop：AGENTS.md/MEMORY.md/USER.md 三层文件驱动的跨 Session Loop 自进化

Loop Engineering 的一个致命问题：一次会话结束，Loop 就"死了"。张心翮（阿里技术）提出了"**意识×Loop**"框架，用三个意识文件（AGENTS.md / MEMORY.md / USER.md）实现跨 Session 自进化。^[raw/articles/意识-loop跨session自进化最佳实践.md]

## 意识层三件套

| 文件 | 定位 | 核心机制 |
|------|------|----------|
| **AGENTS.md** | Proactive rules / evals | Agent 每次输出前自查的硬约束清单。约 15 条，超过 20 条遵从率下降 |
| **MEMORY.md** | Passive lessons / state | 本次会话踩过的坑、被纠正的判断。跨 Session 自动加载 |
| **USER.md** | Judge preferences | 个人的判断标准、输出味道偏好 |

每次会话开启时自动加载，每次会话中自动沉淀更新。^[raw/articles/意识-loop跨session自进化最佳实践.md]

## 三层对齐体系

| 层次 | 核心问题 | 承载 | 约束强度 |
|------|----------|------|----------|
| evals 对齐 | Agent 该主动做什么 | AGENTS.md | **硬约束**（客观正确性） |
| 场景对齐 | 给谁看、怎么被消费、读完做什么决策 | Session prompt | 中等 |
| judge 偏好 | 我的味道 | USER.md | **软偏好**（表达方式） |

分层承载，各归各的文件，绝不能混在一起。硬约束永远优先于软偏好。^[raw/articles/意识-loop跨session自进化最佳实践.md]

## MEMORY→AGENTS 升格机制：人肉阀门

MEMORY 允许试探性表达（"这次学到"），AGENTS 是硬约束（"以后都要"）。从 MEMORY 升格到 AGENTS 必须人肉做——自动升格一旦把偶发当成通用规律，错误规则会污染后续所有 Loop。样本量不够时概括出的"规律"是噪声不是信号。^[raw/articles/意识-loop跨session自进化最佳实践.md]

## Loop 六大框架的落地映射

| 框架 | 调研场景映射 |
|------|-------------|
| Automations | 会话结束时自动触发总结、追加 MEMORY |
| Worktrees | 上下文隔离——并行分析强制从原点起步，防偏见传染 |
| Sub-agents | 正反对抗——专门找支持/反例证据，审校独立子 Agent 避确认偏差 |
| State | 从"记进度"扩展到"行为塑造"——AGENTS+MEMORY+USER |

## 核心判断

- **taste 是 Loop 的准备动作，不是产出**：如果对 topic 没有阅读积累，连 evals 都写不出来。"Loop 让工作变轻松"是真的，"Loop 让新手变专家"是幻觉。^[raw/articles/意识-loop跨session自进化最佳实践.md]
- Loop Engineering 90% 讨论在 AI Coding，但 eval 越难写的场景，搭 Loop 的边际收益越大
- 不能自动化的位置：MEMORY 升格 AGENTS（松变紧可能带偏）、最终硬结论下判（个人信用）


## 深度分析

### Session 边界是 Loop 工程真正的天花板

Loop Engineering 最容易被忽略的失效点不在模型能力，而在状态存活的时间尺度：一次会话结束，调好的 evals、被纠正的判断、踩过的坑全部归零，Loop 退化成一段更花哨的 prompt。把状态外置成可读写的文件（AGENTS.md / MEMORY.md / USER.md），本质是把 state 从会话内存搬到磁盘，Loop 才有了"上一轮"这个概念——这是跨 Session 自进化的最低门槛，也和 [[concepts/context-management-agent-systems|上下文管理]] 同层：真正稀缺的不是窗口大小，而是能被下一轮复用的判断。^[raw/articles/意识-loop跨session自进化最佳实践.md]

### 硬约束与软偏好必须分层承载

硬约束（客观正确性，如"绝不能出现什么"）靠遵从率衡量，软偏好（表达味道，如"厌恶 AI 味"）靠人读起来是否舒服判断——两者的度量方式和失效方式都不同。混进同一个 prompt，模型无法为两类指令分配不同权重，结果通常是味道压过硬约束：风格化表达在生成时更即时、更局部，而规则性约束需要跨整段输出持续在场。AGENTS.md 约 15 条、超过 20 条遵从率下降这条经验值，正是稀释效应的定量证据：清单每长一条，每条被真正执行的概率就低一分。^[raw/articles/意识-loop跨session自进化最佳实践.md]

### 人肉阀门：小样本里概括出来的只有噪声

MEMORY 允许试探性表达（"这次学到"），AGENTS 是硬约束（"以后都要"），二者之间的升格必须人肉完成。理由不是人比模型聪明，而是统计上的：MEMORY 每条多是 n=1 的观察，从 n=1 概括"通用规律"得到的是噪声不是信号。一旦自动升格把偶发当成规律，错误规则就固化进优先级最高的那份文件，此后每一次 Loop 都继承它——污染不是一次性的，而是随轮次复利。这是 [[concepts/agent-self-improvement-loops|Agent 自进化循环]] 里最该被人工守住的那道闸门。^[raw/articles/意识-loop跨session自进化最佳实践.md]

### taste 是写 eval 的前提，Loop 只放大已有判断力

evals 的写法本质是把既有判断标准显式化成清单；对某个 topic 没有足够阅读积累，就连"绝不能出现什么"都说不出来，也就无从写下任何一条 eval。所以"Loop 让工作变轻松"是真的——它放大已有能力；"Loop 让新手变专家"是幻觉——它不制造判断力。反过来看，eval 越难写的场景，Loop 的边际收益越大：单人 1 天完成 61 家公司 3C 深度调研加横向汇总、产出 12 万字，价值不在"快"，而在于调研这类规范模糊、判断密集的任务里 [[concepts/evaluation-harness-design|评测体系设计]] 与子 Agent 对抗、worktree 隔离的组合收益最高，而 Loop Engineering 90% 的讨论仍停留在 AI Coding。^[raw/articles/意识-loop跨session自进化最佳实践.md]

## 实践启示

1. **AGENTS.md 先写 5 条，稳定在 15 条左右，硬上限 20 条**——超过 20 条遵从率下降，清单会因稀释而整体失效；宁可少写、每条都真实被执行。
2. **永不自动升格 MEMORY→AGENTS**：偶发错误一旦被提升为硬约束，会污染其后的每一次 Loop，且优先级最高、最难回滚；升格动作只由人执行。
3. **硬约束与软偏好分文件**：客观正确性进 AGENTS.md，输出味道进 USER.md，别把"我必须做什么"和"我希望读起来像什么"塞进同一个 prompt。
4. **每个 Session 结束花 2 分钟写 MEMORY，主动纠错后立刻写**；每周再扫一遍，已升格进 AGENTS 的删除、不再复现的归档——记忆文件只增不减会重新退化成噪声。
5. **用 Sub-agent 做正反对抗与独立审校**：一组专门找支持证据、一组专门找反例，审校另起独立子 Agent，避免确认偏差让结论自我强化。
6. **并行任务用独立 worktree 强隔离**：每份深度报告强制从原点起步，防止上一份结论的偏见传染下一份，代价只是多一份上下文。

## 与其他实体的关系

- → [[entities/loop-engineering-addy-osmani-challengehub|Loop Engineering：Addy Osmani 的六个框架]] — 本文的 Loop 六大框架理论基础
- → [[entities/agentic-loop-engineering-handbook-empirical-framework|Agentic Loop Engineering 工程手册]] — 17 种 Loop 工程化技术
- → [[raw/articles/意识-loop跨session自进化最佳实践|原文存档]]
