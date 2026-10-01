---
title: "Claude Code 多 Agent Harness 源码拆解：留纸条、抠上下文、抠缓存、捆手脚"
created: 2026-06-23
updated: 2026-10-02
type: entity
tags: [claude-code, multi-agent, harness, source-code, prompt-caching, coordinator, context-isolation, agent-communication, engineering]
sources: [raw/articles/claude-code-multi-agent-harness-source-analysis]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
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


## 深度分析

### pendingMessages：异步信箱把"通信"降级成"留纸条"

源码里没有 agent 之间的实时对话。每个子 AI 配一个信箱（`pendingMessages`），主 AI 调 `SendMessage` 往信箱末尾塞一张纸条就扭头走人，根本不等它看——因为子 AI 接的活可能要跑 5 分钟，主 AI 若交出去就干等，整个会话等于卡死。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:23]

更深一层看：**真正去信箱取纸条的根本不是子 AI，是它外面那套循环机制**。子 AI 不主动查信箱，只是埋头一轮轮干活，harness 在每一轮结束、进入下一轮的接缝处替它瞄一眼，有新纸条就塞进下一轮输入。子 AI 自始至终被动——它既不知道有信箱，也不知道是谁在喂它。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:25] 这意味着"收消息"这个最基础的协作动作，也不在 agent 的能力范围内，而在 harness 的接缝逻辑里。

反向汇报同样没有"完工信号"这种正式协议：子 AI 把完工报告拼成一段 XML，伪装成一条"用户发来的消息"塞进主 AI 的对话（源码叫 task notification）。对主 AI 来说，这玩意儿跟用户突然说一句话没有任何区别。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:31]

### task notification XML：用伪装换协议成本

task notification 的本质是一种**消息伪装**：不定义新协议、不新增消息类型，直接把结构化完工报告（`<状态>完成</状态>` 这类带标签文本）包装成主 AI 已有的输入通道——用户消息。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:33] 工程上的取舍很清楚：主 AI 的理解能力本来就在，与其为 agent 间通信发明一套它需要额外学习的协议，不如复用它每轮都在处理的"用户输入"格式，零新增解析逻辑。这是典型的 harness 思维——能借模型已有能力解决的，就不在脚手架里加机制。

### createSubagentContext：逐项隔离，因为两个极端都是坑

"给子 AI 一个独立工作空间"听起来是发个全新大脑从零开始。源码显示这恰恰是最磨人的细活：主 AI 身上挂着一堆随身记录（文件读到第几行、屏幕显示、停止键状态、后台任务），派子 AI 时这些给不给？ ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:43]

两个想当然的答案都是坑：

- **全给**：主 AI 读文件到 100 行，子 AI 接着读到 200 行，主 AI 的书签被划走——等它自己再读这文件，以为读过了直接跳过。子 AI 一个动作搅乱了主 AI 的记忆。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:47]
- **全不给**：用户按停止键想中止任务，信号广播出去，子 AI 因为跟主 AI 啥都不共享根本收不到，自顾自接着跑，彻底失联。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:49]

`createSubagentContext` 函数的解法是近乎偏执的细心：不一刀切，而是对这一大堆记录**一项一项单独决定**怎么处理。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:51] 这正是上下文工程里"共享 vs 隔离"没有默认答案的实证——每个状态字段的传递策略都必须显式决策，漏一项就出事故。

### Fork Subagent 与 Coordinator 模式：抠缓存的对齐、捆手脚的并行

**Fork Subagent**（分叉子 AI）解决了多 agent 的隐性成本问题：每派一个有专属 system prompt 的子 AI，背后的大模型就得把上万 token 的系统提示词从头重算重收。Prompt Caching 折扣的条件极其苛刻——必须字节级完全相同，错一个字符，从那个字符往后缓存全部作废。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:62] 有真实翻车案例：某团队在系统提示词开头写了"今天是 {当前日期}"，一个每天会变的动态字段，废掉一整天全部缓存折扣。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:64] Fork Subagent 刻意让分身的系统提示词与主 AI 一个字节都不差，就是为了吃满这个折扣。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:66]

**Coordinator 模式**（`CLAUDE_CODE_COORDINATOR_MODE=1` 手动开启）则是任务大到十个 AI 同时上时的答案。反直觉的是：让并行高效的关键不是让 agent 更会协调，而是用最笨的办法把主 AI 的手脚捆死——系统提示词把它焊成"包工头"，只许指挥 worker 调研、施工、验收，绝不许自己下场抢活。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:74] 点睛指令是 "Parallelism is your superpower"：工人各干各的不互相等，能同时上的活绝不排队。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:79]

但整套设计有条最容易被做砸的红线：**包工头必须自己"看懂"，不能只当传话筒**。源码反复叮嘱，工人交回调研结果后，包工头必须自己读懂、嚼碎、写成明确的施工图纸再派下去，不许甩一句"就照你查到的去改"。如果只是原样转话，工人直接跟客户对话就行了，包工头毫无存在必要——它的价值恰恰在于"汇总+出图纸"环节真正动脑子。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:83]

## 实践启示

1. **子 agent 通信默认异步，别设计实时对话。** 用信箱模式（追加消息 + 轮次间隙投递）代替"派活后干等"，主 agent 在子任务执行期间保持可响应。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:21]
2. **子 agent 的完工回报可以伪装成用户消息。** 不必发明新协议——把结构化报告包进主 agent 已有的输入通道，零新增解析成本，主 agent 用理解用户的方式理解回报。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:31]
3. **上下文继承逐字段决策，拒绝全给/全不给两个极端。** 写一个类似 `createSubagentContext` 的显式清单：哪些只读复制、哪些隔离、哪些广播，并对"书签被篡改"和"停止信号收不到"这两类事故分别设防。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:51]
4. **动态内容（日期、时间、随机值）严禁放进 system prompt 开头。** 一个变化字段就能让后续全部 token 缓存作废；需要多 agent 共享缓存折扣时，让子 agent 的系统提示词与主 agent 保持字节级一致。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:64]
5. **规模化并行时捆住协调者的手脚。** 在协调者的系统提示词里明令禁止它自己下场干活，并写入"能并行绝不排队"的强指令——并行度来自纪律，不来自智能。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:77]
6. **协调者必须消化结果再派活，不做传话筒。** 工人回报后，协调者要自己读懂并产出明确的施工图纸；只转发不加工的协调层可以直接删掉，让工人直连需求方。 ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md:83]

## 相关实体
- [[concepts/harness-engineering-framework]]
- [[entities/harness-engineering-10-step-practical-guide-2026]]
- [[entities/claude-code-dynamic-workflows-multi-agent-orchestration]]
- [[entities/long-running-agent-ralph-loop-harness-takeover]]
- [[entities/gufabiancheng-spec-for-complex-tasks-cc-codex]]
- [[entities/production-harness-12-components-framework-comparison]]

→ [[raw/articles/claude-code-multi-agent-harness-source-analysis|原文存档]] ^[raw/articles/claude-code-multi-agent-harness-source-analysis.md]
