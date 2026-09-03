---
title: Hermes-Agent
created: 2026-04-24
updated: 2026-08-01
type: concept
tags: [open-source, agent, nous-research, self-evolving, cron-scheduler, skill-system, memory, persistent-process, hermes, session-management, message-gateway]
sources: ['raw/articles/agent-tools-research']
confidence: high
related:
  - [[entities/hermes-agent|Hermes-Agent 实体页]]
  - [[entities/cli-anything|CLI-Anything 工具集]]
  - [[concepts/hermes-agent-skill|Skill 机制]]
  - [[concepts/managed-agents-architecture|Managed Agents 架构]]
  - [[concepts/agent-memory-system-design|Agent Memory 设计]]
  - [[concepts/harness-engineering-framework|Harness Engineering 框架]]
  - [[concepts/openclaw-architecture|OpenClaw 架构]]
  - [[concepts/agent-memory-lifecycle-philosophies|Agent Memory 生命周期]]
---
# Hermes-Agent

> Hermes Agent 并不是一个绑定在 IDE 中的编程 Copilot，也不是仅封装了单一 API 的聊天机器人外壳。它是一个部署在服务器上的自主智能体，能够记住所学内容，并且运行时间越长，能力就越强。
> — Nous Research 官方定位

## 核心定位

[[entities/hermes-agent|Hermes-Agent]] 是由 Nous Research 于 2026 年 2 月底推出的开源 Agent 项目，GitHub 40k+ Stars，核心特性：

- **持久运行** — 常驻服务器进程，不限单次对话长度
- **自进化** — Skill 即时生成 + RL 权重微调双路径
- **多平台接入** — Telegram/Discord/Slack/飞书等消息通道
- **内置 Cron 调度器** — 定时任务不需要外部 cron 配置
- **40+ 内置工具** — 覆盖搜索、代码、文件、浏览器等场景

## 架构设计

### 进程模型：常驻 vs 按需

Hermes 采用**常驻进程模式**，这是它与 Claude Code 的根本区别：

| 维度 | Hermes | Claude Code | OpenClaw |
|------|--------|-------------|----------|
| 进程生命周期 | 常驻进程 | 每次调用新建 | 常驻进程 |
| 对话状态 | 写入记忆系统 | 无状态 | 写入文件系统 |
| 自进化能力 | Skill + RL 双路径 | 无 | Skill only |
| Cron 调度 | 内置 | 无 | 无 |
| 上下文积累 | 跨会话持续增长 | 每次重置 | 部分积累 |

常驻进程的价值：**上下文是复利**。每次对话都在同一个 Agent 实例中积累记忆、技能和经验，运行时间越长，Agent 越"懂你"。

### 记忆系统

Hermes 的记忆系统与 [[concepts/agent-memory-system-design|Agent Memory 设计]] 高度对齐：

- **向量检索** — 支持语义搜索历史上下文
- **层级衰减** — 重要记忆权重更高，低价值记忆自动淘汰
- **跨会话持久化** — 关闭后重启依然保留历史学习成果

记忆的持久化机制：每次对话结束后，Agent 将关键上下文写入向量数据库和文件系统双重存储，重启时从持久化存储恢复。这与 [[entities/harness不是目的知识才是护城河-一个ai工程交付团队的知识沉淀实践|知识护城河实践]] 中的"文件系统即状态机"哲学一致——**状态不存内存，存文件系统**。

### Session 管理

Hermes 的 Session 是隔离的上下文窗口：

```
[本地运行时路径已隐藏]
├── session-2026-05-21-abc123/
│   ├── context.json      # 当前会话上下文
│   ├── memory.json       # 会话级记忆（会话结束后合并到全局记忆）
│   └── state.json        # Agent 状态快照
```

- **Session 隔离**：不同 conversation 是不同 session，避免上下文污染
- **Session Search**：通过语义搜索历史 session，快速召回历史上下文
- **Session 继承**：可以基于历史 session 继续任务（类似断点续传）

## Skill 自进化系统

