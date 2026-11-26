---
title: "Claude Code Loop 四档控制权：自检→目标→定时→工作域"
created: 2026-07-08
updated: 2026-11-26
type: entity
tags: [claude-code, loop-engineering, control-rights, agent-autonomy, harness-engineering, turn-based, goal-based, time-based, proactive]
sources: [raw/articles/claude-code-loop-control-rights-four-levels]
confidence: 0.9
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Claude Code Loop 四档控制权：自检→目标→定时→工作域

> 架构师（JiaGouX）对 Claude Code 官方四类 Loop 的深度解读：不沿命令清单走，沿**控制权**这条线看——每往前走一档，人就少盯一层，系统就多承担一层责任。^[raw/articles/claude-code-loop-control-rights-four-levels.md]

## 控制权升级四档

| 阶 | Loop 类型 | 交出什么 | 人还握住什么 | 失败半径 |
|---|-----------|---------|-------------|---------|
| 1 | **Turn-based** | 检查权 | 下一轮是否继续 | 当前修改，可回滚 |
| 2 | **Goal-based** | 继续/停止权 | 目标定义和预算边界 | 一轮尝试，有上限 |
| 3 | **Time-based** | 触发权 | 节奏、范围和生命周期 | 跨时间边界，可能空跑 |
| 4 | **Proactive** | 工作域处置权 | 权限、审计、升级和降级 | 跨工具、分支、PR、团队 |

^[raw/articles/claude-code-loop-control-rights-four-levels.md]

## 核心工程原则

### 愿望句 vs 条件句
- **愿望句**：把判断留给模型（"帮我把页面优化一下"）
- **条件句**：把判断落到证据上（"首页 Lighthouse performance 提到 90 以上，最多尝试 5 次"）
- `/goal` 需要的是条件句 ^[raw/articles/claude-code-loop-control-rights-four-levels.md]

### 升级前先看失败半径
- 改错一段测试→可控。改错权限策略、生产配置、客户消息→性质完全不同
- **能降级，才敢升级**：跑不稳就从 proactive 降到 time-based；目标写不清就从 goal-based 降到 turn-based
- 证据越密，团队越敢让它多跑一层 ^[raw/articles/claude-code-loop-control-rights-four-levels.md]

### 渐进路线
1. **W1**：Turn-based（让 Agent 按 Skill 自检，区分"已修改"和"已验证"）
2. **W2**：加 `/goal`（逼团队把"完成"写清楚）
3. **W3**：Time-based（如每 20 分钟检查一次 PR 状态）
4. **W4+**：Proactive routine（补独立身份、最小权限、运行记录、成本上限、人工升级路径）

## 深度分析

四档 Loop 的本质不是功能清单，而是一条**委托梯度**：每往前一档，人让渡出一类判断——检查权（Turn-based）→ 继续/停止权（Goal-based）→ 触发权（Time-based）→ 工作域处置权（Proactive）——系统的自主半径与失败半径同步放大。^[raw/articles/claude-code-loop-control-rights-four-levels.md:13-22] 这与 [[concepts/harness-loop-architecture|Harness Loop 架构]] 中"循环即受控自主"的框架互为印证。

**评估器的证据边界**是 Goal-based 档最容易被误读的细节：评估模型只能读取已进入 transcript 的证据，因此"完成判定"的质量上限由"验证证据是否被显式写进记录"决定——口头声称"已修好"不构成可评估证据。^[raw/articles/claude-code-loop-control-rights-four-levels.md:32-38] 这把验收问题前置成了 [[concepts/evaluation-harness-design|评估面设计]] 问题。

**愿望句 vs 条件句**是停止条件的判别式：愿望句（"把页面优化一下"）把判断留回模型，等于没交出控制权；条件句（"Lighthouse performance ≥ 90，最多尝试 5 次"）把判断锚到可观测证据上，是"交出继续权、但握住验收标准"的关键机制。^[raw/articles/claude-code-loop-control-rights-four-levels.md:36-38]

**时间维度引入了新的成本模型**：Time-based 的关键边界包括 `/loop`（session 内间隔重复）与 recurring task 的 7 天过期边界，以及节奏三问——外部信号多久变化一次、空跑成本能否接受、发现问题最多处理到哪一步。触发权交出后，成本核算单位从"每次操作"变成"每次空跑"。^[raw/articles/claude-code-loop-control-rights-four-levels.md:40-47]

**Proactive 档的收敛前置条件**：输入是否稳定、身份与权限能否单独收敛、每次运行的决策与成本能否审计。工作域处置权只有在审计面闭合后才可交出——这与 [[concepts/autonomous-agent-systems|自主 Agent 系统]] 的治理要求一致。^[raw/articles/claude-code-loop-control-rights-four-levels.md:49-53]

## 实践启示

- **诊断顺序先于选型**：不要先挑命令，先问"这一档的检查/评估判断现在握在谁手里"，再决定交出哪一类权。^[raw/articles/claude-code-loop-control-rights-four-levels.md:13]
- **把降级路径写进设计**：跑不稳就从 Proactive 降到 Time-based，目标写不清就从 Goal-based 降回 Turn-based——降级是能力不是失败，能降级才敢升级。^[raw/articles/claude-code-loop-control-rights-four-levels.md:62-66]
- **用证据密度换授权档位**：把"已验证"与"已修改"区分开，并将验证步骤显式写进 SKILL.md；证据越密，团队越敢让它多跑一层。^[raw/articles/claude-code-loop-control-rights-four-levels.md:24-28]
- **以周为单位的渐进路线**：W1 只做 Turn-based 自检 → W2 加 `/goal` 把"完成"写成条件句 → W3 引入 Time-based 定时检查 → W4+ 沉淀为 Routine/Workflow；每档稳定后再升。^[raw/articles/claude-code-loop-control-rights-four-levels.md:55-60]
- **目标是接口工程而非自动化开关**：Loop Engineering 是 Agent 进入真实工作现场后自然长出的一层接口工程，其方法论详见 [[concepts/loop-engineering-methodology|Loop Engineering 方法论]]。^[raw/articles/claude-code-loop-control-rights-four-levels.md:68]

## 与既有实体的关系

- 与 [[entities/agent-vs-workflow-control-continuum-framework|Agent vs Workflow 控制权连续谱]] 互补——前者讨论宏观的 Agent/Workflow 选型，本文是 Claude Code 具体实现的控制权分层
- 补充 [[entities/loop-engineering-feedback-control-system|Loop Engineering]] 中"停止条件"的设计——条件句 vs 愿望句可以作为停止条件设计的决策框架
- 与 [[entities/claude-code-loop-engineering-guide|Claude Code Loop Engineering 完整攻略]] 互补——前者从功能/命令角度，本文从控制权/工程约束角度
- 四档控制权的渐进路线与 腾讯 Token 优化 的"约束金字塔"理念一致
- **核心洞察**：最容易被忽略的能力是**及时降级**——工程上能降级，才敢升级

---

→ [[raw/articles/claude-code-loop-control-rights-four-levels|原文存档]]

## 关联

- 同题异语种孪生页：[[entities/claude官方教你用-loop如何让claude-code上夜班的四个交接点]]（归并候选，提案卡 #11 批1）
