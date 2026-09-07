---
title: "想让 Agent 在你睡觉时继续干活？先给它排好夜班"
created: 2026-07-02
updated: 2026-09-07
type: entity
tags: [agent, cron, task-scheduling, automation, agent-lifecycle]
sources: [agent-nightshift-cron-task-scheduling]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

## 摘要

本文将传统 `crontab` 和定时任务的概念扩展到 AI Agent 领域，探讨「Agent 夜班」——如何让 Agent 在用户离线时继续执行任务。^[agent-nightshift-cron-task-scheduling.md] 文章系统梳理了从 `/goal`、`/loop` 到 Routines 的能力分层，提出以 GOAL / STATE / EVIDENCE / PERMISSIONS 四文件为核心的交接框架，以及一张 Markdown 排班单把隐含信息摊开的工程实践。^[agent-nightshift-cron-task-scheduling.md] 核心观点是：让模型晚上多跑几小时并不难，难的是让它进入团队流程后第二天还能被人接手、复查和继续推进。^[agent-nightshift-cron-task-scheduling.md]

## 核心要点

- Agent 的工作单元正从「单次 prompt 调用」演变为「可排班、可交接、可检查的运行过程」，时间维度、成本维度和交接维度是关键增量^[agent-nightshift-cron-task-scheduling.md]
- `/goal` 管终点、`/loop` 管节奏、Routines 管长期调度，三者是从小到大的递进关系，不是互相替代^[agent-nightshift-cron-task-scheduling.md]
- 无人值守 Agent 必须具备四项运行时能力：保存现场、调用工具、检查结果、停在边界^[agent-nightshift-cron-task-scheduling.md]
- 四文件模式（GOAL → STATE → EVIDENCE → PERMISSIONS）是极简但有效的可交接工程实践，从 Markdown 文件开始即可落地^[agent-nightshift-cron-task-scheduling.md]
- 排班单表格把任务名、工作窗口、输入位置、完成条件、检查命令、权限边界、停机条件、早班负责人等信息提前摊开，防止任务膨胀和隐含假设^[agent-nightshift-cron-task-scheduling.md]
- 模型 API 的峰谷定价让时间排班从省钱技巧升级为系统架构考量，低成本打开了低风险实验空间^[agent-nightshift-cron-task-scheduling.md]
- 自动化程度越高，人工闸门越要前置；三类任务（不可逆动作、目标未定、事实源不稳定）应留在人手里^[agent-nightshift-cron-task-scheduling.md]
- 早上的 10 分钟验收流程（看越界→看证据→看 blocked→看沉淀）是晚间任务能否进入团队流程的关键^[agent-nightshift-cron-task-scheduling.md]

## 深度分析

### 从 Prompt 到 Loop：Agent 工作单元的重定义

文章提出的核心转变是 Agent 的工作单元正在从「单次 prompt 调用」变成「可排班、可交接、可检查的运行过程」。这个变化的意义不在于「让模型多跑几小时」，而在于**时间维度、成本维度和交接维度**的加入。当 `/goal` 解决「做到什么才算完」、`/loop` 解决「什么时候回来再看一次」、Routines 解决「任务能不能独立跑」时，Agent 从对话工具变成了可生产部署的工作进程^[agent-nightshift-cron-task-scheduling.md]。

这与 Harness Engineering 中「状态边界要清楚」的主张一脉相承——任务不能只存在于一轮聊天里，现场、产物、目标和后续动作都要能留下来。工作现场要能跨时间存在，状态边界也要能在无人值守时生效^[agent-nightshift-cron-task-scheduling.md]。

### 无人值守任务的四个运行时能力

文章提炼了 Agent 在无人值守模式下必须具备的四项基础能力：**保存现场**（知道上次做到哪里）、**调用工具**（不只生成文本还执行命令）、**检查结果**（拿出可验证的测试、截图、diff）、**停在边界**（遇到不可逆动作或连续失败时停止）。这四条既是对 Agent 能力的约束，也是工程保障——它们确保无人值守不会变成无人管控^[agent-nightshift-cron-task-scheduling.md]。

尤其是「停在边界」这一条，与传统 cron 任务有本质区别。传统脚本到点就跑、跑完就结束；Agent 无人值守则需要在遇到不确定、权限不足或连续失败时主动停下来等人。blocked 不是坏事，一个好的长任务 Agent，要知道什么时候停下来等人，而不是把不确定包装成完成^[agent-nightshift-cron-task-scheduling.md]。

### GOAL / STATE / EVIDENCE / PERMISSIONS 四件套

文章提出了一套极简的工程实践：四份 Markdown 文件分别承载完成条件、交班状态、验收证据和权限边界。这种设计不是为了让 Agent「更聪明」，而是为了把隐含信息摊开——让 Agent 不必猜测哪些能做、哪些不能做，让人不必第二天从头翻聊天记录^[agent-nightshift-cron-task-scheduling.md]。

