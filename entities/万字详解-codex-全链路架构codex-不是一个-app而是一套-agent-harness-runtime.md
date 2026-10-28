---
title: "万字详解 codex 全链路架构：Codex 不是一个 App，而是一套 Agent Harness Runtime"
created: "2026-07-01"
updated: 2026-10-27
type: entity
tags: [codex, agent-harness, runtime, app-server, sandbox, openai, architecture, multi-agent]
provenance_state: inferred
rating: v9c9
sources:
  - raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime
  - raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26
  - raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 万字详解 codex 全链路架构：Codex 不是一个 App，而是一套 Agent Harness Runtime

OpenAI 的 Codex 已从单一 CLI 工具演进为一套围绕 agent runtime 展开的多端系统——涵盖 Web/Cloud、桌面 App、CLI、IDE 扩展、Exec、SDK、MCP Server 及 GitHub Action 等八个端，全部由同一套 Codex harness 驱动。其核心架构分为四层：客户端表面、协议与集成层、本地 runtime、OpenAI 后端与云端环境。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]

## 核心要点

- **多端统一 harness**：Codex 所有端由同一个 Codex harness 驱动，使用同一套 thread/turn/item 抽象和同一个 app-server 协议面。
- **App Server 是关键底座**：它不是一个传统业务后端，而是一个本地 agent runtime 的控制接口，通过 JSON-RPC event stream 将客户端请求翻译成 Codex Core 内部操作。
- **三层会话模型**：Thread（可持久化的会话）、Turn（一次用户输入触发的完整任务）、Item（最小事件单元：agent message、plan delta、command output、file patch 等）。
- **三档沙箱体系**：read-only（只读探索）、workspace-write（工作区可写、外部资源受限）、danger-full-access（不隔离），支持不同任务分配不同权限边界。
- **多端集成策略**：MVP 用 `codex exec`，富交互场景用 App Server，单纯做工具用 MCP Server。

## Codex 产品形态全景

Codex 的产品形态可拆解为八个端，各自针对不同使用场景：^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]

| 端 | 主要用途 | 定位 |
|------|---------|------|
| Codex Web / Cloud | 云端并行任务、GitHub PR、后台 code review | OpenAI 托管的 agent runner |
| Codex App | 桌面 command center，多线程、多 worktree、多项目 | 本地 agent 工作台 |
| Codex CLI / TUI | 终端交互、本地代码修改、命令执行 | 面向工程师的原生入口 |
| Codex IDE Extension | VS Code、Cursor、Windsurf 等 IDE 内协作 | 编辑器里的 agent 面板 |
| Codex Exec | CI、脚本化、一次性任务 | 自动化命令入口 |
| Codex SDK | 程序化控制 Codex | 给业务系统接入的开发包 |
| Codex MCP Server | 把 Codex 暴露给其他 agent harness | 可调用的 coding tool |
| GitHub Action | 在 CI 里跑 Codex | 把 Codex 接进流水线 |

## App Server：关键底座

App Server 是 Codex 架构中最关键但也最容易误解的组件。它不是传统 HTTP API 后端——客户端通过 JSON-RPC 发起 thread、turn、approval、工具调用，App Server 再将这些请求翻译成 Codex Core 能理解的内部操作。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]

OpenAI 的工程文章将 App Server 进程拆成四块：^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]

1. **stdio reader**：读取客户端输入
2. **Codex message processor**：处理协议消息
3. **thread manager**：管理会话生命周期
4. **core threads**：执行实际 agent 任务

客户端不直接操作 Rust Core——客户端只说"start thread""resume""approve""interrupt"，App Server 负责翻译。监听方式支持 `stdio://`、`unix://`、`ws://IP:PORT` 和 `off`，说明 App Server 正在变成 Codex 多端统一协议面。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


## 三层会话模型

Codex 把会话拆成三个 primitive，以支持富客户端体验：^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]

### Thread（线程）
可持久化的 Codex 会话，支持 resume、fork、archive，客户端断线后可重新接上。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


