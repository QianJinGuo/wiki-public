---
title: "Agent 作为软件架构的新一层：Codex Harness 平台化 + DSH Subagent（Agent-as-Execution-Unit）"
author: 若飞
source: 架构师 JiaGouX (2026-08-21)
score: v=8, c=6, v×c=48
type: entity
created: 2026-08-21
updated: 2026-10-01
tags: [agent-runtime, harness, codex, app-server, dsh, deepseek-harness, subagent, agent-as-runtime, orchestration, claude-code, agent-execution-unit, agent-composition, control-split]
sources:
  - raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21
  - raw/articles/deepseek-harness-v012-runtime-chain-cordis-graph-goal-ruofei-2026
confidence: high
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agent 作为软件架构的新一层：Codex Harness 平台化 + DSH Subagent

## 一句话总结

若飞源码级拆解 DeepSeek Harness（DSH）0.1.0-rc.8 的 subagent 子系统与 OpenAI Codex 的平台化开放，提出**「Agent 正在成为软件架构的新一层」**：完整 Agent（带着自己的 Loop、状态和权限体系）开始被当作另一个系统的可调用执行单元，组合粒度从「模型调用 → 工具调用」抬升到「**Agent 调用**」，并推演出 Agent 编排需拆分的四种控制权与 Harness 作为执行基础设施层的判断。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md]

## 为什么是独立维度

既有相关实体覆盖不同层次：`claude-fable-5-agent-runtime-contract-ruofei-2026` 讲 Runtime 契约协议层（任务协议/能力路由/状态/治理）；`agent-runtime-7-responsibilities-secondcurve-2026` 讲 Runtime 七大职责；`codex-goal-agent-runtime` 讲 Codex goal 状态机。而本文的核心维度是**「Agent 作为可被另一系统调用的执行单元」**——具体到 Codex app-server 的控制接口/事件转换层职责、DSH 把 Codex/CC 接成 `subagent_codex`/`subagent_claude_code` 具名工具的实现，以及「模型调用→工具调用→Agent 调用」的组合粒度跃迁——这一「Agent 进入软件架构新一层 + 编排控制权拆分」视角全库零覆盖。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md]

## 核心机制

### Codex Harness 平台化（app-server）
OpenAI 把 Codex 背后的 Harness 放到平台位置，开放 CLI/SDK/app-server/Skills/Plugins。**app-server 是控制接口和事件转换层**，不是再造 Agent Loop——它把外部请求转换成 Thread/Turn 操作，把核心事件整理成客户端可消费的通知。源码链路：`外部宿主 → app-server → thread/start → turn/start → run_turn → 模型采样与工具执行`。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md]

### DSH 把完整 Agent 接成子代理
DSH rc.8 里 Codex 和 Claude Code 是 Profile Bundle，安装并打开工具后，父 Agent 看到 `subagent_codex`/`subagent_claude_code` 具名工具。调用分两段：父 Agent 决定委派/选工具/写任务/前后台；DSH 运行时负责注册适配器、校验、启动进程、管理 Job、取消、收结果。provider 预绑定 `providerName` 和固定 `toolName`，模型只见稳定工具名。Codex 走 app-server（临时 thread/turn），Claude Code 走官方 SDK（`persistSession: false`）。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md]

### 组合粒度跃迁
回顾 MCP → Agent 技术栈 → Context/Loop/Harness/Environment 的演进，DSH 这次接进来的是带自己 Loop/状态/权限体系的完整 Agent——组合粒度从「模型调用 → 工具调用」抬升到「模型调用 → 工具调用 → **Agent 调用**」。工具协议解决「Agent 怎样使用能力」，Runtime 层则问「一个完整 Agent 怎样被另一个系统调用」。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md]

### Agent 编排的四种控制权拆分
| 参与者 | 在调用链里做什么 |
|--------|----------------|
| 父 Agent | 判断是否委派、选择子代理、组织任务、决定是否等待 |
| DSH 运行时 | 暴露工具、启动适配器、管理进程/Job/取消/结果收集 |
| Codex/Claude Code | 在各自 Harness 内维护上下文、运行 Loop、调工具、执行原生沙箱和权限策略 |
| 业务系统与人 | 提供权威事实、决定业务动作能否发生、用真实结果验收 |

三件事不能混：选中子代理 ≠ 拿到全部权限；子代理说「完成」≠ 业务验收通过；取消 Job ≠ 副作用已回滚。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md]

### Harness 作为执行基础设施层
模型提供推理能力，Harness 管状态/Loop/工具/沙箱/审批/事件，宿主接进产品和业务流程。这一层不取代业务后端——Harness 的状态不是订单/运单/发布记录，审批不继承公司业务授权。ARC-AGI-3 例子（保留状态+上下文压缩，GPT-5.6 Sol 13.3%→38.3%、Token 降 1/6）只支撑克制判断：模型没换，执行状态怎么保留结果差很多。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md]

## 当前局限（克制边界）
OpenAI 文档仍把 app-server 命令和 WebSocket 传输标在实验性边界内（不支持生产工作负载）；DSH rc.8 两个外部 provider 还是一次性委派，无共享长期记忆/持续协作/统一验收。代码开放、协议跑通、生产可托付是三件事。

## 深度分析

### 「Agent 调用」不是更粗的工具，而是另一种调用对象

MCP 解决「Agent 怎样使用能力」，subagent 机制把问题换成「一个完整 Agent 怎样被另一个系统调用」。这不只是组合粒度抬升一层，而是调用对象性质变了：工具调用是类型明确的请求/响应，Agent 调用传入的是自然语言任务文本，子代理在自己 Harness 里跑独立 Loop、维护上下文、执行原生沙箱与审批策略，返回的「已完成」只是自述而非可入账的业务事实——像服务调用但不能照抄微服务，任务合同、超时、隔离、可观测、权限联动、副作用处理都还没有成熟标准。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:57-67]