详见 [[concepts/hermes-agent-skill|Skill 机制]]，核心是内外双路径：

1. **内部路径（Skill 即时生成）** — 当遇到新任务场景，Agent 动态生成 Skill（Markdown 文件），下次同类任务直接复用
2. **外部路径（RL 微调）** — 将成功经验蒸馏到模型权重，需要更多计算资源但效果更持久

### 内部路径：Skill 即时生成

当 Hermes 执行任务时检测到新模式，会触发 Skill 生成：

1. **触发条件** — 同一类型任务出现 ≥3 次，或检测到手工重复步骤
2. **生成内容** — Markdown 文件，包含：意图描述、参数 Schema、执行步骤、最佳实践
3. **存储位置** — `[本地运行时路径已隐藏]<domain>/<skill-name>.md`
4. **激活方式** — 下次同类任务触发时，记忆系统检索并自动注入上下文

关键约束：Skill 生成是**轻量的**，不修改模型权重，但需要记忆系统有足够的索引质量才能可靠触发。

### 外部路径：RL 微调

当 Skill 积累足够多高质量样本后，可以触发 RL 微调：

1. **数据收集** — 从成功执行的 Skill 中提取 (task, outcome) 对
2. **微调目标** — 固化 Skill 中的决策模式到模型权重
3. **效果** — 比 Skill 即时生成更持久，但成本高、周期长

与 [[concepts/managed-agents-architecture|Managed Agents 架构]] 的对比：Managed Agents 通常只有外部路径（集中微调），Hermes 的双路径设计让进化更灵活。

## Cron 调度系统

Hermes 内置 Cron 调度器，无需外部 cron 配置即可执行定时任务：

```yaml
# [本地运行时路径已隐藏]
name: 每日知识库报告
schedule: "0 9 * * *"        # 每天 9:00
prompt: |
  总结过去24小时内 wiki 的重要更新，
  包括新 ingest 的文章、概念扩展、
  以及 any 知识缺口需要补充。
timezone: Asia/Shanghai
enabled: true
```

### Cron vs 外部 cron

| 维度 | Hermes Cron | 外部 cron + 脚本 |
|------|------------|----------------|
| 配置方式 | YAML 声明 | crontab + shell |
| 上下文 | 完整 Agent 上下文 | 无 |
| 记忆访问 | 可以查询历史记忆 | 需自己实现 |
| 错误恢复 | Agent 自动重试 | 需自己实现 |
| 多任务编排 | 支持依赖链 | 需自己实现 |

Hermes Cron 的本质：**定时触发一个拥有完整记忆和工具的 Agent 实例**。

## 消息网关：多平台接入

Hermes 通过消息网关支持多平台接入：

```
用户消息 (Telegram/Discord/Slack/飞书)
    ↓
Message Gateway (协议适配)
    ↓
Session Router (路由 + 隔离)
    ↓
Agent Core (记忆 + Skill + 工具)
    ↓
Response
    ↓
Message Gateway (协议适配)
    ↓
平台回复
```

### 平台隔离策略

不同消息平台使用独立的 Session，避免跨平台上下文污染：

- **Telegram session** — 私聊 + 群组 @ 触发
- **Discord session** — DM + slash command
- **飞书 session** — 私聊 + 机器人 @

每个平台的 Session 配置独立，包括：触发关键词、权限控制、上下文保留长度。

### 与飞书的集成

飞书是 Hermes 的重要接入渠道：

1. **个人助手模式** — 私聊机器人，执行任务、查询知识
2. **群组模式** — @ 触发，支持多轮对话
3. **应用模式** — 作为飞书小程序接入，支持富媒体交互

飞书消息的 Session 路由：飞书给每条消息分配 `open_id`，Hermes 用 `open_id` 作为 Session 的唯一标识，实现用户隔离。

## 故障恢复与状态持久化

Hermes 的状态持久化设计确保了故障恢复能力：

### 三层持久化