### Turn（轮次）
一次用户输入触发的完整任务，内部包含推理、计划、搜索、读文件、跑命令、改代码、申请用户确认等多步操作。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


### Item（事件项）
最小事件单元——可以是 agent message、plan delta、command execution output、file patch、MCP tool call、approval request 等。客户端通过 event stream 持续接收 items，实现实时渲染进度、diff、执行日志。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


这一抽象确保富客户端不只拿到"最终答案"，而是能够感知每一步的执行状态。Codex Web 的 worker 在容器里启动 App Server binary，浏览器通过 Codex backend 的 HTTP/SSE 看事件——即使网页关掉，任务还能继续跑。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


## 沙箱与权限体系

Codex 定义了三档沙箱模式，对应不同使用场景：^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]

| 沙箱模式 | 含义 | 适用场景 |
|---------|------|---------|
| `read-only` | 只读探索，不改文件 | review、理解代码、风险分析 |
| `workspace-write` | 工作区可写，外部资源受限 | 日常开发、测试修复 |
| `danger-full-access` | 不隔离 | 外部已有容器保护的 CI |

这一体系的核心设计理念是给不同任务分配不同边界——Codex 的使用姿势不是"让 AI 随便跑"，而是在可控制的范围内赋予适当的权限。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


## 多端集成策略

Codex 的集成能力呈现明显的用途-成本权衡：^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]

- **`codex exec --json`**：适合 CI、脚本、一次性自动化。代价低，会话控制能力有限，但能跑通、能回滚、能进 CI。
- **`codex app-server`**：适合富客户端、IDE、桌面 App、远程控制台。功能完整但需要处理 JSON-RPC 绑定。
- **`codex mcp-server`**：适合已有 MCP workflow 只想把 Codex 当工具的场景。会损失一些独特的 thread/diff/approval 语义。
- **TypeScript SDK**：适合 Node 服务里程序化控制 Codex，底层仍是 spawn CLI + JSONL。

推荐模式：MVP 先用 exec，确认业务价值后再迁移到 app-server 实现富交互。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


## 深度分析

### 1. "端是入口，Harness 才是身体"——多端统一架构的战略意义

Codex 最值得分析的并非其前端功能，而是其"多端共享同一 harness"的架构决策。这一决策意味着：所有端的改进都沉淀在底层 harness 层，任一端的创新自动惠及其他端。对比竞品（如 Claude Code 以 CLI 为主、Cursor 以 IDE 为主），Codex 的"多端统一"架构使其天然更适合嵌入企业工作流的不同环节——工程师在终端用 CLI，管理者在 Web 看任务状态，CI 流程通过 exec 调用，业务系统通过 SDK 集成。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]

### 2. App Server 协议面的设计哲学：从"聊天 API"到"任务运行时"

Chat API 的范式是"请求→响应"，而 App Server 的范式是"订阅运行现场"。这一转变反映了 AI 代理从"对话系统"到"工程系统"的定位变化。App Server 协议面当前包含的能力（thread 生命周期、turn 控制、item 流、命令执行、文件操作、审批流、MCP 管理、远程控制）已经远超传统聊天 API 的范围，更接近一个操作系统的进程管理 API。这种"AI 代理即服务"的模式可能是企业级 AI 集成的未来形态。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


### 3. 沙箱设计："权限分层"而非"权限开关"

Codex 的三档沙箱体现了精细化的权限设计哲学——不是简单的"有/无"权限，而是根据任务类型（只读分析、日常开发、完全控制）提供不同的边界。这种"权限分层"思路比二元模型更适合企业场景：代码审计走 read-only，开发任务走 workspace-write，CI 环境走 full-access。每一档都有清晰的使用范围和约束条件。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


### 4. "Codex 不是写代码的工具，而是代码 agent 的 runtime"的深远影响

