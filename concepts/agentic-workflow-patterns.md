---
title: "Agentic Workflow Patterns"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [workflow, agent, automation, token-efficiency, optimization, patterns]
sources:
  - raw/articles/github-token-efficiency-agentic-workflows
summary: Agent 工作流编排的核心模式，涵盖 ReAct、Plan-Execute、Reflection 等常见范式，以及 Token 效率优化、工作流自动化、CI/CD 集成等工程实践。
---

# Agentic Workflow Patterns

> AI agent 的工作流编排模式。常见模式：ReAct、Plan-Execute、Reflection、Multi-Agent 协作。关注任务分解、状态管理、错误恢复。

## 一、常见工作流模式

### 1.1 ReAct（Reasoning + Acting）

将推理和执行交替进行：
```
Thought → Action → Observation → Thought → Action → ...
```

**核心思想**：让 Agent 在执行动作前先思考，通过观察动作结果来指导下一步行动。

### 1.2 Plan-Execute

分离规划与执行：
```
Planner Agent: 生成执行计划
↓
Executor Agent: 按计划执行
↓
Validator Agent: 验证结果
```

**优点**：规划过程可以独立优化，执行过程可以并行化。

### 1.3 Reflection / Self-Correction

让 Agent 审视自己的输出并自我改进：
```
生成 → 审查 → 改进 → 再审查 → ...
```

### 1.4 Multi-Agent 协作

多个专门化的 Agent 协同工作：
- **Orchestrator**：负责任务分解和协调
- **Specialist Agents**：各自处理子任务
- **Critic**：评估和反馈

## 二、Token 效率优化

### 2.1 效率问题的本质

Agentic workflows 的 token 效率问题不是简单的"用少一点"，而是**结构性问题**。

**GitHub 的经验**：最大效率来源是**消除不必要的 LLM 调用**，而不是减少单次调用的 token 消耗。最便宜的 LLM 调用是根本不做的调用。

### 2.2 三种主要优化模式

#### 模式 1：消除未使用的 MCP 工具注册

LLM API 无状态，所以 agent 运行时通常在每次请求中包含完整 MCP 工具函数名和 JSON schema。

**问题**：对于有 40 个工具的 GitHub MCP server，每个 turn 可能增加 10-15KB schema。如果 agent 只用两个工具，剩下 38 个就是纯开销。

**解决方案**：通过交叉引用工具清单和实际工具调用来识别和消除未使用的工具。

#### 模式 2：将 MCP 替换为 CLI

MCP 工具调用不仅是数据检索，还是一个推理步骤：
- Agent 必须决定调用工具
- 构造参数
- 在上下文中接收输出

这是完整的 LLM API 往返，消耗工具调用 JSON schema、参数块和响应的 token。

**解决方案**：
- `gh pr diff`：确定性 HTTP 请求，没有 LLM 参与
- pre-agentic 数据下载：YAML 中 setup 步骤在 agent 启动前运行 `gh` 命令
- in-agent CLI 代理替换：轻量透明 HTTP 代理路由 CLI 流量

#### 模式 3：区分不同类型的 token 成本

```
ET = m × (1.0 × I + 0.1 × C + 4.0 × O)
```

| 参数 | 含义 | 权重 |
|------|------|------|
| m | 模型成本乘数 | Haiku=0.25×, Sonnet=1.0×, Opus=5.0× |
| I | 新处理输入 token | 1.0× |
| C | 缓存读取 token | 0.1× |
| O | 输出 token | **4.0×** |

> 输出 token 权重 4× 是因为它是所有主要提供商中最贵的 token 类型。

### 2.3 优化飞轮设计

GitHub 构建了一套自动化的优化飞轮：

```
Token 审计器 → 优化器分析 → 生成修复建议 → 人工确认 → 实施 → 结果回流
     ↑                                                            ↓
     └──────────────── 每日报告形成良性循环 ←──────────────────────┘
```

**关键数据**：
- 运行频率高的更重要
- 6.8 次/天 × 62% 节省 > 1 次/天 × 20% 节省

## 三、工作流设计原则

### 3.1 从可观测性开始

在优化 token 效率之前，必须先知道 token 用在哪里。

**关键数据**：
- 每次 API 调用的输入 token
- 输出 token
- 缓存读取 token
- 缓存写入 token
- 模型、提供商、时间戳

### 3.2 先消除不必要的 LLM 调用

最低挂的果实：
- 未使用的 MCP 工具（从 40 个工具 prune 到实际使用的 2 个）
- 总是需要的数据（PR diff、变更文件列表）用 pre-agentic CLI 步骤获取
- 安全敏感的变更过滤在 LLM 之外完成（relevance gate 直接跳过）

### 3.3 用正确的指标

不要只看原始 token 数量。使用 Effective Tokens (ET) 考虑模型成本差异。

### 3.4 区分效率提升和负载变化

当看到 ET 变化时，需要问：
- 是工作流更高效了，还是只是工作负载变轻/变重了？
- 跟踪 LLM 调用次数配合 token 数量
- 恒定的 LLM turns/次 + 下降的 tokens/次 = 真正的效率提升

## 四、常见问题诊断

### 4.1 配置错误的影响

一个配置错误可能导致灾难性的效率问题。

**典型案例**：工作流复制测试文件到 `/tmp/`，然后调用 `gh aw compile *`，但沙箱的 bash allowlist 只允许相对路径 glob 模式。每次编译尝试都被阻止，agent 陷入 64 turn 后备循环手动阅读源码。

**特征**：极高 ET 和明显的循环模式。

### 4.2 测量效率提升的困难

三个混淆因素：
- **不是所有 token 生来平等**：相同 token 数量在不同模型上成本差异很大
- **工作负载本身在变化**：同一个工作流有时处理 5 行修复，有时处理 200 行 PR
- **质量问题最难衡量**：process-level 信号只是近似，不是 outcome 信号

## 五、关联实体

- [[entities/agent-workflows]] — Agent 工作流原文
- [[entities/agent-skills-vs-coze-dify-n8n-lowcode-yexiaocha]] — Agent Skills vs Workflow 低代码平台对比
- [[entities/four-sub-agent-patterns]] — 四种 Sub Agent 模式
- [[concepts/multi-agent-systems]] — 多智能体系统
- [[concepts/harness-engineering-framework]] — Harness 工程框架

## 关联实体

**上游依赖**:
- [[entities/agent-workflows]] — 提供基础理论/方法

**下游应用**:
- [[entities/agent-skills-vs-coze-dify-n8n-lowcode-yexiaocha]] — 具体应用场景

**平行协作**:
- [[entities/four-sub-agent-patterns]] — 替代/补充方案
- [[entities/claude-research-agent-auto-newsletter-cyrilxbt]] — 替代/补充方案


→ [[raw/articles/github-token-efficiency-agentic-workflows|原文存档]]

## 新增关联实体
- [[entities/claude-research-agent-auto-newsletter-cyrilxbt]]

## 所属 MOC

- [[moc/workflow-orchestration|Workflow Orchestration]]
