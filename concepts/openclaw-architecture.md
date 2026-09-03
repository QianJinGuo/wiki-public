---
title: OpenClaw 架构解析（800行实现）
created: 2026-04-24
updated: 2026-08-01
type: concept
tags: [openclaw, architecture, tool, message-bus, subagent, typescript, anthropic, claude-api, agent-framework, replit, taobao]
sources: ['raw/articles/openclaw-architecture-800lines']
confidence: high
---
# OpenClaw 架构解析（800行实现）
## Overview
淘天集团会员技术团队对 OpenClaw 核心架构的代码级拆解，聚焦 Tool 系统、MessageBus、SubagentManager、REPL 主循环四个模块。核心观点：**薄抽象 + 显式控制流 + 贴近模型 API = 工程确定性**。

OpenClaw 是 [[concepts/open-source-ai-ecosystem|开源 AI 生态系统]] 的代表性框架之一，与 OWL、Agentium 并列为当前最具影响力的三大开源 Agent 框架。参见 [[concepts/open-source-ai-ecosystem|Open Source AI Ecosystem]] 的生态全景对比。
文章作者：苏雄（淘天集团-会员技术团队），业务负责 88VIP、天猫积分、省钱卡等淘宝核心会员体系。
## 四大核心模块
### 1. Tool 层
**Tool 抽象类**四要素：`name` / `description` / `input_schema` / `execute`
```typescript
export abstract class Tool {
  abstract readonly name: string;
  abstract readonly description: string;
  abstract readonly input_schema: AnthropicTool["input_schema"];
  abstract execute(args: Record<string, unknown>): Promise<unknown>;
  toSchema(): AnthropicTool { /* 转换为 Anthropic API schema */ }
}
```
**设计取舍**：schema 用运行时对象而非 Zod，好处是零依赖、直接对齐 SDK；代价是无运行时参数校验（错误只在 execute 阶段暴露）。
**ToolRegistry**：`Map<string, Tool>`，核心方法：
| 方法 | 作用 |
|------|------|
| `register(tool)` | 注册工具 |
| `execute(name, args)` | 调用工具 |
| `getToolDefinition()` | 导出 Anthropic function calling schema |
| `exclude(names)` | 返回剔除指定工具的新注册表（为子 Agent 生成受限工具集）|
**内置工具一览**：
| 类别 | 工具 | 关键设计 |
|------|------|---------|
| 文件 | ReadFileTool | 动态 `import("node:fs/promises")` |
| 文件 | WriteFileTool | 自动 `mkdir(recursive: true)` 创建父目录 |
| 文件 | EditFileTool | **强制唯一匹配**：0次报错，>1次拒绝写入并要求更精确片段 |
| 文件 | ListDirTool | stat 区分 `[folder]` / `[file]` |
| 命令 | ExecTool | 三层：正则黑名单 → 超时/内存限制 → 输出截断首尾各5KB |
| 网络 | WebSearchTool | 封装 Brave Search API |
| 网络 | WebFetchTool | 纯正则 htmlToText，无 DOM 解析库 |
| 通信 | MessageTool | 注入 `sendCallback`，出站方向 |
| 调度 | CronTool + CronService | setInterval，支持 `*/N` / 每小时 / 每天 |
**ExecTool 三层安全防护**：
1. 正则黑名单（`rm -rf /`、fork bomb、写裸设备等）
2. 资源限制（默认 30s 超时，2MB maxBuffer）
3. 输出截断（保留首尾各 5KB，因为末尾通常含最有价值的信息）
### 2. MessageBus 入站消息总线
数据结构 `InboundMessage`：channel / senderId / chatId / content
```typescript
export class MessageBus {
  subscribe(channel, handler): () => void  // 实时回调
  drain(channel?): InboundMessage[]          // 队列轮询
  publish(message): Promise<void>           // 发布消息
}
```
两种消费模式，有订阅者走回调，无订阅者入队列。消息只走一条路径。
**与 MessageTool 的关系**：MessageTool = 出站（Agent → 外部），MessageBus = 入站（外部/子系统 → Agent）。两者方向相反，无代码耦合。
### 3. SubagentManager 后台子 Agent
**并发模型**：单进程 Promise 并发，共享 Node.js 事件循环，无多进程/Worker。
**工具集隔离**：通过 `exclude(["spawn", "message", "edit_file", "cron"])` 生成受限子集。
- 排除 `spawn`：防止递归创建子 Agent
- 排除 `message`：子 Agent 不能直接向用户发消息，应通过 MessageBus 回传
- 排除 `edit_file`：限制写入能力
- 排除 `cron`：避免子 Agent 创建定时任务
**生命周期**：`spawn()` 分配自增 ID → 启动独立 `AgentLoop` → 立即返回 ID（调用方不等待）。子 Agent 最大迭代 15 次（主 Agent 是 10 次）。
**结果回传**：通过 `bus.publish("system", ...)` 发送 `[Subagent completed]` 通知，`promise.finally()` 自动清理 `runningTasks`。
### 4. REPL 主循环
**并发控制**：布尔互斥锁 + 暂存队列，保证 shared history 不被并发修改。
```typescript
let processing = false;
const pendingSubagentResults: InboundMessage[] = [];
async function tryDrainPending() {
  if (processing) return;  // 互斥锁
  processing = true;
  try { await drainPendingResults(); }
  finally { processing = false; }
  rl.prompt();
}
```
用户输入和子 Agent 结果都可能触发 `agent.run()`，锁 + 队列确保 history 一致性。
**异步汇入路径**：
```
SpawnTool / CronService
  → bus.publish("system", 结果)
  → handler 入队 pendingSubagentResults
  → tryDrainPending()
  → agent.run([SYSTEM NOTIFICATION])
  → 主 Agent 总结后输出
```
**MessageTool 的接线**：REPL 场景下 `sendCallback = console.log`；Bot 场景只需替换为平台发送函数，工具系统和 Agent 逻辑无需改动。
## 数据流全景
```
终端 stdin
  │
  ▼
REPL 主循环
  │  ┌─ history (共享，互斥访问)
  ▼  ▼
AgentLoop.run() ──→ Tool 调用
  │        ├── 文件/命令/网络 → 直接返回
  │        ├── SpawnTool → SubagentManager.spawn()
  │        │               └── 子AgentLoop → bus.publish("system", 结果)
  │        ├── MessageTool → sendCallback → stdout
  │        └── CronTool → CronService → bus.publish("system", 触发)
  │
  │ ◄── bus.subscribe("system") ◄── pendingSubagentResults
  ▼
stdout
```
两条主线：
- **同步路径**：用户输入 → REPL → agent.run() → 工具 → 结果回传 → 最终回复 → stdout
- **异步路径**：SpawnTool/CronService → bus.publish() → 入队 → tryDrainPending() → 主 Agent 处理
## 设计取舍总结
| 决策 | 好处 | 代价/局限 |
|------|------|---------|
| 零框架依赖，直接基于 Anthropic SDK | 完全控制，不穿透框架抽象 | 基础能力需自实现 |
| schema 用运行时对象而非 Zod | 零依赖，对齐 SDK 类型 | 无运行时参数校验 |
| 子 Agent 无持久记忆 | 实现简单，适合并行任务 | 不适合跨任务积累上下文 |
| CronService 简化 cron 解析 | 无需引入 cron 库 | 复杂表达式静默降级为每分钟 |
| MessageBus 无持久化 | 实现简单 | 进程重启后队列丢失 |
| ExecTool 正则黑名单 | 低开销第一道防线 | 可被变量展开/别名绕过 |
| REPL 布尔锁并发 | 单用户场景足够 | 多用户 Bot 需独立队列/锁 |
## 核心洞察
> **薄抽象 + 显式控制流 + 贴近模型 API = 800 行代码的确定性 Agent 运行时。**
扩展新工具：继承 `Tool` 并注册。切换接入层（REPL → Bot）：替换 `sendCallback` 和输入源。子 Agent 能力边界：通过 `exclude()` 控制。
## OpenClaw 与 Claude Code 的记忆系统设计分歧
OpenClaw 和 Claude Code（参见 [[concepts/claude-code-deep-architecture-analysis|Claude Code 架构深度分析]]）在记忆系统上的设计选择代表了 Agent 框架的两个极端：OpenClaw 的子 Agent **完全没有持久记忆**，而 Claude Code 维护了七层上下文压缩体系。这个分歧的根源在于两者的设计优先级不同。