文章的核心判断——"Codex 真正要做的不是某一个端，而是一套围绕 agent runtime 展开的多端系统"——揭示了 OpenAI 在 coding agent 领域的战略定位。如果这一判断成立，那么 Codex 的长期竞品不是 Copilot 或 Claude Code，而是操作系统级别的 agent runtime 平台。就像 Windows/Mac/Linux 提供了应用运行的基础设施，Codex 旨在成为 AI agent 运行的基础设施——管理进程、文件、权限、会话、状态和通信。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


### 5. 状态管理是企业集成的关键挑战

文章特别指出 Codex 不是无状态工具——`CODEX_HOME` 中包含 config、auth、sessions、history、logs、sqlite 状态。这对企业集成非常关键：不能让所有业务 agent 共享同一个全局 `[本地运行时路径已隐藏]`，否则登录态、配置、历史、插件全混在一起，排查会很困难。建议做法是每个业务 runtime 给独立 `CODEX_HOME`，每个项目有自己的 AGENTS.md，每类任务有自己的 sandbox 和 approval policy。^[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime.md]


## 实践启示

1. **评估 AI 编码工具时关注"底层运行时"而非"表层功能"**：工具提供的 CLI/IDE/Web 界面只是入口，真正决定长期价值的是一套统一的底层运行时（harness）——它决定了工具的可嵌入性、可审计性和可扩展性。

2. **沙箱与权限体系决定 AI 代理能否进入生产环境**：一个没有完善沙箱设计的 AI Coding 工具无法通过企业安全审查。在选型时，read-only 分析和写权限分离是最低要求。

3. **"先 exec 跑 MVP，再 app-server 做深度集成"是务实路径**：不需要一开始就搭建完整的 App Server 集成。用 `codex exec --json` 快速验证业务价值，确认后再投入资源做深度的 app-server 集成。

4. **企业部署需注意状态隔离**：如果计划在多个业务场景中使用 Codex，务必为每个场景配置独立的 `CODEX_HOME` 和工作目录，避免状态污染。

5. **MCP 集成有功能取舍**：如果通过 MCP 集成 Codex，需意识到会在 thread/diff/approval 等独特语义上有所损失。对于需要完整 agent 体验的场景，app-server 集成更合适。

> 参考资料：OpenAI Codex App Server 工程文章、Codex CLI 文档、Codex SDK 文档、Codex 官网

→ [[raw/articles/万字详解-codex-全链路架构codex-不是一个-app而是一套-agent-harness-runtime|原文存档]]

## 增补（2026-10-26 产品史复盘）：人回来的时机

前文拆的是"能力层"——App Server、Thread/Turn/Item、沙箱与多端。2026-10-26 若飞编译的 Every《How Codex Happened》复盘补上了另一层：**当 Agent 从 30 秒一次反馈变成能独立跑几小时，人该在什么时候回来。** 这一层不是协议能力，而是交互政策，既有实体此前未覆盖。^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]

**反馈单位从动作换成事件。** 2024 年末推理模型刚接入终端时，产品标准是"用户每隔 30 秒、最多 1 分钟就要知道模型做到哪了"——当时的模型走不了太远，读几个文件、跑一条命令、遇到一个报错都可能把任务带偏，人频繁查看是在替系统保存方向。模型能跨文件修改、自己跑测试之后，把每条命令都推给人反而打断任务；可几小时后只回来一句"完成了"，人也不敢直接合并。于是长任务没有取消反馈，只是把反馈的单位从**动作**（读了哪个文件、执行了什么命令）换成**事件**：为什么停下、缺什么信息、哪个动作越过权限、方向是否需要调整、最后有什么证据证明结果可用。^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]

**四次该回来，一次必须回来。** 短任务按动作反馈；长任务只在**信息不足、权限越界、方向变化、执行异常**时请人介入，最后在**验收**时回来。^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]

