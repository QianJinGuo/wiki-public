---
title: "我用阿里 AgentScope 复刻了一个 WorkBuddy — 从开源框架到可运行 Agent 的实践拆解"
created: 2026-07-22
updated: 2026-09-28
type: entity
tags: [agent, agentscope, workbuddy, multi-agent, harness-engineering, tool-management, permission, model-config, mcp, skill-loader, practice, tutorial]
sources:
  - raw/articles/我用阿里-agentscope-复刻了一个-workbuddy
  - raw/articles/我用阿里-agentscope-复刻了一个-workbuddy-mcp-skills管理
confidence: 0.75
rating: v7c7
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 我用阿里 AgentScope 复刻了一个 WorkBuddy — 从开源框架到可运行 Agent 的实践拆解

## 核心定位

本文来自叶小钗，是一篇 AgentScope Python 框架的实践教程。作者使用阿里开源的 AgentScope 框架（Python 版），完整复刻了 WorkBuddy 的核心工作流，涵盖模型配置管理、Toolkit 工具系统、权限控制、工作目录管理和前端交互等关键模块。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

与现有 [[entities/workbuddy专家团提示词全曝光多agent协作原来是这样产品化的|WorkBuddy 产品架构]] 文章从提示词和产品化角度切入不同，本文聚焦于**工程实现层面**——如何在 AgentScope 框架上构建一个具备完整功能的 Agent 产品。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

## 关键技术点

### 模型配置管理

AgentScope 的模型层由 Credential（API 凭证）和 ChatModel（模型对象）组成。实际产品化的 mini-WorkBuddy 需要在此之上添加模型配置管理——用户在前端添加多个模型、设置默认模型、在聊天时切换模型。项目将配置保存在 `workspace/models.json` 文件中，每轮聊天请求按用户选择的 model_profile 创建对应的模型客户端。为保证模型切换即时生效，系统将模型配置 ID 放入 Agent 缓存 key 中，切换后旧缓存失效，自动创建新模型 Agent。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

### Toolkit 工具系统

AgentScope 的Toolkit 是工具容器和调用入口，管理四类工具：
- 普通 Tool（Read、Write、Bash 等）
- MCP Client 提供的远程工具
- Skill Loader 加载的 Agent Skills
- Tool Group 动态启用/停用的工具组^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

模型通过 ReAct 循环调用工具，Toolkit 执行工具解析、权限判断、流式结果返回全流程。mini-WorkBuddy 显式选择了文件读取/搜索/编辑、Bash 执行等6个基础工具。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

### 权限系统五种模式

AgentScope 2.0.4 提供五种 PermissionMode：
- **DEFAULT** — 最保守，所有操作都需确认
- **ACCEPT_EDITS** — 放行工作目录内的读写操作
- **EXPLORE** — 严格只读，Write/Edit 直接拒绝
- **BYPASS** — 跳过确认（高风险）
- **DONT_ASK** — 无人值守任务，需询问的操作直接拒绝^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

mini-WorkBuddy 简化为用户可选的三种模式：默认（DEFAULT）、自动审批（ACCEPT_EDITS）、完全放行（BYPASS），并使用 PermissionRule 支持自定义规则（如"uv run pytest 始终允许""rm 命令直接拒绝"）。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

### 工作目录隔离

项目通过两层机制控制 Agent 的文件操作范围：一是 Bash(cwd=workdir) 指定命令起始目录，二是 PermissionContext.working_directories 限定读写权限范围，防止 Agent 修改工作目录外的文件。长期记忆目录和 Skill 目录也按需加入权限范围。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

### 工具审批流

当权限结果为 ASK 时，AgentScope 将工具调用记录到 AgentState 并产生 RequireUserConfirmEvent。mini-WorkBuddy 定义了自己的 PendingApproval 结构记录原 Agent、待确认工具、原始请求和已流式内容，用户确认后后端构造 UserConfirmResultEvent 交回原 Agent 继续执行。模型一次发出多个并行工具调用时，按 reply ID 保存 ConfirmationBatch 统一处理。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

### 工具调用可视化

AgentScope 通过 ToolCallStartEvent、ToolCallDeltaEvent、ToolCallEndEvent 流式输出工具名称与参数。后端按 tool call ID 聚合参数片段，前端据此更新工具卡片显示工具名称、格式化参数、执行状态和结果。需要审批的工具从 RequireUserConfirmEvent 中取出 ToolCallBlock 放入审批卡片展示。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

## 与既有内容的关系

- [[entities/workbuddy专家团提示词全曝光多agent协作原来是这样产品化的|WorkBuddy 产品架构]] — 提示词与产品化视角，本文补充了工程实现视角
- [[entities/agentscope-java-harness-framework|AgentScope Java Harness]] — AgentScope Java 版的企业级 Harness 实现，本文对应 Python 版实践
- [[entities/mem0-vs-workbuddy-agent-memory-comparison|WorkBuddy 记忆对比]] — 记忆层面的比较分析
- [[entities/openclaw-workbuddy-loop-engineering-who-is-hot-useful-demo|OpenClaw vs WorkBuddy]] — 工作流引擎对比

## 深度分析

### 框架层与产品自建层的边界