| 层级 | 存储位置 | 触发时机 | 用途 |
|------|---------|---------|------|
| 实时状态 | `state.json` | 每个工具调用后 | 故障快速恢复 |
| 会话状态 | `session/*.json` | 对话结束后 | 跨会话上下文 |
| 全局记忆 | 向量数据库 + 文件 | 记忆更新时 | 长期知识积累 |

### 故障恢复流程

```
Agent 进程崩溃
    ↓
重启后检测到 state.json
    ↓
从最近状态快照恢复上下文
    ↓
向用户发送"上次任务中断，是否继续？"提示
    ↓
用户确认后从断点继续
```

这与 [[entities/harness不是目的知识才是护城河-一个ai工程交付团队的知识沉淀实践|知识护城河]] 中的"断点恢复"设计一致：每个阶段出口都有明确的持久化产物，支持从任意断点恢复。

## 与 OpenClaw 的关系

Hermes 官方支持从 OpenClaw（已归档）无缝迁移，设计上有明显传承：

- **工具系统**：Hermes 的工具注册机制与 OpenClaw 兼容
- **Skill 格式**：Hermes Skill 文件格式兼容 OpenClaw 的 Skill 文件
- **进程模型**：都采用常驻进程，但在记忆系统上做了显著升级

关键差异：

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| 记忆系统 | 文件系统（朴素） | 向量检索 + 层级衰减 |
| 自进化 | 单一 Skill 路径 | Skill + RL 双路径 |
| 消息网关 | 无 | 多平台内置 |
| Cron | 无 | 内置调度器 |

许多 OpenClaw 用户迁移到 Hermes，延续"养虾"（养 OpenClaw）的习惯。

## 在 Agent 生态中的位置

Hermes 既不是纯工具框架（OpenClaw）也不是纯托管平台（AgentCore），而是介于两者之间：

| 维度 | OpenClaw | Hermes | AgentCore |
|------|----------|--------|-----------|
| 定位 | 工具框架 | **持久化个人 Agent** | 托管平台 |
| 进程 | 常驻 | 常驻 | 托管/共享 |
| 自进化 | Skill only | Skill + RL 双路径 | 集中微调 |
| 多租户 | 无 | 单用户 | 多用户/团队 |
| 生态 | 开源 | 开源 + 商业服务 | 商业化 |

核心差异：Hermes 是**个人超级助手**，围绕单用户的知识积累和任务自动化；AgentCore 是**团队/企业平台**，强调审计、合规、多租户隔离。

这也呼应了 [[concepts/harness-engineering-framework|Harness Engineering]] 的核心洞察：**Agent 的价值来自长时间运行积累的上下文**，Hermes 的持久化架构是实现这一价值的基础设施。

## 关键配置参数

| 参数 | 位置 | 说明 |
|------|------|------|
| cron 调度 | `[本地运行时路径已隐藏]` | YAML 配置文件 |
| 记忆策略 | `[本地运行时路径已隐藏]` | 向量库 + 衰减参数 |
| Skill 目录 | `[本地运行时路径已隐藏]` | 动态生成的 Skill 文件 |
| 消息通道 | `[本地运行时路径已隐藏]` | Telegram/Discord/飞书等 |
| Session 存储 | `[本地运行时路径已隐藏]` | 每个 Session 的上下文快照 |
| 状态持久化 | `[本地运行时路径已隐藏]` | 实时状态和断点恢复 |

## 七、关联查询

- [[queries/hermes-agent-vs-openclaw-claude-code-core-differences-and-use-cases|Hermes Agent 与 OpenClaw/Claude Code 核心差异和适用场景]] — 三框架对比与选型决策

## 关联实体

**上游依赖**:
- [[entities/hermes-agent]] — 提供基础理论/方法

**下游应用**:
- [[entities/cli-anything]] — 具体应用场景
- [[entities/hermes-agent]] — 具体应用场景

**平行协作**:
- [[entities/harness不是目的知识才是护城河-一个ai工程交付团队的知识沉淀实践]] — 替代/补充方案
- [[entities/harness不是目的知识才是护城河-一个ai工程交付团队的知识沉淀实践]] — 替代/补充方案

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
