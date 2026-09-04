---
title: "Intuit EWOK Agent：模型决策/确定性执行分离的灾害恢复 Agent 架构（typed skills + 有界循环）"
created: 2026-09-05
updated: 2026-09-05
type: entity
tags: [agent-architecture, disaster-recovery, decision-execution-separation, typed-skills, mcp, guardrails, bounded-agentic-loop, deterministic-execution, aws, bedrock]
sources: [raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026]
confidence: 0.7
provenance_state: extracted
---

# Intuit EWOK Agent：模型决策/确定性执行分离的灾害恢复 Agent 架构（typed skills + 有界循环）

> Intuit 在跨多 AWS Region 数千微服务的规模上，将灾害恢复（DR）决策从「on-call 工程师的部落知识」转移到 AI Agent：构建 EWOK Agent（Amazon Bedrock）叠加在已有的确定性执行系统 EWOK（Ecosystem Wide Orchestrator Kit）之上。核心设计立场是 **"The model decides what to do, and the EWOK Agent deterministically executes how"**——模型决策与确定性执行分离。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

## 问题：EWOK 解决了执行，没解决决策

Intuit 已有一个中央灾害恢复系统 **EWOK**，用 YAML 声明式定义恢复意图，标准化 compute/数据库/网络/缓存/异步负载的故障切换（failover）执行，把恢复时间从数小时降到约 20 分钟。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md] 但 EWOK 只解决**执行**，不解决**决策**：选择哪个恢复工作流适用、确认资产是否就绪，仍依赖 on-call 工程师的部落知识（tribal knowledge）。EWOK Agent 正是为这个决策缺口而构建。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

**关键可迁移洞见（文章原文）**：下文所述模式（typed skills + 薄 Amazon Bedrock 层 + 确定性执行器之上的有界 agentic 循环）并非 EWOK 特有，可应用于任何暴露了"已认证、可审计 API"的系统。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

## 四层架构

EWOK Agent 自顶向下四层，从工程师的自然语言请求到确定性恢复动作：^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

- **Consumer 层（顶）**：Intuit Engineering Portal + IDE 集成，经 [[concepts/coding-agent-architecture|MCP]] 连接，是工程师提交自然语言请求的两个入口。
- **Agent 层**：运行于 Amazon Bedrock，将基础模型选择 + 有界推理循环 与 每次调用都应用的 Amazon Bedrock Guardrails 配对；负责模型选择、guardrails、skill 分发。
- **Skill 层（右）**：持有**类型化、带版本的 skills**（typed, versioned skills），每个定义为 YAML schema + prompt body，编译成模型可选的工具规格。
- **Execution 层（底）**：EWOK API 层，确定性地执行资产解析、恢复工作流查找、就绪检查、策略门（policy gates）、基于 execution-ID 的追踪、变更记录，以及故障切换本身——通过 compute/database/cache/traffic 各自的工作负载专用 agent 完成。

状态沿同一条路径向上流回（execution → agent → consumer）。

## 核心设计原则：决策/执行分离

文章反复强调一条贯穿全部设计选择的原则：**模型决定做什么，EWOK Agent 确定性执行怎么做**（The model decides what to do, the EWOK Agent deterministically executes how）。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

- **决策/执行分离**（decision/execution separation）：模型负责在类型化恢复 skills 中做选择；确定性 EWOK 执行器负责执行选中的工作流。这把模型错误的爆炸半径限定住。
- **类型化、带版本的 skills**：每个 skill = YAML schema + prompt body，编译成工具规格，把部落知识（runbook 查找、哪个工作流适用、资产是否就绪）变成**版本控制、可评审、可测试**的产物。
- **有界推理循环**（bounded reasoning loop）：Agent 在确定性执行器之上运行有界循环，每次调用应用 guardrails，而非不受约束的自主探索。
- **逐任务模型选择**：把基础模型选择与 skill 分发决策配对。

## 工程师视角的故障切换体验

工程师对着 Agent 说"You failover payments-gateway in production"，EWOK Agent 依次：解析资产并发现可用恢复工作流 → 选择合适工作流（或让工程师在多选适用时挑选）→ 验证就绪并检查策略门（如激活中的变更冻结窗口）→ 通过 EWOK 系统触发执行并返回 execution ID + change record → 分阶段监控上报直至完成。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

这些任务过去是 runbook 查找 + 控制台访问的序列，由工程师协调 API 调用；现在改为**监督对话**（supervise conversations）。工程师保留在判断与批准环节，但不再需要当编排器。^[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026.md]

## 与 wiki 已有知识的关联

- 有界循环 + guardrails 逐次应用 → 对照 [[concepts/harness-loop-architecture|Harness 循环架构]]、[[entities/amazon-bedrock-guardrails-code-generation-six-patterns|Bedrock Guardrails 六模式]]
- typed skills 编译成工具规格 → [[concepts/skill-engineering-principles|Skill 工程原则]]
- 心智"模型决策 + 确定性执行" → [[entities/agent-architecture-harness-new-backend|Harness 成为 Agent 新后端]]

→ [[raw/articles/intuit-ewok-agent-disaster-recovery-deterministic-execution-2026|原文存档]]