**第一版 Cloud 的失败是接续失败，不是配置麻烦。** 2025 年春约 7 周做出第一版 Cloud（远端机器读代码、改文件、跑测试，电脑可合上）。内部跑得不错，外部却卡住：开发者要先把笔记本上的软件、权限、设置、凭证与依赖搬进云端，这个准备过程可能耗掉几小时；任务启动后若 Agent 理解错了，**用户不能在原任务里补一句要求——追加提示会拉起另一台云端机器**。"代码过去了，开发现场没有完整过去"：异步执行要成立，人离开前留下的目标、Agent 已做过的尝试、人回来后的补充指令，必须落在同一个任务状态里；少了任何一段，"放手"只是把中途纠偏换成事后返工。^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]

**审批请求本身要可判断。** 以"升级数据库驱动并修复受影响测试"为例：Agent 改完驱动版本、锁文件与连接池兼容代码，单元测试通过后准备跑 `./scripts/test-migrations.sh`，脚本需要读沙箱外的 `/srv/shared/test-db.env`，任务停在权限边界。此时只弹一句"需要审批"，开发者几乎没法判断；合格的审批请求至少要说清：准备执行什么、将读取哪份配置、哪些文件已被修改、当前还没有执行迁移也未触碰共享库、**拒绝后会保留哪些改动**。开发者据此决定"不要碰共享环境，改用仓库里的 `docker-compose.test.yml` 起本地 PostgreSQL"，Agent 保留已完成修改、只调整测试夹具并从迁移测试继续。整个过程人只被叫回两次：一次定权限与测试路径，一次验收结果——**少看了很多过程，不等于少了控制，控制被压缩到真正影响结果的两个时刻**。^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]

**Runtime 要记住的不只是聊天记录。** Codex Cloud 受挫后 Tibo 接手，把重心转回本地 CLI，并推动用 Rust 重写底层 Agent，把任务循环、工具调用、上下文与运行状态从界面里拆出来——这个选择后来同时支撑 CLI、IDE、桌面与手机：入口不同，下面接的是同一条任务链。App Server 接口已能看出这条链：Thread 保存任务及历史，`turn/steer` 把补充要求送进正在运行的 Turn，`turn/interrupt` 中断当前执行，`thread/status/changed` 通知外层产品状态变化，`thread/resume` 在人回来后恢复原 Thread。对应到本例，系统至少要保住六样东西：原目标、已完成（驱动版本/锁文件/兼容代码）、当前停因（申请读取哪份配置）、人的决定、恢复位置、验收要求。这些状态若只躺在某个界面的内存里，换到手机、重启客户端或中断一次，任务就要重新解释；**Runtime 把它们从界面里拿出来，人的补充才不是"再发一个 Prompt"，而是对当前执行的转向**。^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]

**控制面要先回答四个问题。** 任务走到哪里、为什么停下来、人要决定什么、决定后从哪里继续——这不是一块更复杂的任务看板，而是 Agent 控制面的定义。手机入口之所以有用，正因为状态已从界面拆出：OpenAI 的 Remote Connections 文档写明项目、文件、凭证、Shell、工具与执行环境仍在连接的电脑或远程主机上，手机只负责继续任务、回答问题、补充要求、处理审批、查看 Diff/测试结果/终端输出与截图。**手机不是新的开发现场，它只是离人最近的控制入口**；相应地，一条可处理的通知至少要带停因、已完成的动作、影响范围与可选路径，否则"任务需要关注"等于让人重新翻上下文。^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]

**Goal 保存目标，证据决定结束。** Codex 的 Goal 是 Thread 范围的持久目标：一次 Turn 结束、Thread 空闲、没有待处理工作与排队输入时，系统才判断是否继续；中断会暂停目标；**一次续跑没有调用工具，下一次自动续跑也会被抑制**，避免原地空转。但 Goal 不能替 Agent 宣布完成——预算用完只说明这次不能再跑，Turn 结束只说明一轮执行结束，模型说"已经修复"也只是一句话；最终仍要回到可检查的东西：驱动版本是否更新、Diff 有没有越界、迁移命令退出状态、单元/迁移测试是否通过、日志里有没有被跳过的失败。^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]

