---
title: "Agent Harness 演化论：从一次 LLM 调用到完整运行时（四框架机制拆解）"
created: 2026-09-07
updated: 2026-10-19
type: entity
tags: [harness, agent, architecture, evolution, tencent, pi, opencode, codex, hermes, runtime]
sources: [raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026]
confidence: 0.7
---

# Agent Harness 演化论：从一次 LLM 调用到完整运行时

> 腾讯技术工程（ivanxxie、davoszhang）系统性还原 Agent 从朴素模型调用逐层长出完整 Harness 的演化路径，并以 Pi、OpenCode、Codex、Hermes 四个框架为实例拆解 harness 设计空间，附 Composio/OpenBench/PAST-Bench 三套量化测评。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

## 核心洞察：Harness 是模型撞到边界后工程层一层层补上的回答

Agent 系统看起来像小型操作系统，但复杂度并非一次设计出来——每当模型试图越过一次调用边界，系统就在模型外面增加一个新部件：记不住历史 → 装配上下文；碰不到世界 → 接入工具；一步做不完 → 启动循环；历史装不下 → 分出长期记忆；真实副作用越来越多 → 长出会话、权限、沙箱、事件和子 Agent。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

**Agent 决定下一步做什么；Harness 决定这一步在什么上下文、权限、生命周期与持久化规则下发生。** 模型看见的一切都是运行时为它构造出来的——这是所有 Harness 的核心原则。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

## 演化路径：六层能力逐层叠加

1. **朴素 LLM**：`answer = LLM(question)`，一次前向调用，模型参数带知识但无跨轮记忆。
2. **Q&A Bot（上下文装配层）**：装配 System Prompt + Chat History + User Prompt，模型没突然获得记忆，是应用每次请求前重新装配历史。
3. **ReAct（循环层）**：Thought → Action → Observation 闭环，`while not finished` 是最早的 Agent Runtime——真正负责"继续运行"的是模型外部的程序。
4. **Tool Calling（结构化动作协议）**：2023 年 OpenAI Function Calling 结构化输出工具名+参数；2024 年 Anthropic MCP 标准化连接。工具通过 schema 注册，Tool Router 把"模型想做什么"翻译为确定的程序调用。
5. **Memory（跨会话积累）**：拆成本轮可见的 Working Memory 与模型外部可持久化的 Long-term Memory。RAG（2020）区分参数知识/外部检索；MemGPT（2023）借用操作系统分层存储。三类记忆：程序（程序性技能/SKILL.md）、语义（稳定事实/偏好）、情景（任务轨迹）。
6. **Harness（完整运行时）**：会话恢复、副作用安全、上下文压缩、多客户端、扩展装配、任务并行 6 个工程问题同时出现，Agent Loop 被包进更大运行时。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

## 四框架机制拆解：Harness 设计空间

### Pi：极简 runtime kernel
Pi 自称 minimal terminal coding harness，只暴露 read/write/edit/bash 四个工具；Resource Loader 显式装配当前会话的指令/Skills/Prompt Templates；Session Manager 用活动分支+Compaction 把完整会话投影成更紧的 Working Memory。结果：Composio 把 DeepSeek V4 Flash 放进 4 套 harness 跑 30 个 Agentic Tasks，Pi 通过率 66.7%、中位成本 $0.012（完成最多+花最少）；Databricks 追踪每轮上下文约少三倍、更少轮次——"极简不只审美，直接变成任务成本和执行效率"。代价：无内置权限系统，默认继承用户权限，高风险环境需外部沙箱。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

### OpenCode：事件驱动 + Agent Profile
OpenCode 保存的是 Agent 运行全过程——一条 Assistant Message 下挂 Reasoning/Text/Tool/Step Start/Step Finish/Patch/Compaction 等 Part；Session/Message/Part 通过 Projector 投影进 SQLite。Compaction、Title、Summary 是隐藏后台 Agent，就近上下文上限时汇总早史+裁剪旧工具输出。关键：**进程状态可结构化记录 → 退出界面≠丢失现场、Session 可恢复、行为可审计、子 Agent 可隔离、同一运行时被多界面消费**。代价：需维护 Agent 配置合并/权限组合/事件顺序/数据库投影/压缩边界。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