传统定时脚本只负责「到点执行」，而 Agent 引入的正是这套**可交接的状态管理**。四文件模式的深层价值在于：很多 Agent 任务失败，不一定是模型差一点，而是任务一开始就没有被整理成「能交接」的形状。目标散在脑子里，输入散在聊天里，权限靠默认理解，失败靠模型自己体会^[agent-nightshift-cron-task-scheduling.md]。

### 排班单：把隐含信息压成一张表

文章进一步将四文件信息压缩成一张可操作的 Markdown 排班单，包含十个字段：任务名、工作窗口、输入位置、完成条件、检查命令、证据位置、状态位置、权限边界、停机条件、早班负责人。这张表不高级，但它很管用^[agent-nightshift-cron-task-scheduling.md]。

文章还给出了团队实际使用的排班单范例：billing/refund 测试排班，工作窗口 22:30-07:30，最多处理 3 类测试，明确列出检查命令 `npm test -- billing/refund`、权限边界（可改测试不可改 schema 和密钥）、停机条件（同一项连续 3 次失败）^[agent-nightshift-cron-task-scheduling.md]。这种具体化让 Agent 从「模糊执行」变成「结构化交接」。

### 低峰窗口下的工程经济学与渐进自动化路径

模型 API 的峰谷定价（如 Qwen3.7-Max 在 off-peak 时段 Credits 倍率从 0.5x 降至 0.1x）使得时间排班从「省钱技巧」升级为系统架构考量。但文章指出，真正重要的不是便宜——而是**低成本打开了低风险实验空间**，让团队愿意让 Agent 在夜间处理那些「白天舍不得用昂贵 API 跑」的任务^[agent-nightshift-cron-task-scheduling.md]。

与此同时，`/loop` 有 7 天过期边界，Routines 仍是 research preview——过期机制本身就是安全设计。忘掉的 loop 如果一直跑，成本、权限和副作用都会变成问题。因此文章建议的渐进路径是：白天手动跑 → `/goal` 固化验收 → `/loop` 短周期看护 → 连续稳定后沉淀为 Routine。长期自动化更像一条小生产线，需要定期看良率、看坏件、看成本^[agent-nightshift-cron-task-scheduling.md]。

## 实践启示

1. **无人值守任务必须先有验收标准** — 没有明确完成条件的任务不适合交给 Agent 夜间执行；强目标应包含可测量的结束状态和约束条件。只有「我已经完成」一句不算完成，那更像是把复核成本转移给第二天早上的人^[agent-nightshift-cron-task-scheduling.md]。
2. **四文件模式可零成本实施** — GOAL → STATE → EVIDENCE → PERMISSIONS 从 Markdown 文件开始，不需要复杂系统，却能从根本上改善 Agent 任务的可交接性。
3. **渐进的自动化路径更可靠** — 白天手动跑 → `/goal` 固化验收 → `/loop` 短周期看护 → 连续稳定后沉淀为 Routine，避免一步到位的全自动化陷阱。自动化会放大流程里的确定性，也会放大流程里的含糊^[agent-nightshift-cron-task-scheduling.md]。
4. **让 Agent 运行时有「刹车机制」** — 权限分层（可自动做 / 可生成待审 / 需人工确认 / 禁止）和停机条件（连续失败 N 次即停止）是无人值守的安全底线。无人值守任务越普及，错误也越容易批量化^[agent-nightshift-cron-task-scheduling.md]。
5. **白天的工作重心转向「任务拆解」** — 团队下班前准备 GOAL/STATE 文件的过程，本身就是一种知识显性化实践。白天人最值钱的地方，不是陪 Agent 等输出，而是把任务切成能被 Agent 执行和验证的形状^[agent-nightshift-cron-task-scheduling.md]。
6. **早上的 10 分钟验收不可省略** — 先看越界（改了什么不该改的），再看证据（测试、diff、来源是否可复核），然后看 blocked 项（停下来等人不是坏事），最后看有没有值得沉淀为 Routine 的流程^[agent-nightshift-cron-task-scheduling.md]。

## 关联

- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关概念: [[concepts/loop-engineering-methodology|Loop Engineering]]
- 相关实体: [[entities/codex-goal-six-hour-run|Codex 六小时长跑]]
- 相关实体: [[entities/claude-code-loop-engineering-guide|Claude Code Loop Engineering]]
- 相关实体: [[entities/ai-agent-loops-claude-code-codex|AI Agent Loops]]
- 相关: Agent 架构、定时任务、任务调度

→ [[raw/articles/agent-nightshift-cron-task-scheduling|原文存档]]
