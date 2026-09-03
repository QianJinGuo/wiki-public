---
title: "Loop Engineering 主题地图 (MOC)"
created: 2026-06-15
updated: 2026-06-15
type: moc
tags: [moc, loop-engineering, harness, claude-code, peter-steinberger, boris-cherny, 2026, agent-paradigm, feedback-control]
sources: [raw/articles/loop-engineering-infoq-boris-cherny-peter-steinberger, raw/articles/loop-engineering-peter-steinberger-boris-cherny, raw/articles/loop-engineering-工程现场-ruofei, entities/loop-engineering-feedback-control-system, entities/loop-engineering-addy-osmani-challengehub, entities/harness-engineering, entities/claude-code-core-internals]
---

# Loop Engineering 主题地图 (MOC)

> "我的工作就是写循环" —— Boris Cherny, Claude Code 创建者, 2026-06
> "你不该再给编程 Agent 写提示词了。你应该设计一套循环机制" —— Peter Steinberger, OpenAI 龙虾之父, 2026-06

2026 年 6 月，由 Anthropic Boris Cherny + OpenAI Peter Steinberger 同时力挺的新范式——**Loop Engineering**——从「Claude Code 内部实践」快速演化为「全行业共识」。本 MOC 收口 4 个相关 entity + 3 篇原始资料 + 与 8 大相邻概念的连接。

---

## 核心论点（10 条）

### 1. 范式跃迁：从 Prompt 到 Loop

旧模式（人驱动循环）：人 → 提示 → Agent → 输出 → 人审查 → 人修正 → 重复
新模式（循环驱动）：人设定目标 → 循环运行 → Agent 发现 → 规划 → 执行 → 验证 → 迭代 → 完成

**关键洞察**：提示给 Agent 的是「指令」，循环给 Agent 的是「工作」。

### 2. Loop ≠ Cron

最常见误解是「Loop 就是 cron，每 5 分钟让 Agent 重做一次」——这种「open-loop」会失败，因为 Agent 会在循环中不断重复并自我确认错误。真正的 Loop Engineering 是 **closed-loop**：有 verifier、有 stop condition、有 rollback、有 human-in-the-loop。

### 3. Loop 的 6 个必备组件（若飞）

1. **自动触发**（cron, `/goal`, `/loop`）
2. **隔离工作区**（worktree, 临时分支）
3. **过程资产**（Skills, 规则, 模板）
4. **外部连接**（MCP, 插件, CLI）
5. **独立验证**（sub-agent/reviewer/测试）
6. **状态记忆**（plan.md, issue, 日志）

### 4. 单 Agent Loop vs Fleet Loop

- **单 Agent Loop**：一个 Agent 独立运行 5 阶段周期
- **Fleet Loop**：编排者 + 专业 Agent + 子 Agent 的分布式循环

### 5. 成本结构（被忽视的隐性障碍）

- 单 Agent Loop：5-20 万 token / 次
- Fleet Loop：50-200 万 token / 次
- 每日定时 Loop：每周数百万 token

低成本模型（DeepSeek / Kimi / MiniMax）+ 百万级上下文 让 Agent 循环在经济上变得可行。

### 6. 与 Harness 的层级关系

**Loop > Harness > Prompt**。Harness 管「这一次任务怎么跑」，Loop 管「这类任务怎么持续发生」。

### 7. 反馈闭环是 Loop 的灵魂

5 阶段循环：**发现 → 规划 → 执行 → 验证 → 迭代**。通过验证就交付，未通过就继续循环。**没有 verifier 的 loop = 失控的自动化**。

### 8. 在 Claude Code 的落地

- `/loop` 命令：定时循环 + Auto Mode
- `CLAUDE.md` 规则 + Hooks：状态记忆
- Review Mode：human-in-the-loop gate
- /goal：Fleet loop 早期形态

### 9. 在 OpenAI Codex 的落地

- `/goal` 命令：编排者 Agent 拆目标
- Sub-agent 分工：每个子 Agent 跑自己的 loop
- Loop Ledger 审计账本：trace/span 记录

### 10. 反对声音与边界

- **成本失控**：长时间 loop 累计 token 耗尽预算
- **反馈偏差累积**：verifier 缺陷导致错误被放大
- **过度工程化**：简单任务直接 prompt 更高效
- **生态未成熟**：loop 监控/调试/审计工具链仍不完善

---

## 相关 Entity（4 篇 + 1 篇主 entity）

### 主 Entity（必读）

