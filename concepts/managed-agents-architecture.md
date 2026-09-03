---
title: Anthropic Managed Agents 架构：脑手分离设计
created: 2026-05-07
updated: 2026-08-29
type: concept
tags: [anthropic, managed-agents, agent-architecture, session-harness-sandbox, k8s-analogy, cattle-not-pet, prompt-injection, security, context-management, outcomes-loop, evaluation-harness, brain-hand-separation]
related:
  - [[entities/anthropic-claude-managed-agents-platform-2026|Anthropic Claude Managed Agents 平台正式发布]]
  - [[entities/claude-managed-agents-developer-guide|Claude Managed Agents 开发者指南]]
  - [[entities/multica-managed-agents-platform|Multica — 开源 Managed Agents 平台]]
description: Anthropic Managed Agents 将 Agent 组件虚拟化为 Session/Harness/Sandbox 三大接口，K8s 思路解耦大脑与手，实现真正的 Cattle-not-Pet 架构。
---

# Anthropic Managed Agents 架构：脑手分离设计

## 什么是 Managed Agents（Anthropic 的托管 Agent 产品）

Managed Agents 是 Anthropic 在 2026 年正式发布的**托管式 Agent 运行平台**，核心目标是将 Agent 执行所需的全部基础设施——工具调用循环、上下文管理、代码执行环境、错误恢复——全部收归平台，开发者只需定义"做什么"（Agent 角色和任务）。^[raw/articles/anthropic-claude-managed-agents-platform-launch.md]

这是 Anthropic 第一次明确将自己定位为 **Agent 基础设施运营商**，而非单纯的模型供应商。传统 Messages API 要求开发者自己实现 ReAct 循环、工具调用、上下文窗口管理和沙箱隔离，而 Managed Agents 则提供预建好的 Agent 运行框架，适合**长时间异步任务**和多步骤复杂任务的自动化执行。^[raw/articles/claude-managed-agents-developer-guide.md]

### 四大核心概念（官方定义）

| 概念 | 含义 |
|------|------|
| **Agent** | 定义的角色——用哪个模型、系统提示、能用哪些工具和 MCP 服务器 |
| **Environment** | 运行容器——预装 Python/Node.js/Go，配置好网络访问规则 |
| **Session** | 一次具体的任务执行实例 |
| **Events** | 应用与 Agent 之间互发的消息流（ SSE 流式通信） |

### 产品定位光谱

| 方法 | 你管什么 | 适合场景 |
|------|---------|---------|
| Messages API | 全部（循环、工具、容器） | 需要完全自定义 |
| Agent SDK | 工具执行、容器 | 想用工具但自己托管 |
| **Managed Agents** | **只管提示词和任务** | **后端自动化** |
| Claude Code CLI | 基本不用管 | 本地交互式开发 |
| Claude Cowork | 不用管 | 非技术用户 |

这一定位类比 **AWS 从 EC2 到 Lambda 的演进**：EC2 需要自己管理服务器，而 Lambda 只需上传函数代码。Managed Agents 正是 AI 时代的"无服务器化"——开发者只管业务逻辑，平台负责运行时的所有复杂性。^[raw/articles/claude-managed-agents-developer-guide.md]

## 核心定位：K8s 思路虚拟化 Agent 组件

Anthropic 官方 Managed Agents 的核心理念：**用 K8s 的思路虚拟化 Agent 组件**——将 session（事件日志）、harness（大脑/工具调用循环）、sandbox（代码执行环境）解耦为三个独立接口，让每个组件都能独立故障、独立替换、独立扩展。^[raw/articles/anthropic-claude-managed-agents-platform-launch.md]

### K8s 历史映射

| 问题领域 | K8s 解法 | Managed Agents 对应 |
|----------|---------|-------------------|
| 硬件耦合 | Pod/Service/PersistentVolume 抽象 | Session/Harness/Sandbox 接口抽象 |
| 状态丢失 | PersistentVolume 持久化 | Session 日志外部化 |
| 扩展困难 | 无状态 Pod 可任意副本 | Sandbox `provision()` 按需创建 |
| 组件耦合 | 接口解耦，组件独立替换 | 脑手分离 |