### Codex：安全执行两层边界 + Thread 生命周期
**Approval（某动作是否获准）与 Sandbox Policy（命令写到哪/能否上网/触碰哪些资源）是两道独立边界**——用户点允许≠模型拿到整台电脑。App Server 用 Thread/Turn/Item 三层协议模型承载可 Start/Resume/Fork/Interrupt/Steer 的任务生命周期；**Thread Manager 用 `HashMap<ThreadId, Arc<CodexThread>>` 记录活跃线程**，冷线程才从 Thread Store/Rollout 读取重建。StepContext 引用 TurnContext 捕获环境/Capability Roots/MCP Binding/Tool Router/AGENTS.md，让"已经开始的一步面对自洽的工具和环境"。子 Agent = Thread Manager 派生的子 Thread，可 fork 父任务持久化历史。成本量化：OpenBench 2026-07 把 gpt-5.6-sol 放进 7 套 harness 只比 42 个全跑任务，Codex 完成 31（73.8%）、中位 94.6s、每成功任务 117,107 新 token（最重档）。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

### Hermes：自进化（持久化认知系统）
Pi 关心"少花 harness tax 完成眼前任务"，Hermes 问"第二次遇到类似任务能否少走弯路"。前台 Agent Loop 解决当前任务，Session Archive 留原始经历，Memory+Skills 承载稳定事实/可复用流程，Background Review 决定哪些经验影响未来——**在模型外维护一套随使用变化的认知系统**。PAST-Bench（26 场景/204 跨会话 Episode）固定 MiniMax-M2.7 后 Hermes 是唯一四类能力（Memory/Procedural Reuse/Information Gathering/Update）全部正向增益的框架，总体 +0.13，机制证据分 0.64（对照同增益 nanobot 0.57）。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

## 设计空间对比：不同路线交换成本与控制力

| 维度 | Pi | OpenCode | Codex | Hermes |
|---|---|---|---|---|
| 核心策略 | 最小上下文 | 结构化状态 | 安全边界+线程生命周期 | 持久化认知系统 |
| 聚焦 | 压低 harness tax | 会话可恢复/可审计 | 长任务监管/恢复/并行 | 跨会话学习 |
| 代价 | 无权限系统 | 状态工程复杂 | 每任务成本最重 | 后台管线开销 |
| 量化锚点 | 66.7%/$0.012 | — | 73.8%/117K token | +0.13/0.64 |

不同路线交换的是**成本、控制力、可恢复性和长期学习能力**——选择取决于 Agent 最终进入的真实环境。模型越能行动，外面的运行时越不能含糊。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

## 与既有实体关系

- **补充而非替代**：与 `[[concepts/harness-engineering-framework|Harness Engineering Framework]]`（概念框架）、`[[entities/harness-engineering-exploration-tencent-tech|腾讯 Harness Engineering 探索]]`（出码率/提效视角）互补——本文提供**逐层演化路径 + 四框架机制级横向拆解**这一库内零覆盖维度。
- 与 `[[entities/pi-openclaw-coding-harness|Coding Harness 工程本质：从 Pi 到 OpenClaw]]` 互为姊妹：该篇深拆 Pi→OpenClaw 演进，本文横向对比 Pi/OpenCode/Codex/Hermes。
- Hermes 自进化维度呼应 `[[entities/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528|harness 作为训练参与方]]` 与 `[[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|长期运行 Agent 实践]]`。

## 深度分析

### 否定式定义：Harness 由模型的"不能"反向定义

Harness 的边界不是自己画的，是模型的失败模式画的，于是有个不常被说出的后果：**任何 Harness 改进都是暂定的**，它补的是模型此刻的缺口。文章自带证据——早期 ReAct 靠约定文本让程序用正则解析动作，Function Calling 把工具名与参数变成结构化输出后，整层"解析模型意图"的胶水就被删掉了。据此可把部件分两类：**会被吸收的**（提示词脚手架、推理模板、输出格式、单步重试）每随模型强一档而贬值；**不会被吸收的**（状态持久化、权限与沙箱、审计、验证与回滚）处理的是现实世界的不可逆性——文件被覆盖、命令执行过、凭证被读过，不因模型变聪明而可撤销。投资应压在后一类，前一类更该按 [[concepts/harness-component-expiry-and-build-to-delete|组件保质期与 build to delete]] 的原则来写。

