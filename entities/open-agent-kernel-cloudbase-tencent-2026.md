---
title: "OpenAgentKernel：腾讯云 CloudBase 的 Agent 开发框架层"
type: entity
created: "2026-08-03"
updated: 2026-09-14
tags: [wechat, agent, framework, cloudbase, mcp, hitl, sandbox]
rating: v7c8
confidence: 0.8
provenance_state: extracted
sources:
  - raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# OpenAgentKernel：腾讯云 CloudBase 的 Agent 开发框架层

**来源**: 腾讯云开发CloudBase（云开发团队）

**发布日期**: 2026-08-03

**原文链接**: https://mp.weixin.qq.com/s/cxPY_AQgp5ci5nWI2RNTRA

**仓库**: https://github.com/TencentCloudBase/OpenAgentKernel（npm: `@cloudbase/open-agent-kernel@beta`）

## 摘要

腾讯云原生适配 CloudBase 平台的 Agent 开发框架，定位类比 Web 时代的 Django/Rails——把 Agent 开发中 70% 的重复性底层工作（会话历史与压缩、断线重连、沙箱对接、MCP/Skills 对接、HITL、模型多轮循环）标准化内置，让开发者专注业务逻辑。核心 API 为 `createAgent()` + `startSession()` + `session.send()` 的事件流模式，可部署到 CloudBase HTTP 云函数运行。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

## 五大内置能力

### 1. 会话持久化与跨进程恢复

会话记录默认持久化到云开发数据库（不依赖进程内存），通过 `conversationId` 在任意进程恢复上下文：`agent.resumeSession(conversationId)`。适配 Serverless 场景的多次函数调用。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

### 2. MCP 接入（三模式）

内置进程内（`createSdkMcpServer` 直接注册 tool）、本地 stdio（`npx` 子进程）、远程 HTTP（带 headers 鉴权）三种 MCP server 接入方式，统一通过 `mcpServers` 配置项声明。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

### 3. 人机响应（HITL）

`permissions.requireApproval` 声明敏感工具（如 `database_delete`、`reset_password`），命中时事件流触发 `tool_approval_required` 事件，开发者通过 `session.respondApproval()`（携带 `toolUseId`）实现人工确认交互。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

### 4. Agent 记忆

`userMemory: true` 开启后，用户私有 `.claude/` 记忆文件自动同步到 CloudBase 云存储并跨会话生效；也提供 `writeUserMemoryFiles` API 主动预置/删除记忆。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

### 5. 沙箱（Sandbox）

`sandbox: { enabled, cloudbaseTools, scope: shared|session, ttl }` 提供隔离执行环境跑代码/读写文件，沙箱内置 CloudBase 密钥可直接部署产物到云端，开发者无需管理调度。CloudBase Sandbox 能力当前内测中。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

## 内部自用案例

- **CloudBase Agent**：云开发控制台 Agent 板块基于 OpenAgentKernel 焕新，官方 `cloudbase-agent` 模板开箱即用（会话持久化/MCP/人工审批），`tcb fn code download` 本地定制 + `tcb fn deploy` 一键部署。控制台负责托管/日志/调试，框架负责运行时。
- **Issue Agent**：CNB 社区 Issue Bot 跑在流水线里，进程内 MCP server 提供受控工具（最终答复必须通过 `reply` 工具发布，机制上保证"有据可查或明确标注推断"）+ skills 加载答疑经验 + 多模态附件识别报错截图（视觉会话逐张识别后进主流程）。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

## 深度分析

### 会话持久化：durability 成为内核原语

手写 Agent loop 时，消息数组、工具调用历史、working set 全是进程内存里的变量；一旦崩在「工具已调用、结果未回填」的中间态，状态即丢失，若已产生外部副作用，重跑就是重复执行。OpenAgentKernel 把会话默认写入云开发数据库并用 `conversationId` 在任意后续进程 `resumeSession()` 恢复。Serverless 下这不是增益而是前提：每次 HTTP 云函数调用都是新进程。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

隐含约束更重要：要幂等恢复，状态必须外部化并 checkpoint，不能靠内存兜底。内核只管存取，幂等键、副作用去重与断点语义仍在业务侧——只有开发者知道哪个工具可安全重放。durability 由此从应用逻辑升为基础设施契约，也解释了它与 [[concepts/agent-orchestration-patterns|Agent 编排模式]] 中长任务形态的契合。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

### MCP 接入：能力获取，而非定制连接器

内核内置进程内（`createSdkMcpServer`）、本地 stdio（`npx`）、远程 HTTP（`headers` 鉴权）三种接入，统一由 `mcpServers` 声明。相对「每个 SaaS 手写连接器」，收益不只省代码：tool schema 成为统一能力接口面，加能力等于挂一个 [[concepts/model-context-protocol-mcp|MCP server]]，不改 Agent 主体逻辑。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

代价同源：每次接入都在扩大动作空间（blast radius）。远程 HTTP 依赖 bearer 凭据，信任边界与轮换归开发者；stdio 把第三方代码拉到本地进程旁；进程内 server 与 Agent 共进程、失败互相污染。三者风险递减、成本递增，选型实质是回答「这个工具值多大的信任半径」，因此必须与人机审批配套（参见 [[concepts/agent-security-architecture|Agent 安全架构]]）。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