OpenClaw 的哲学是**显式优于隐式**：Agent 的状态完全来自当次对话的上下文和工具调用的结果，没有任何隐式的向量存储或记忆压缩机制。当子 Agent 完成后，它的工作结果通过 MessageBus 的出站通知传递，AgentLoop 的主 Agent 收到后负责解读和总结——这个过程是显式的、人工编排的。相比之下，Claude Code 的记忆系统更接近[[entities/context-engineering-three-memory-paradigms|Context Engineering 中的外部记忆方案]]：上下文窗口是有限的，但框架通过多层压缩（tool result trimming → summary rewriting）尽可能保留决策相关的信息。

对 Agent 工程实践的启示是：OpenClaw 的"无隐式记忆"设计实际上是更安全的——因为隐式记忆系统（如向量数据库存储历史对话）可能成为 prompt injection 的持久化载体。OpenClaw 的子 Agent 在 `exclude()` 工具隔离之上额外保证了它无法通过 message 工具向外持久化任何内容，所有结果必须通过 MessageBus 的内存通知传递，进程重启即丢失。这在 [[entities/openclaw-security-and-feature-enhancement-practices|OpenClaw 安全实践]] 中被描述为"Agent 安全模型的核心挑战"——当 OpenClaw Agent 拥有执行权限时，任何持久化能力都是潜在的攻击面。