## 三大接口抽象

### 1. Session — 记忆接口

```
职责：记录完整事件日志（发生了什么）
写入：emitEvent(id, event)
读取：getSession(id)
特性：外部持久化，不受上下文窗口限制
```

Session 是外部化的持久化事件日志，记录了 Agent 执行过程中的所有事件。这与 K8s 的 PersistentVolume 类似——即使容器（Sandbox）被销毁重建，状态也不会丢失。^[raw/articles/anthropic-claude-managed-agents-platform-launch.md]

### 2. Harness — 大脑接口

```
职责：调用 Claude + 路由工具调用循环
写入：emitEvent(id, event) 写 Session
调用：execute(name, input) → Sandbox
故障：wake(sessionId) 拉起新实例，从 last event 继续
```

**设计原则**：
- Harness **不住在容器里**——独立部署
- Harness **本身无状态**（Cattle）
- 崩溃不丢数据，Session 是真相来源

Harness 是整个架构的"大脑"，负责与 Claude 模型交互、路由工具调用、管理执行循环。它的无状态设计确保了横向扩展能力和故障恢复能力。^[raw/articles/claude-managed-agents-developer-guide.md]

### 3. Sandbox — 手接口

```
职责：Claude 跑代码、编辑文件的执行环境
调用：execute(name, input) → string
故障：provision({resources}) 重建
处理：将失败当工具调用错误传回 Harness
```

Sandbox 是实际执行代码和文件的"手"。当 Sandbox 故障时，Harness 会像工具调用错误一样处理——由 Claude 决定是否重试，而非人工干预。^[raw/articles/anthropic-claude-managed-agents-platform-launch.md]

## 脑手分离架构

```
         Harness（大脑）
              ↓ execute()
         Sandbox（手）
              ↓ emitEvent()
         Session（记忆）
```

**关键原则**：
- **Sandbox 是 Cattle**：挂了就杀掉重建
- **Harness 是 Cattle**：无持久状态，session 在外部
- Claude 决定重试（而非人工抢救）
- Token 初始化时写入 sandbox，之后全程不可达

**脑手分离的价值**：
1. **独立扩展**：大脑（Harness）可以用更强模型，手（Sandbox）可以按需扩容
2. **故障隔离**：手坏了换一只，不影响大脑记忆
3. **异构执行**：不同任务的 Sandbox 可以有不同的运行环境配置
4. **资源优化**：大脑保持轻量，手可以按任务类型动态配置资源^[raw/articles/claude-managed-agents-developer-guide.md]

## 与自托管 Agent 系统的本质区别

| 维度 | Managed Agents（托管） | 自托管 Agent 系统 |
|------|----------------------|------------------|
| **上下文管理** | 平台接管，自动处理窗口耗尽 | 开发者自己实现 context reset/compaction |
| **工具执行** | 平台提供标准工具集 + MCP | 自己实现工具链、权限隔离 |
| **故障恢复** | Session 外部化，从 last event 继续 | 自己实现 checkpoint 和恢复机制 |
| **多 Agent 协作** | 协调器+工作者模式（研究预览） | 自由实现各种编排模式 |
| **输出质量** | Outcomes Loop 自动评估修正 | 依赖 prompt 工程或人工审核 |
| **运维负担** | 接近零运维 | 全套基础设施自己维护 |
| **成本结构** | Token 费 + Session 运行时费（$0.08/小时） | 基础设施成本 + 人力维护成本 |
| **适用场景** | 后端自动化、长时异步任务 | 需要完全自定义控制的场景 |

自托管系统（如基于 Messages API 搭建的 Agent）的核心问题是**基础设施代码与业务逻辑混杂**。当工具调用逻辑超过 500 行且跨项目复用时，Managed Agents 的优势就非常明显——平台吸收了所有这些复杂性。^[raw/articles/claude-managed-agents-developer-guide.md]

## Evaluation Harness：Outcomes Loop 质量保证机制