- [[entities/loop-engineering-feedback-control-system|Loop Engineering: 把反馈循环放进工程现场 — **1781 字**，5 阶段循环 + 单 Agent vs Fleet + 现实案例（Claude Code/Codex/Hermes），由 3 篇 raw 蒸馏合并

### 平行 Entity

- [[entities/loop-engineering-addy-osmani-challengehub|Loop Engineering：不再写提示词，而是设计替你写提示词的循环 — **5864 字**，Addy Osmani 视角 + 6 大构件 + 5 阶段循环，含 Fleet Loop 拓扑 + Boris/Peter 公开访谈原始引用

### 上游 / 下游 Entity

- [[entities/harness-engineering|Harness Engineering — Loop 是 Harness 的「编排层」
- [[entities/claude-code-core-internals|Claude Code 核心机制 — `/loop` 命令的产品化实现
- [[entities/agent-harness-architecture|Agent Harness 架构 — loop 运行的容器
- [[concepts/agent-self-improvement-loops|Agent 自我改进循环]] — 抽象理论框架

---

## 原始资料（3 篇同主题 raw）

> 3 篇 raw 内容高度互补，主 entity 已综合 3 篇蒸馏合并

| 原始资料 | 篇幅 | 视角 | 状态 |
|----------|------|------|------|
| [[raw/articles/loop-engineering-infoq-boris-cherny-peter-steinberger|InfoQ 报道]] | 12955b | 褚杏娟 / InfoQ 媒体视角 | ✅ 已蒸馏 |
| [[raw/articles/loop-engineering-peter-steinberger-boris-cherny|Peter 本人论述]] | 12053b | Peter + Boris 原话 + 单 Agent vs Fleet | ✅ 已蒸馏 |
| [[raw/articles/loop-engineering-工程现场-ruofei|若飞工程现场]] | 13059b | 若飞 6 大组件 + 8 步框架（判断/提示词/控制层/闭环/验证/预算/状态/人在场/试点/收束） | ✅ 已蒸馏 |

---

## 与相邻概念的区分

| 对比项 | Loop Engineering | Harness Engineering | Prompt Engineering | Cron Job | AutoML |
|--------|------------------|--------------------|--------------------|----------|--------|
| 抽象层级 | 最高 | 高 | 中 | 低 | 低（领域特定）|
| 反馈 | 必须 closed-loop | 可选 | 无 | 无 | 模型性能 |
| 适合 | 重复 + 持续任务 | 单次任务 | 一次性对话 | 周期维护 | 调超参 |
| 核心组件 | loop 控制器 | tool + verifier | prompt | 调度 | hyperparam |
| 失败成本 | 高（持续运行）| 中 | 低 | 低 | 低 |

**关键区别**：Loop Engineering 把「agent 设计」从「单次 prompt 工程」升级为「持续运行的控制系统」——这是继 Harness 之后的下一层抽象。

---

## 学习路径（4 步）

### Step 1：理解基础

读 [[entities/loop-engineering-feedback-control-system|Loop Engineering: 把反馈循环放进工程现场 掌握 5 阶段循环 + closed-loop 概念。

### Step 2：看工程实现

读 [[entities/loop-engineering-addy-osmani-challengehub|Loop Engineering:不再写提示词,而是设计替你写提示词的循环——先写刹车再写循环（6 来源深度合并：Addy Osmani / Boris Cherny+Peter Steinberger / 教科书 / 若飞 工程现场 / TechFarrari 批判 / 若飞 实用指南） 了解 Addy Osmani 视角的 6 大构件 + Fleet Loop 拓扑。

### Step 3：理解生态

读 [[entities/harness-engineering|'Harness Engineering：AI 从 把 Loop 放回 Harness 框架，理解「Loop > Harness > Prompt」的层级。

### Step 4：实践

在自己的 1 个真实任务上跑 loop：
1. 找 1 个「重复 + 持续」的任务（CI 失败修复、依赖升级、监控响应）
2. 设计 loop 控制器（cron + worktree + state file + verifier + budget）
3. 跑 1 周，记录 token 消耗 / 成功率 / 回归率
4. 每月 review，根据数据调 loop 频率和 verifier 严格度

---

## 现状与时间线

- **2024 Q4-2025 Q1**：Claude Code 发布 `/loop` 命令（早期 loop 形态）
- **2025 Q3**：Claude Code Review Mode（human-in-the-loop gate）
- **2026 Q1**：Codex 发布 `/goal` 命令（Fleet loop 早期）
- **2026 Q2**：Boris Cherny + Peter Steinberger 公开力挺 Loop Engineering 范式
- **2026 Q2**：若飞《Loop Engineering 详解》系统化论述
- **预期 2026 下半年**：Loop Engineering 成为继 Prompt Engineering 之后的「必备技能」

---

## 相关 Skill / Script / Cron

- `wiki-loop-pilot`（若飞提出）— Loop 的元 skill，把 Loop 设计能力封装为可复用的 skill
- `loop-engineering-工程现场-ruofei` raw 详细描述了「6 大组件 + 8 步框架」的可操作清单

---

_本 MOC 由 4 entity + 3 raw 综合收口，最后更新 2026-06-15_

## 待关联概念

- [[concepts/activation-engineering|Activation Engineering]]
- [[concepts/harness-component-expiry-build-to-delete|Build to Delete 工程原则与开放问题]]
- [[concepts/routa-harness-visualization|Routa Harness 可视化：Vibe Coding 时代的工程可控性]]