**三件事不能混。** 长任务里：**Agent 负责执行，Runtime 保存状态并接住人的介入，测试/日志/制品负责证明结果**。模型能力变强时扩张最快的是执行；人能不能放心离开，取决于后两块有没有跟上。这也解释了"30 秒"标准为什么没有过时——只是理由变了：早期回来是为了看动作防止走偏，现在回来是因为信息不够、权限越界、方向改变、执行异常，以及结果等待验收。**Agent 越能自己干活，人越不该被困在过程里，但每次被叫回来都要有一个具体、可判断的理由。**^[raw/articles/codex-team-retro-feedback-unit-handoff-chain-jiaogoux-2026-10-26.md]


## 增补（2026-10-27 源码级拆解）：三层循环、World State 与 Token Budget

千问AI平台基于 Codex CLI 开源源码的拆解，补齐了前两轮增补都未涉及的**实现层细节**与 8 月的记忆更新。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**模块化解耦：118 个 crate，且明令"不要往 core 里加代码"。** 整个 Harness 被拆成 118 个 Rust crate，核心的 `codex-core`（Session/主 Loop/工具执行/上下文管理与压缩）在 AGENTS.md 里写了一条硬规则：**"Resist adding code to codex-core"**——每加一个新概念先想它该不该放在别的 crate。收益是编译隔离（改一个小 crate 不必重编全项目）与高内聚低耦合；但作者也指出，Codex 只做到代码级拆 crate，**没有像 DeepSeek Harness 那样把组件彻底抽象成插件**。关键 crate：`codex-core`（运行时）、`codex-protocol`（Op/Event/Response Item 数据定义）、`codex-api`（Response API 的 HTTP 与 WebSocket）、`codex-app-server`（对外 JSON-RPC，TUI/Desktop 统一接它）、`codex-tui`、`codex-exec`（无头前端）、`codex-rollout`（会话持久层，即 Agent Trace，存文件或 SQLite）、`codex-arg0`。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**Op/Event 双向事件驱动。** 客户端→Agent：用户操作封成 `Op`（TurnInput / Interrupt / Compact / Shutdown 等），装进 Submission 信封传输；Agent→客户端：`Event`（TurnStarted / TurnComplete / AgentMessage / AgentReasoning / ExecCommandBegin / OutputDelta / TokenCount 等）。与 DeepSeek 的全插件广播不同，Codex 的事件驱动主要服务这两个方向的协议传递。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**三层循环：Task → Turn → Sampling。** 最外层 Task Loop 处理用户需求（含把用户消息置为 pending）；内层 Turn Loop（Run Turn）是一轮对话，**上下文压缩也算一个 Turn**；最内层 Sampling Loop 对应 DeepSeek 的 Step，但粒度更细——在 Codex 里每一个操作、每一个小步骤都算一次 Sampling。这解决了前文"Thread/Turn/Item"抽象在运行时的层次问题：Item 是事件面，Task/Turn/Sampling 才是执行面。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**工具并发但有序拼装。** 一次模型响应里的多个工具调用用 `FuturesOrdered` 并发执行、并发收集，但写回模型时**严格按调用顺序拼装**（调 A/B/C 就按 A/B/C 返回）。理由是上下文确定性：顺序错乱会让模型混淆。表象是串行、底层是并行，这是工程上很划算的一处设计。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**World State：用差分喂环境，保住 prefix cache。** Shell 权限、AGENTS.md 模式这类环境信息每轮都可能变，全量重读会浪费窗口。Codex 把"当前世界"拆成一组**带类型的 Section**，每个 Section 有稳定 ID、快照类型，以及与上次相比的 Diff；初始化读全量，后续变更只注入"AGENTS.md 改了哪几处"这类补丁语句，模型结合前文全文理解变化。好处是**不破坏 Prefix KV Cache**、降低窗口消耗。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**SubAgent 可嵌套 + Mailbox 两种语义。** 主 Agent 可 Spawn 子 Agent，子 Agent 还能再 Spawn。Fork Turns 有三种上下文模式：全部继承（做并行"影分身"）、继承最近三轮（保留近期但清理上下文）、无上下文（彻底隔离的干净子 Agent）。Agent 间通信走 Mailbox：**Send Message** 只入信箱不触发对方新轮次，消息等对方下次有轮次时投递——不打断，但对方不再开轮次就会滞留；**Follow-up Task** 入信箱同时若对方空闲则唤醒一轮，若在运行则在消息边界或当前工具调用完成后投递，**不硬打断正在进行的推理**。两种都保留，按"要不要惊动对方"选。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**arg0 决定身份。** 同一个二进制被不同名字调用即不同角色：`codex` 是主 CLI Agent，`codex-linux-sandbox` 是 Linux 沙箱助手，`apply-patch` 是打补丁工具——启动时通过 arg0 传入不同值让模型知道"我现在是谁"。这是把**身份注入做成进程级事实**而非提示词约定，与身份层"模型不能指定任意人"的思路同向。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**压缩：把"总结"改写成"交接文档"。** 旧机制有四个触发点：窗口 80% 的 compact limit、阈值更低的软压缩、模型切换（不同模型提示词不同，需重建上下文）、切到窗口更小的模型。策略上不要求模型"总结并删除无关信息"，而是告诉它**"你要把当前工作交接给另一个模型"**——模型因此会写得完整、结构化、抓重点，效果优于干瘪 Summary。压缩**从最早的消息开始**，前缀 KV cache 只丢一次，下一轮基于压缩后的轮次重建缓存。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**长期 Memory：两阶段 + 可遗忘。** ①Rollout Extraction 从旧会话轨迹里提取重要信息，交模型产出需记录内容，脱敏后入库；②Global Consolidation 做 Top-N 筛选，按长期运行中的**被引用次数（Citation）**排序，引用越多排名越前、召回概率越大——不常用的信息自然被遗忘，形成记忆衰减，作者把它读作"为自进化服务"的机制。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**Token Budget：跨窗口的短期记忆（8 月更新）。** 传统压缩的毛病是"让压缩模型猜哪些信息以后有用"。Token Budget 改为**让主 Agent 自己管**：仍在阈值触发，但不做压缩，而是**新开一个 Context Window**（有 ID、序号递增、内容清空），衔接靠三件套——`Thread Hint`（后端取约 4000+ 字节，注入新窗口的 Developer Message，只写"上个窗口做到哪"，详细内容不进窗口）、`Notes`（模型私人笔记本，服务端托管的**虚拟文件系统**：list/read/字面搜索/append/create，上一窗口持续记录改了哪些文件、卡在哪、哪里失败、哪些测试有问题）、`History`（完整 Agent Trace 全量留存，提供 list 窗口/条目、读全文、搜索等工具，供渐进式下钻）。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**两者正交。** 长期 Memory 存数据库与磁盘、跨 Session 积累；Token Budget 是**单 Session 内**的机制，只管当前任务做到哪一步。作者的关键判断是：Token Budget 名义上解决压缩，形式上已经是 Memory——**不再压缩上下文，而是把上下文整理成按需读取的文件**。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**判读一：这套设计的共同母题是"不要替模型猜，把判断权还给持有上下文的那一方"。** World State 不重读全文而是给差分、压缩不说"总结"而说"交接"、Token Budget 不让压缩模型猜重要性而让主 Agent 自己写笔记——三处都在减少一次**信息有损的中介转换**。反过来看，代价也一致：都要求更强的写入纪律（Section 快照、交接文档、Notes），纪律松了就会退化成"窗口更大但更乱"。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]

**判读二：Notes + History 组合其实是"记忆分层的最小实现"。** Thread Hint 是索引（约 4KB，只说做到哪）、Notes 是结构化工作记忆（模型自写的进度与坑）、History 是不可变事实层（全量 Trace，可搜索可下钻）。这个三层结构与个人 Agent 实践里的"稳定背景 / 当前推进 / 完整历史 + 索引按需"同构，区别只在 Codex 把它做成了**单会话内的自动机制**，而人类实践靠人写约定。^[raw/articles/codex-harness-token-budget-window-memory-qianwen-2026-10-27.md]


---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

