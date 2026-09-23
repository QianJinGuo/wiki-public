---
title: "Claude Code Loop Engineering 完整攻略"
created: 2026-07-01
updated: 2026-09-24
type: entity
tags: [claude-code, loop-engineering, agent-workflow, boris-cherny, best-practices]
sources: [raw/articles/claude-code-loop-engineering-guide-tutuangi-2026]
confidence: 0.92
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Claude Code Loop Engineering 完整攻略

基于兔兔AGI（技术极简主义）的 Loop Engineering 实战指南，系统讲解如何设计让工作自己推进的循环系统。^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## 核心观点

Boris Cherny （Claude Code 负责人）：^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]


> I'm no longer prompting Claude. I'm just running a loop that prompts him and then thinks about what to do next. My job is to write loops.

身份转变：从「输入提示词的人」变成「设计提示词系统的人」。^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## 演进三阶段

| 年份 | 范式 | 核心问题 |
|------|------|----------|
| 2024 | Prompt Engineering | 一句话怎么问得好 |
| 2025 | Multi-Agent Orchestration | 多个任务怎么分派得开 |
| 2026 | Loop Engineering | 整个流程怎么自己转得动 |

^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## Loop 三种形态

### 1. 单 Agent 循环

研究 → 起草初稿 → 对照目标检查 → 修复薄弱区域 → 重复，直到超出标准^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]


一个 Agent 反复迭代，改到满意才交付。^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

### 2. 多 Agent 舰队循环（Fleet Loop）

Orchestrator → Specialist → Sub-agent 的树状结构，构成「发现 → 规划 → 执行 → 验证」的持续闭环。^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

### 3. Open-loop vs Closed-loop

| 维度 | Open-loop（开环） | Closed-loop（闭环） |
|------|------------------|-------------------|
| 自由度 | 高，可自主探索多条路径 | 低，在预设路径内运行 |
| Token 消耗 | 极高 | 可控 |
| 适用场景 | 探索性任务 | 执行性任务 |
| 当前成熟度 | 成本是硬伤 | 真正产出成果的模式 |

**先跑稳 Closed-loop，再考虑 Open-loop 做探索。**^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## 理论基础

### ReAct：推理 + 行动

思考 → 行动 → 观察 → 思考 → ... → 最终答案^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]


ReAct 是 Loop 的引擎，每一次「改 → 跑 → 看 → 改」就是一次循环迭代。^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

### Reflexion：失败即燃料

任务执行 → 失败 → 用自然语言反思失败原因 → 存入记忆 → 下次尝试时使用^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]


Reflexion 是 Loop 的学习机制，落地形式是 SKILL.md 和 progress.txt。^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## Claude Code 内置 Loop 工具

| 命令 | 触发方式 | 适用场景 | 版本 |
|------|---------|---------|------|
| /loop | 时间驱动 | 轮询检查类任务 | — |
| /goal | 目标驱动 | 有明确完成标准的任务 | v2.1.139 |
| Dynamic Workflows | AI 自编工作流 | 超大规模复杂任务 | v2.1.154 |

从 /goal 开始试最不容易出错。^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## 双层结构

| 层级 | 范围 | 目的 | 状态维护 |
|------|------|------|----------|
| Inner Loop | 单任务内 | 提升任务可靠性 | 上下文窗口内 |
| Outer Loop | 跨会话 | 随时间持续改进 | 持久化文件（SKILL.md、progress.md） |

Inner Loop 决定这一轮跑不跑得通，Outer Loop 决定下一轮还踩不踩同一个坑。^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## 「5+1」工程组件（Addy Osmani）

| 组件 | 作用 | 关键提醒 |
|------|------|----------|
| Automations | 让 Loop 自己跑起来 | 必须有停止条件 |
| Worktrees | 多 Agent 并行时避免文件冲突 | 只解决机械冲突 |
| Skills | 项目知识不丢失 | 没有 Skills 每轮都在重新理解 |
| Plugins | 让 Agent 接入真实工具链 | 权限边界要提前锁死 |
| Sub-agents | 分离生成与验证 | 高风险环节才启用 |
| + Memory | 跨会话不遗忘 | 没有 Memory 每次都是从头再来 |

^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## 四个设计原则

1. **从小闭环开始**：先跑通一个「10 分钟内能验证」的小 Loop
2. **每个 Loop 必须有停止条件**：验证通过 / 达到上限 / 标记挂起
3. **让 Outer Loop 形成经验复利**：每次结束花 30 秒更新 lessons.md
4. **认知要跟得上代码**：定期 Review Loop 产出，设计者对输出质量负责

^[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026.md]

## 金句

> 设计循环，而不是提示词。

→ [[raw/articles/claude-code-loop-engineering-guide-tutuangi-2026|原文存档]]

---
## 深度分析

### 为什么 Loop Engineering 在 2026 取代了 Prompt Engineering