### 三段调用链暴露「控制权拆分」是编排的真正设计单位

把 `subagent_codex` 一次调用拆开看：父 Agent 只做委派决策与任务组织，DSH 运行时只做进程/Job/取消/结果收集，Codex 在自己 Harness 内完成采样、工具执行与沙箱审批，业务系统与人掌握权威事实和验收——四种责任各有归属，这解释了为什么「主 Agent」更像任务里的临时角色而非固定产品身份。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:69-80] 若飞的三组「不能混」实质是把传统软件工程的授权、验收、事务三大边界平移到 Agent 编排层，Relay 物流示例（重订运单前把动作摆到人面前审批、执行完刷新权威记录）就是落地形态。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:80-82]

### rc.8 的「一次性委派」揭示了当前组合能力的真实水位

Codex 适配器每次新开进程/临时 thread/turn，Claude Code 设 `persistSession: false`，子代理只拿到独立任务文本 + 父会话工作目录，不继承父对话、角色、工具筛选与深度策略，中间细节也不复制回父会话，后台 Job 可查可取消但不能凭 Job ID 续会话。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:47-51] 换句话说，「Agent 调用」在协议上已经成立，在协作形态上仍是单发 RPC 而非常驻团队——没有共享长期记忆、持续协作、统一超时验收和副作用回滚。对「Agent 进入架构新一层」的判断，这是一条重要的降火线：代码开放、协议跑通、生产可托付是三件事。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:90]

### Harness 基础设施论的克制之处：它明确不取代业务后端

ARC-AGI-3 实验数字（保留推理状态 + 上下文压缩后 GPT-5.6 Sol 13.3%→38.3%、输出 Token 降到 1/6）很容易被读成「Harness 比模型重要」，但若飞只用它支撑一个克制判断：模型没换，执行状态怎么保留、上下文怎么整理，结果差很多。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:84-88] 更关键的划界是：Harness 的状态不是订单/运单/发布记录，审批不继承公司业务授权——它负责让 Agent 持续可控地干活，业务系统负责事实、授权与验收。这与 [[entities/agent-runtime-7-responsibilities-secondcurve-2026|Agent Runtime 七大职责]] 的分层思路一致：Runtime/Harness 层与业务后端是互补而非吞噬关系。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:88]

## 实践启示

### 评估 subagent 方案时先问「状态去哪了」

选型或自建 Agent 编排时，不要停留在「能不能调通」，先逐项检查：子代理是否继承父上下文？会话是否可持久化（对照 rc.8 的临时 thread/turn 与 `persistSession: false`）？中间产物能否回传观测？后台任务能否凭 Job ID 续会话？ ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:47-51] rc.8 的现状说明这些都是空白——记忆与状态策略必须在编排层自行设计，而不是指望 provider 提供。

### 把三组「不能混」做成工程闸门

选中子代理 ≠ 拿到全部权限、子代理说完成 ≠ 业务验收通过、取消 Job ≠ 副作用已回滚——这三条可以直接翻译成系统约束：子代理工具白名单独立于父 Agent 授权而非全量继承；「完成」信号后必须走独立验收步骤才允许进入业务流程；取消/超时路径必须显式处理副作用清理而非假定进程回收等于状态回滚。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:80] Relay 示例给出了模板：真要执行敏感动作时把具体动作摆到人面前审批，执行完刷新权威记录。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:82]

### 对外部 Agent 依赖做版本锁定，别信系统 PATH

DSH 的两个适配器都做了防呆：Codex 固定 `@openai/codex@0.147.0`，Claude Code 由固定版本 SDK 选随包 CLI，都不找系统 PATH。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:45] 把另一个 Agent 当执行单元接入自己系统时应复制这一做法——外部运行时是黑盒升级源，隐式依赖系统命令会让行为随环境漂移且无法复现。

### 并发委派前先隔离工作目录

rc.8 沿用父会话工作目录给子代理，并发改同一批文件会冲突，且无统一回滚。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:51] 多子代理并行写同一仓库的场景应在委派层自行隔离（独立 worktree/沙箱目录/文件范围划分），并把「副作用可回滚」当作委派决策的前置条件。

### 区分三个成熟度：代码开放、协议跑通、生产可托付

app-server 的 thread/turn 命令与 WebSocket 传输仍被 OpenAI 标在实验性边界内（不支持生产工作负载），跨主机 WebSocket 还要单独处理认证与 TLS。 ^[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21.md:90] 引入这一代 Agent 基础设施时，用这三个台阶评估各组件的真实水位——这一分寸感与 [[entities/harness-engineering-paradigm-comprehensive-2026|Harness 工程范式]] 一致。

## 相关概念
- [[entities/claude-fable-5-agent-runtime-contract-ruofei-2026|Fable 5 信号：Agent 拼 Runtime（若飞 Runtime Contract）]]
- [[entities/agent-runtime-7-responsibilities-secondcurve-2026|Agent Runtime 七大职责]]
- [[entities/codex-goal-agent-runtime|Codex Goal Agent Runtime]]
- [[entities/harness-engineering-paradigm-comprehensive-2026|Harness 工程范式]]
- [[entities/agentic-environment-engineering-jiagoux-2026-06-27|Agentic Environment Engineering]]
- [[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21|原文存档]]

→ [[raw/articles/codex-harness-platform-dsh-subagent-agent-as-execution-unit-ruofei-2026-08-21|原文存档]]