Managed Agents 引入的 **Outcomes Loop** 是整个平台最具护城河潜力的功能，解决了一个根本问题：**Agent 怎么知道任务做好了？** ^[raw/articles/anthropic-claude-managed-agents-platform-launch.md]

### 工作原理

1. **用户定义 Rubric 评分标准**：具体可量化的验收条件（如：收入预测需用5年历史数据、增长率假设需明确说明、产出必须是 xlsx 格式且包含特定sheet）
2. **Agent 完成后，召唤独立评分者 Agent 打分**：执行者与评分者用**独立上下文窗口隔离**，防止自评自夸
3. **评估结果三种状态**：
   - `satisfied`（达标，结束）
   - `needs_revision`（需修改，继续迭代）
   - `max_iterations_reached`（达最大迭代次数，默认3次，最多可设20次）

### 设计洞察

Outcomes Loop 本质是**"计划-执行-评估-修正"闭环**的工程化实现。传统 AI 工作流依赖 Prompt 工程来保证输出质量，结果不可预测且难以量化。而 Outcomes Loop 将输出质量的评估从主观变成可配置、可复现的自动化流程。^[raw/articles/anthropic-claude-managed-agents-platform-launch.md]

评分者与执行者之间上下文窗口的**刻意隔离**，是防止"自己评自己"的关键设计。这种工程细节处理体现了 Anthropic 对实际应用场景的深度理解——如果允许 Agent 自己评估自己的输出，质量保证机制就会形同虚设。

### 与传统评测框架的对比

| 维度 | Outcomes Loop | 传统 Agent 评测框架 |
|------|--------------|-------------------|
| 触发时机 | 每次任务完成后自动触发 | 通常是离线批量评测 |
| 评估者 | 独立 Agent（隔离上下文） | 可能是同一模型自评 |
| 修正机制 | 自动迭代修正 | 通常只评不改 |
| 评分标准 | 用户自定义 Rubric | 固定评测指标 |
| 适用场景 | 生产环境质量保障 | 研发阶段能力评估 |

Outcomes Loop 面向**生产环境**的质量保障，而传统评测框架（如 AgentEval）更多用于**研发阶段**的能力评估和回归测试。^[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation.md]

## 操作原则：Cattle-not-Pet 的工程实践

### 核心操作原则

| 原则 | 说明 |
|------|------|
| **Cattle not Pet** | 所有组件（Sandbox、Harness）都应能独立挂掉重建 |
| **脑手分离** | Claude+Harness（大脑）与 Sandbox（手）分离，各自独立扩展 |
| **Session 持久化** | `emitEvent` → `getSession` → 从 last event 继续 |
| **Token 不可达** | 架构性安全 > 权限收窄 |
| **Session ≠ Context** | Session 是外部持久化日志，不受上下文窗口限制 |
| **故障当错误** | Sandbox 失败被当作工具调用错误传回 Harness |

### 幂等性设计

在自托管系统中，故障恢复往往需要复杂的幂等性设计。而在 Managed Agents 中，由于每个组件都是 Cattle，任何组件的故障都不会影响整体任务的正确性——只需要从 Session 的 last event 继续即可。^[raw/articles/claude-managed-agents-developer-guide.md]

### 成本优化策略

- **短时任务**（秒级）：实际成本低于预期
- **长时任务**：精确到毫秒计费，需关注累积成本
- **提示词缓存**：命中缓存享 0.1x 折扣，system prompt 较长的场景受益明显
- **Environment 复用**：同类任务的多个 Session 应尽可能复用同一 Environment，利用平台缓存加速启动

## 安全架构：Token 结构性隔离

**设计原则：让 token 在 sandbox 里根本不可达**

```
OAuth Token → 安全保险箱 → MCP 代理 → 专用 MCP 工具
                                    ↑
                              Claude（通过代理调用）
```

**对比传统方式**：
- 传统：收窄 token 权限（假设 Claude 拿着受限 token 干不了什么）
- Managed Agents：结构性消除（Claude 根本碰不到 token）