AgentScope 只提供模型对象、Toolkit、权限引擎这些框架原语，而产品化所需的状态管理——models.json、connectors.json、skills.json 及其增删改查——全部由项目自建的 WorkspaceStore 承担。这条边界划得非常清楚：凡是需要跨请求存活的用户配置，框架一概不管。作者用"无数据库 + JSON 文件 + 临时文件 os.replace 原子写入"的方案换取小产品的简单性，API 层从不直接碰 JSON，运行时也统一从 WorkspaceStore 读配置再交给 AgentScope。这给选型者的启示是：评估 Agent 框架时先看它把哪些问题留给了应用层——留白多意味着接入成本高，但也意味着框架更薄、更可控。参见 [[entities/agentscope-builder-enterprise-self-evolving-agent-harness|AgentScope Builder]]。

### 缓存 key 即一致性机制

文章两篇里反复出现同一个模式：模型、连接器、显式 Skill 这三类用户可切换的资源，全部被编码进 Agent 缓存 key（`base_key:connectors:rev` / `:skill:id`）。本质上是用"缓存不可命中即重建 Agent"代替"增量更新"——直接丢弃绑定了旧模型客户端、旧 MCP Client 或旧 Skill Loader 的 Agent，避免页面状态与 Agent 内部绑定出现静默漂移。这是无状态 Web 后端与有状态 Agent 对象之间一座成本低廉的桥。有趣的是折中设计：动态发现类资源（自动发现模式下已启用的 Skill 列表）刻意不进 key，靠动态 Loader 在运行时重读启用列表、权限上下文随 Agent 复用同步更新来兜底——只有用户显式选择的资源才值得支付重建代价。

### 权限模型的三层收敛

AgentScope 的权限体系实际有三层：PermissionMode（5 档交互策略）、PermissionRule（工具名 + 命令/路径的细粒度 allow/deny/ask 规则，存于 PermissionContext）、以及工具自检（只读工具返回 PASSTHROUGH 交规则裁决，MCP 工具按 Server 返回的 readOnlyHint 决定 ALLOW 或 ASK）。mini-WorkBuddy 把 5 档收敛为 3 档暴露给用户，而作者实测结论是"自动审批"（ACCEPT_EDITS）才是日常档位——DEFAULT 模式下 Agent 每步都要点确认会把用户逼走，BYPASS 风险高到几乎不用。这个产品化裁剪与 WorkBuddy 本尊只保留"默认 + 完全放行"两档的取舍互相印证：权限档位不是越多越好，关键是把高频路径做到零摩擦、把危险路径（rm、写 shell 配置）拦死，再用 PermissionRule 补个性化例外（如 `uv run pytest` 恒允许）。

### STDIO/HTTP 双传输的生命周期分化

MCP 接入中最有含金量的细节是传输方式决定客户端形态（背景见 [[concepts/model-context-protocol-mcp|MCP]]）：STDIO Server 是本地子进程，必须用 Stateful Client 并在创建 Agent 前显式 `connect()`，让后续多次工具调用复用同一进程；HTTP Server 独立运行，用 Stateless Client，由框架按需管理临时会话。更进一步，作者没有直接用 AgentScope 的 MCPClient，而是实现了 RequestSafeMCPClient 子类——把 `connect()`/`close()` 固定在一个长期存活的 owner task 中执行。原因是 AgentScope 底层用 AnyIO 管理 STDIO 连接，cancel scope 不能跨越任务：在 A 请求里 connect、B 请求里 close 会破坏 Starlette 的任务状态。这是"框架默认实现 × Web 服务器并发模型 = 隐性冲突"的典型案例，任何把 Agent 框架嵌入 SSE 流式后端的项目都会撞上。

### Skill 渐进式披露与"目录即真相"

Skill 体系继承并落地了渐进式披露设计：Toolkit 只把名称、简介交给模型，完整的 SKILL.md 由一个特殊 Skill 工具按需读取，避免十几个技能的全文一开始就挤进上下文、规则互相干扰。产品层面则体现"目录即真相"：skills.json 只是索引，扫描时发现目录里有 SKILL.md 但不在索引中的，会被自动识别为 manual 安装——用户从别处复制技能进目录即可用，无需改索引。审批与权限也贯穿到技能：Skill 目录（用户选择的、自动启用的、专家包携带的）都要加入 PermissionContext.working_directories，否则模型按技能说明执行 `scripts/run.py` 时会因目录越界被拒。最值得注意的是闭环设计：添加技能这个产品功能本身就是一个内置 skill（skill-creator 一字未改地复制自 WorkBuddy），用 Agent 的方式扩展 Agent。参见 [[entities/workbuddy-skill-全拆解从创建到自进化|WorkBuddy Skill 全拆解]] 与 [[concepts/skill-engineering-principles|Skill 工程原则]]。

## 四层工具架构

mini-WorkBuddy 的工具组织为四层：基础 Tool（Read/Write/Bash）→ MCP 接入 → Skills/Skill Loader → Tool Group 动态分组。这套结构为后续拆解"专家"和"专家团"能力提供了可扩展的工具基础。^[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy.md]

→ [[raw/articles/我用阿里-agentscope-复刻了一个-workbuddy|原文存档]]
