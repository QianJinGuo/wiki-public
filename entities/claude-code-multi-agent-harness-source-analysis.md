---
title: "Claude Code 多 Agent Harness 源码拆解：留纸条、抠上下文、抠缓存、捆手脚"
created: 2026-06-23
updated: 2026-08-01
type: entity
tags: [claude-code, multi-agent, harness, source-code, prompt-caching, coordinator, context-isolation, agent-communication, engineering]
sources: [raw/articles/claude-code-multi-agent-harness-source-analysis]
confidence: 0.8
provenance_state: extracted
---

# Claude Code 多 Agent Harness 源码拆解

## 核心结论

多 Agent 协作不是 AI 协作出来的，是 harness 用土得掉渣的工程手段在模型外面硬搭出来的脚手架。决定多 agent 系统好不好用的，从来不是里面的 agent 有多聪明，而是外面这层脚手架搭得有多结实。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]

Claude Code 源码泄露（50 万行 TypeScript）揭示了四个底层机制，每一个都与浪漫想象相反。^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]


## 四个源码级机制

### 1. 通信 = 互相留小纸条（pendingMessages）

不是实时对话，是**异步信箱**： ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]

- `pendingMessages`（待处理消息）：每个子 AI 配一个信箱
- 主 AI 调 `SendMessage` 塞纸条就走，不等回复（避免主 AI 阻塞 5+ 分钟）
- 子 AI **不会主动查信箱**——harness 在两轮接缝处替它取，塞进下一轮输入
- 子 AI 自始至终被动：不知有信箱，不知谁在喂它

**反向汇报**更绝：子 AI 把完工报告拼成 XML（`<状态>完成</状态>`），伪装成"用户消息"塞进主 AI 对话，源码叫 `task notification`。主 AI 看到的跟"用户突然发来一句话"无区别。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]

### 2. 隔离 = 一项一项手工抠（createSubagentContext）

不是"全新大脑从零开始"，也不是"全盘复制"——是**逐项决策**： ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]

| 给不给 | 风险 |
|--------|------|
| 全给 | 子 AI 读文件到 200 行 → 主 AI 书签被篡改 → 记忆串味 |
| 全不给 | 用户按停止 → 子 AI 收不到信号 → 失联继续跑 |

`createSubagentContext` 函数逐项决定每个状态字段的传递策略：哪些只读复制、哪些隔离、哪些广播。^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]


### 3. 省钱 = 抠到一个标点都不差（Fork Subagent + Prompt Caching）

Prompt Caching 折扣条件：**字节级完全相同**（byte-identical）。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]

- 错一个字符 → 从该字符往后缓存全部作废 → 按原价重算
- 真实翻车：某团队系统提示词含 `今天是 {当前日期}` → 一天缓存命中率 0%
- Claude Code 的解法：**Fork Subagent**——刻意让分身的 system prompt 与主 AI 一字节不差，吃满缓存折扣（一折）

### 4. 并行 = 捆住主 agent 的手脚（Coordinator 模式）

`CLAUDE_CODE_COORDINATOR_MODE=1` 开启后： ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]

- 主 AI 被系统提示词焊死成"包工头"（coordinator），禁止自己下场搬砖
- 核心指令：**"Parallelism is your superpower"** — 能同时上的活绝不排队
- 红线：**包工头必须自己读懂结果、写施工图纸**，不许当传话筒
- 传话筒没有存在意义——工人直接跟客户对话就行。包工头的价值在于"汇总+出图纸"环节真正动脑子

## 设计模式提炼

| 模式 | 源码实现 | 工程本质 |
|------|----------|----------|
| 异步信箱 | `pendingMessages` + `SendMessage` | 解耦发送与接收，避免主 AI 阻塞 |
| 消息伪装 | `task notification` XML | 复用现有消息处理通道，零新增协议 |
| 精细隔离 | `createSubagentContext` | 逐字段决策，避免记忆串味+信号丢失 |
| 缓存对齐 | `Fork Subagent` | byte-identical system prompt → 一折计费 |
| 手脚绑定 | Coordinator 模式 | 禁止 coordinator 自己干活，强制并行派单 |

## 与 Harness Engineering 理论的关系

这篇文章是 Harness Engineering 理论的**源码级实证**： ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]

- [[concepts/harness-engineering-framework|Harness Engineering 框架]] 定义了"模型外面的脚手架"——本文展示了这层脚手架在工业级系统中长什么样
- [[entities/harness-engineering-10-step-practical-guide-2026|Harness 实践指南 10步]] 的 Step 3（上下文管理）和 Step 10（并行多 agent）在本文有源码级对照
- [[entities/claude-code-dynamic-workflows-multi-agent-orchestration|Claude Code Dynamic Workflows]] 侧重编排模式和实战场景，本文侧重底层通信/隔离/缓存/并行机制——**互补不重复**

## 关键洞察

> "四样里，没有任何一样是 AI 在协作。全是有人在 AI 外面，用留纸条、复印、没收权限、伪装身份这些土到掉渣的老办法，一锤一锤搭出来的脚手架。" ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]

这是 Harness Engineering 的核心命题：**决定 AI 系统行不行的，不是里面那个模型，是外面这层 harness**。^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]


## 相关实体
- [[concepts/harness-engineering-framework]]
- [[entities/harness-engineering-10-step-practical-guide-2026]]
- [[entities/claude-code-dynamic-workflows-multi-agent-orchestration]]
- [[entities/long-running-agent-ralph-loop-harness-takeover]]
- [[entities/gufabiancheng-spec-for-complex-tasks-cc-codex]]
- [[entities/production-harness-12-components-framework-comparison]]

→ [[raw/articles/claude-code-multi-agent-harness-source-analysis|原文存档]] ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]