架构性安全 > 权限收窄。当 token 物理上不可达时，即使 Claude 被 prompt injection 攻击，攻击者也无法获取 token 权限。^[raw/articles/anthropic-claude-managed-agents-platform-launch.md]

## 平台生态与关联产品

### 关键关联实体

- [[entities/anthropic-claude-managed-agents-platform-2026|Anthropic Claude Managed Agents 平台正式发布]] — 平台发布公告，四大新功能详解
- [[entities/claude-managed-agents-developer-guide|Claude Managed Agents 开发者指南]] — API 详解、产品定位、定价结构
- [[entities/multica-managed-agents-platform|Multica — 开源 Managed Agents 平台]] — 开源实现对比
- [[entities/harness-generator-evaluator-anthropic|Claude Harness 设计：Generator-Evaluator 架构]] — Generator-Evaluator 双代理结构解决自我评估偏差
- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation-framework|AgentEval：YAML驱动的Agent评测框架]] — 传统评测框架，与 Outcomes Loop 对照
- [[entities/anthropic-long-running-agent-adversarial-architecture|Anthropic 长时运行 Agent 架构]] — 对抗式设计 + Outcomes Loop 实践
- [[concepts/harness-engineering-framework|Harness Engineering 框架]] — Managed Agents 的 Harness 工程背景
- [[concepts/openclaw-architecture|OpenClaw 架构]] — 对比：Harness-centric 架构
- [[concepts/hermes-agent|Hermes Agent]] — 对比：内置闭环学习机制
- [[entities/cloudflare-glasswing-mythos-security|Project Glasswing]] — Token 隔离的安全实践参照

### 与 OpenClaw/Hermes 对比

| | Managed Agents | OpenClaw | Hermes Agent |
|--|---------------|----------|--------------|
| 架构 | Session/Harness/Sandbox 接口抽象 | Harness-centric | 内置闭环学习 |
| 持久化 | Session 外部日志 | 工具链输出 | Memory/Skill 写回 |
| 扩展性 | 组件独立替换 | 较难拆分 | 较难拆分 |
| 安全 | Token 不可达设计 | 工具链隔离 | 用户档案隔离 |
| 质量保障 | Outcomes Loop | 无内置机制 | 闭环学习 |

## 关键要点

1. **Managed Agents 是 Anthropic 的托管 Agent 平台**，而非开源框架或自研系统。它代表了从"调用模型"到"委托任务"的范式转变
2. **脑手分离**是架构核心：Harness（大脑）负责决策，Sandbox（手）负责执行，Session（记忆）负责持久化
3. **Cattle-not-Pet** 原则：所有组件都应能独立故障重建，Claude 决定重试而非人工干预
4. **Outcomes Loop** 解决了 Agent 质量保障问题：通过独立评分者 + Rubric 标准实现自动化的"计划-执行-评估-修正"闭环
5. **Token 结构性隔离**是安全基础：架构性安全优于权限收窄
6. **与传统自托管系统的本质区别**：平台吸收了所有基础设施复杂性，开发者只管业务逻辑

---

## 关联实体

**上游依赖**:
- [[entities/anthropic-claude-managed-agents-platform-2026]] — 提供基础理论/方法
- [[entities/claude-managed-agents-developer-guide]] — 提供基础理论/方法
- [[entities/multica-managed-agents-platform]] — 提供基础理论/方法

**下游应用**:
- [[entities/claude-managed-agents-developer-guide]] — 具体应用场景
- [[entities/multica-managed-agents-platform]] — 具体应用场景
- [[entities/harness-generator-evaluator-anthropic]] — 具体应用场景

**平行协作**:
- [[entities/anthropic-long-running-agent-adversarial-architecture]] — 替代/补充方案
- [[entities/cloudflare-glasswing-mythos-security]] — 替代/补充方案
- [[entities/5238111]] — 替代/补充方案


→ [[raw/articles/anthropic-claude-managed-agents-platform-launch.md|原文存档]]

## 新增关联实体
- [[entities/5238111]]
- [[entities/langchain-harrison-chase-sandbox-architecture]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