### HITL：从 prompt 约定到控制流原语

用 `permissions.requireApproval` 声明敏感工具（如 `database_delete`、`reset_password`），命中时事件流抛出 `tool_approval_required`，开发者凭 `toolUseId` 调 `session.respondApproval()` 放行或拒绝。这与在 system prompt 写「删除前请询问用户」本质不同：前者是挂在工具调用路径上的确定性闸门，模型绕不过；后者是概率性约束，由可能被上下文注入操纵的模型自行判断。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

审批成为事件流的一类事件后，interrupt/resume 必须成立：人工确认可能隔几分钟或几天，会话要跨这个窗口存活——问题又回到持久化原语。内核只给机制，超时与升级策略（等多久算拒绝、拒绝后回退还是转人工）仍属开发者。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

### 记忆的作用域与确定性边界

`userMemory: true` 把用户私有 `.claude/` 记忆文件同步到云存储并跨会话生效，也可用 `writeUserMemoryFiles` 主动预置或删除。首先要定作用域：本会话的临时事实、本用户的长期偏好、本组织的共享知识若混存一处，就会出现跨用户串味或跨会话遗忘这类难复现故障。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

第二个轴是确定性：会话状态（谁在等谁、工具执行到哪步）必须 deterministic、可回放、可审计，它属于控制流；记忆检索本质 probabilistic，允许模糊近似。两者混谈，重启后就会得到「看起来一样、行为不同」的会话（参见 [[concepts/agent-memory-architecture|Agent 记忆架构]]）。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

### 沙箱与最小 API：安全边界与产品立场

`sandbox: { enabled, cloudbaseTools, scope, ttl }` 提供隔离环境跑代码、读写文件、执行 Shell。这必须在内核级解决：被执行代码是模型现场生成的不可信产物，任何依赖生成代码自身配合的约束都会在幻觉或越狱下失效；`ttl` 限资源与生命周期，`scope: session` 以独立实例换隔离、`shared` 以共享换成本。路径 allowlist 与 egress 控制仍需开发者决策，而 `cloudbaseTools: true` 把 CloudBase 密钥放进沙箱换取直接部署——便利极高，同时把凭据面推进沙箱。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

三十秒看懂的最小 API（`createAgent` → `startSession` → `send`）是产品立场而非技术必然：它定义了框架为谁服务——跑在 CloudBase 上、接受平台托管的开发者；也定义了在哪停下——自托管运行时、多云可移植与深度调度定制都在边界外。内部自用案例（控制台 CloudBase Agent 模板、CNB 社区 Issue Agent）说明这是 internal-first 框架：先消化自家平台的重复劳动再开源，路径经实战但抽象偏向自家形态。边界要如实承认：框架锁定与 [[entities/the-new-ai-lock-in|AI lock-in]] 风险，以及 eval、prompt/harness 设计这些内核不碰、也不可能替你做的工作（参见 [[concepts/evaluation-harness-design|评估 Harness 设计]]、[[concepts/agentic-engineering-paradigm|Agentic 工程范式]]）。^[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03.md]

## 实践启示

1. **先定 checkpoint 与 resume 契约，再写循环**：把「崩在工具调用中途」当默认场景，凡改写外部世界的工具必须可安全重放或显式标记不可重放，并配幂等键。
2. **用 `requireApproval` 覆盖不可逆操作，别在 prompt 里写「请先确认」**：审批按 blast radius 分级（删除/付款/改权限 → 硬闸门，只读 → 免审批），并明确超时与拒绝后的回退路径。
3. **按信任半径分层接入 MCP**：进程内 > 本地 stdio > 远程 HTTP；远程凭据走 `headers` 注入并纳入轮换与最小权限，第三方 stdio server 需锁定来源与版本。
4. **记忆按作用域拆开建模**：会话状态 deterministic 可回放，用户/组织记忆走检索；预置记忆用 `writeUserMemoryFiles` 显式写入，避免记忆变黑箱。
5. **沙箱参数就是安全策略**：`ttl` 限滥用窗口，高隔离选 `scope: session`，一旦开 `cloudbaseTools` 就承认沙箱内有可用凭据，需额外 allowlist 与 egress 策略兜底。
6. **清楚框架不做什么**：eval、prompt/harness 设计、审批升级流程、跨云可移植性都归开发者；若可能迁移运行时，就别让业务逻辑依赖内核私有约定。

## 相关链接

- → [[raw/articles/open-agent-kernel-cloudbase-tencent-2026-08-03|原文存档]]
- 同平台姊妹篇：[[entities/agent-loop-engineering-handbook-8-questions-chen-jin-tencent-self-2026|Agent Loop Engineering 手册（腾讯）]]、[[entities/workbuddy-product-framework-agent-harness-anne-2026|WorkBuddy 产品实践：从模型到 Harness 的 Agent 可用产品架构（腾讯技术工程）]]
- 相关主题：[[entities/agent-harness-12-components-7-decisions|Agent Harness 12 组件 7 决策]]、[[entities/专为-managed-agents-而生的-harness-底座agentscope-20|AgentScope 2.0：Managed Agents 底座 Harness]]