从 [[entities/hermes-agent-self-evolving|Hermes Agent]] 的设计来看，企业级 Agent 框架普遍选择走中间路线：保留记忆能力但加上 prompt injection 的正则防护层。这说明 OpenClaw 的"无记忆=安全"逻辑在某些场景下过于严格——如果业务需要 Agent 记住用户的偏好和历史交互，就必须找到办法在保持安全的同时实现持久化。
## REPL 主循环作为事件处理核心：异步汇入模式的工程价值
OpenClaw 的 REPL 主循环表面上是一个简单的交互式入口，但它的异步汇入机制（用户输入和子 Agent 结果共享同一队列和锁）是整个框架中最具工程巧思的设计。这个模式在现代前端事件处理（事件循环、宏任务/微任务队列）中广泛存在，但被迁移到 Agent 运行时中时，它解决了一个实际问题：**如何让 Agent 能够并发处理多个来源的事件而不破坏对话历史的一致性**。

具体来说，tryDrainPending() 的互斥锁 + 队列设计保证了：
1. **历史一致性**：shared history 的写入是互斥的，不会出现子 Agent 结果和用户输入交错写入导致的上下文乱序
2. **非阻塞异步**：子 Agent 在后台运行时，主 Agent 不必等待——结果通过 bus.publish() 入队后，主 Agent 可以继续处理其他任务
3. **可重现性**：由于执行顺序完全由队列决定而非依赖时间戳，Agent 的完整执行历史是确定性的

这个设计与 Claude Code 的 git state injection（每次 turn 动态注入）形成对比：Claude Code 的上下文管理是**时间驱动的**（每轮对话自动压缩），而 OpenClaw 的上下文管理是**事件驱动的**（只在有消息时触发 drain）。前者适合需要持续探索代码库的交互式编程场景，后者适合需要后台任务处理的自动化运营场景。

从系统设计角度看，OpenClaw 的 MessageBus + tryDrainPending 模式本质上是[[entities/openclaw-multi-agent-team-practice|OpenClaw 多 Agent 协作]]的事件总线模式的缩小版——无论是一对一的主 Agent-子 Agent 通信，还是多 Bot 的聊天室场景，MessageBus 都是统一的消息路由基础设施。这个统一性意味着在 REPL 场景下开发的调试经验可以直接迁移到更复杂的 Bot 部署场景。
## Related
- [[entities/cat-wu-anthropic-pm-interview|Cat Wu — Anthropic Claude Code/Cowork PM 访谈]] — Cat Wu 谈 Anthropic 的发布速度、100% 自动化原则与 Evals
- [[entities/hermes-agent|Hermes Agent]] — OpenClaw 竞品，Nous Research 开源
- [[entities/claude-code-architecture|Claude Code 架构解析]] — 同样是 Agent 架构研究，Query Loop 模块可对照
- [[entities/memos-hermes-plugin|MemOS Hermes 插件]] — OpenClaw（Hermes）的记忆管理系统
- [[entities/openclaw-architecture-8-part-summary|OpenClaw 架构解析（8篇完整版）]] — 800行架构的详细8篇对照
- [[entities/openclaw-multi-agent-team-practice|OpenClaw 多 Agent 团队协作实践]] — 多 Bot 协作场景下的 MessageBus 扩展
- [[entities/openclaw-security-and-feature-enhancement-practices|OpenClaw 安全与功能增强实践]] — Agent 安全模型：执行权限 vs 传统 Web 安全
- [[entities/ai-agent-tool-count-trap|AI Agent 工具数量陷阱]] — 子 Agent 工具集隔离与 LLM 决策质量的量化关系
- [[entities/openclaw-comprehensive-guide-32k-chars|OpenClaw 完整指南（32K版）]] — OpenClaw 功能全景对照
- [[raw/articles/openclaw-architecture-800lines|原始文章存档]]

## 新增关联实体
- [[entities/claude-dispatch-interfaces-mollick]]
- [[entities/developers.googleblog-announcing-genkit-middleware-intercept-extend-and-harden-y]]

## 关联实体

**上游依赖**:
- [[entities/context-engineering-three-memory-paradigms]] — 提供基础理论/方法
- [[entities/openclaw-security-and-feature-enhancement-practices]] — 提供基础理论/方法
- [[entities/hermes-agent-self-evolving]] — 提供基础理论/方法

**下游应用**:
- [[entities/hermes-agent]] — 具体应用场景
- [[entities/claude-code-architecture]] — 具体应用场景
- [[entities/memos-hermes-plugin]] — 具体应用场景

**平行协作**:
- [[entities/openclaw-security-and-feature-enhancement-practices]] — 替代/补充方案
- [[entities/ai-agent-tool-count-trap]] — 替代/补充方案
- [[entities/openclaw-comprehensive-guide-32k-chars]] — 替代/补充方案

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