三段演进的本质是「优化的对象」逐层上移：Prompt Engineering 优化单次问答的质量，Multi-Agent Orchestration 优化任务的分派结构，Loop Engineering 则把整个流程本身当作被设计的对象。翻转的动因是「人肉循环」的不可持续——传统模式下每一轮迭代都靠人手动触发，人名义上是导演，实际上被卡在循环里当监工。Loop Engineering 把触发、审查、判断、继续这一整套迭代逻辑交给系统，人的工作从「写提示词」退到「写让提示词自己生成的循环」。这一身份转变与 [[concepts/harness-engineering-framework|Harness Engineering]] 的思路同源：竞争的焦点不再是模型能力，而是围绕模型搭建的环境。

### ReAct 是引擎，Reflexion 是学习机制

ReAct 的「思考 → 行动 → 观察」交替架构解释了单轮迭代为什么能推进：每一次「改 → 跑 → 看 → 改」就是一圈循环。但只有引擎的系统会在同一个坑上反复摔倒——Reflexion 补上了失败处理：把失败用自然语言说清楚，存入记忆，供下一次尝试引用。落到工程上，这对组合分别对应 Inner Loop 的自驱验证行为和 Outer Loop 的持久化经验文件（SKILL.md、progress.md）。引擎没有学习机制，Loop 只是机械重复；学习机制没有引擎，反思无从产生。两者耦合才构成一个能随运行次数变聪明的系统，这也是 [[concepts/agent-self-improvement-loops|Agent 自改进循环]] 的理论起点。

### Inner/Outer Loop 的复利机制

Inner Loop 决定这一轮跑不跑得通，Outer Loop 决定下一轮还踩不踩同一个坑——这句话点出了复利的来源。Inner Loop 的产出是可靠交付（写测试、跑测试、修边界、绿灯后才算完成）；Outer Loop 把每轮结束时的经验教训写入持久化文件，下一轮启动时先读再干。单次「更新 lessons.md 花 30 秒」的投入极低，但它改变的是后续所有轮次的起点：Loop 与普通脚本的本质区别不在于自动化，而在于系统状态随运行次数单调递增。Context Engineering 在这里的角色是设计「哪些信息值得跨会话保留」——记忆写错了，复利就变成了复亏。

### 「5+1」组件是一条依赖链而非清单

把 Addy Osmani 的六个组件平铺看待会低估其结构：它们之间存在明确的依赖次序。Automations 让循环跑起来（但没有停止条件就是吞噬 Token 的黑洞）；跑起来后多 Agent 并行需要 Worktrees 隔离机械冲突；要让每轮「重新理解项目」的成本归零，必须有 Skills 沉淀项目知识；要接入真实工具链则需要 Plugins，且权限边界要提前锁死；高风险环节才引入 Sub-agents 分离生成与验证；最后 Memory 把这一切串成跨会话的连续进程。砍掉中间任何一环，上游组件的投入都会被下游的遗忘吃掉——这正是 [[concepts/loop-engineering-methodology|Loop Engineering 方法论]] 强调「组件齐备才成系统」的原因。

### Open-loop 与 Closed-loop 的经济学

两种形态的取舍表面上是自由度之争，实质上是预算经济学。Open-loop 让 Agent 自主探索多条路径，能产出启动时未定义的结果，但 Token 消耗随探索宽度爆炸，当下只有预算不设限的团队玩得起；Closed-loop 在人类预设的轨道内运行，路径受限所以预算可控，是当前真正产出成果的模式。原文给出的次序判断值得记住：先跑稳 Closed-loop，再考虑用 Open-loop 做探索。这个次序与 [[entities/claude-code-loop-types-official-taxonomy-four-modes|Claude Code Loop 官方分类]] 中按自主程度分级的思路一致——自由度是逐步放出去的，不是一次性交给的。

## 实践启示

1. **从「10 分钟内能验证」的小闭环起步**。不要一上来就搭建全自主 Agent 舰队；先定义一个清晰目标、设好停止条件的小 Closed-loop，跑通之后再逐步扩大范围。
2. **没有停止条件的 Loop 不是系统，是 bug**。每个循环必须有三种终止路径之一：验证通过（测试绿灯、lint 零告警）、达到迭代上限（自动挂起并通知人）、标记挂起（Agent 判断超出能力，附上分析转 needs-human）。
3. **用 /goal 作为第一个 Loop 工具**。Claude Code 内置的 /loop、/goal、Dynamic Workflows 三者中，/goal 目标驱动、有明确完成标准，最不容易出错；Dynamic Workflows 留给超大规模编排。
4. **把每轮结束的 30 秒当作最划算的投资**。花 30 秒更新 lessons.md（遇到什么问题、怎么解决的），下一轮的 Agent 就能直接跳过这个坑——这是 Outer Loop 复利的最低成本入口。
5. **生成者与验证者必须分离**。最危险的结构是让写代码的 Agent 自己验证自己的代码；在高风险环节引入 Generator/Evaluator 分工，普通 CRUD 不必上这个复杂度。
6. **Loop 是杠杆，会同时放大判断力和缺席**。设计者不能「让它在后台跑就不管了」——定期 Review Loop 的产出和它的经验教训质量，最终对输出负责的仍然是设计 Loop 的人。控制权如何分阶段移交，可参考 [[entities/claude-code-loop-control-rights-four-levels|Claude Code Loop 控制权四级]] 与 [[concepts/context-engineering|Context Engineering]]。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