### 六层阶梯是成本曲线，不是成熟度阶梯

把六层读成"能力等级"是错的：正确读法是**每一层都在用通用性换确定性，而交换比率逐层恶化**。前四层近乎无条件划算：装配历史只是字符串拼接，`while not finished` 是几十行代码，工具 schema 一旦标准化就被所有工具复用——成本近零，收益是"从不能行动到能行动"。拐点在记忆与完整运行时：记忆的难点不是存储，而是**写入与读取策略**（什么值得保存、重复事实何时合并、冲突谁覆盖、检索几条不挤掉当前任务、错记怎么纠正），每条都是买不到通用件的策略判断，且判断错了不会立刻报错。所以坍塌点不在某一层，而是一个条件：**当这一层的收益只能靠避免低频事故、或跨会话的第二次任务来体现时**，它就进入收益长尾，单次评测看不见。

### 四框架收敛于同一设计空间，分歧都在"状态住在哪"

四者在机制上高度收敛：一个循环控制器、一套工具协议、一个上下文管理器、一个往往隐式的验证者。真正的分歧只有两处，而这两处决定调试体验。**状态住哪**：Pi 是活动分支加 Compaction 后的会话投影，近乎无状态；OpenCode 投影进 SQLite，状态即可回放的事件流；Codex 是双层：内存一张活跃 Thread 表，冷任务才从 Rollout 重建；Hermes 把长期状态放进文件系统与会话归档。**谁拥有重试与终止**：依次是最小循环的收尾判断、Profile 的步数与权限、Turn/Item 协议状态机、后台 Review 管线。代价很具体：事件投影能回放到 Part 粒度，却新增事件顺序与投影一致性这类状态工程 bug；内存活跃表让 Resume 变快，代价是同一份历史不能起两套循环（Resume 命中活跃实例就直接返回）；文件式记忆可 diff、可纠正，但没有并发控制。选型该问的不是功能表，而是**半夜出问题时能不能定位到状态**。

### 可删除的工地：接口要窄，评估只能端到端

承认部件会过期，代码就该按"将来能整层删掉"写，而不是按"将来能继续加"写——这正是 [[entities/harness-engineering-deletable-worksite-ruofei|可删除的工地]] 的工程含义。它反向约束接口：上下文层只承诺给模型一段 token，工具层只承诺 schema 与 Tool Result 的身份对应，验证层只承诺可判定的成功/失败信号；契约越窄，模型升级时越能整层拿掉，不留半死分支。评估口径必须与接口同源：三套测评分别量通过率乘成本、共同子集完成率与每成功 token、跨会话增益加机制证据，都只在端到端成立，因为 Harness 质量是**成功率、成本、可恢复性的乘积**，任何单层指标都会被别层补偿（压缩强可掩盖检索差，权限厚可掩盖循环弱）。用层内指标（如压缩率）优化，几乎必然优化出别处漏水的系统；同模型同推理强度仅换 Harness 就能让单任务成本差两倍以上，也说明"模型贵不贵"在模型层面回答不了。参见 [[concepts/evaluation-harness-design|评估 Harness 设计]] 与 [[entities/harness-evolution-vs-test-time-scaling-2026|Harness Evolution vs 多跑几遍实证]]。

## 实践启示

- 从朴素调用出发按"模型撞到哪条边界就补哪一块"的顺序演进，不要一次设计成完整 OS。
- 想压低成本学 Pi 的极简（每轮上下文少→token/轮次少）；要生产级可恢复学 OpenCode 的结构化事件+SQLite；要安全监管学 Codex 的 Approval/Sandbox 双边界+Thread 生命周期；要长期学习学 Hermes 的 Memory+Skills+后台管线。
- 评估任何带 self-improvement 的框架，都应与"多跑几遍的 test-time scaling"公平比较（呼应 rethinking-harness-evolution 批判）。^[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026.md]

→ [[raw/articles/from-llm-call-to-complete-harness-tencent-tech-2026|原文存档]]