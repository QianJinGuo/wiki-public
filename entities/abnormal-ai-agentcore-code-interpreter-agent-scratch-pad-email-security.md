---
title: "Abnormal AI：Agent 计算暂存区与零信任沙箱的生产实践"
created: 2026-09-15
updated: 2026-09-15
type: entity
tags: [agent, harness, sandbox, code-interpreter, bedrock-agentcore, production-agents, zero-trust, checkpointing, tiered-escalation, security, aws]
sources: [raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security]
confidence: 0.7
provenance_state: extracted
---

# Abnormal AI：Agent 计算暂存区与零信任沙箱的生产实践

> 来源：AWS Machine Learning Blog（AWS China ML feed），2026-09-14 发布。Abnormal AI 行为安全服务（覆盖 25%+ Fortune 500）在生产环境用 Amazon Bedrock AgentCore Code Interpreter 支撑实时内联邮件威胁检测，本文由 AWS 架构师与 Abnormal AI 的 AI 战略 VP 合著。

## 核心命题：Agent 需要一个计算暂存区（scratch pad）

在每天数十亿次操作的规模上，一个反复出现的架构模式浮出水面：Agent 需要一块计算暂存区——不仅用于编码任务，也用于数据聚合、分析、校验，以及任何"语义推理不够用"的工作流。^[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security.md]

Abnormal AI 用 AgentCore Code Interpreter 支撑其实时内联邮件威胁检测的 Agent：今天他们 80% 的代码变更由 Agent 以某种方式参与、40% 由后台 Agent 端到端完成。这说明该模式不是 demo，而是与"AI-native 开发方式"一体的生产运行时选择。^[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security.md]

## 三层检测管道：把 Agent 算力留给最难的样本

Abnormal 的检测管道按难度分级，每一层只处理上一层"没把握"的样本：Tier 1 用小模型、启发式规则与轻量分类器（logistic regression）处理最大流量（十亿/天量级），在此规模下跑更大的模型既贵也无必要；Tier 2 用深度学习/ML 模型做行为信号分析（百万/天）；Tier 3 才交给内联 Agent + Code Interpreter（数万/天）——这些是原本需要人类分析师介入的最难样本，Agent 收到威胁情报后在沙箱里动态写脚本分析，再结合整体行为模型做判定。^[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security.md]

除实时管道外还有一个批处理 analyst agent：摄入误分类与调参信号、跨大消息集找模式、自动撰写候选启发式规则供 Tier 1 使用并改进 Tier 2 模型，约 100 个批任务/周。这类任务可持续 30 分钟以上，甚至跨越一整天——例如跑一段 Code Interpreter 会话、在外部训练模型、再重新调用 Code Interpreter 处理结果。^[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security.md]

## 零信任沙箱：为什么选择 no-egress

Abnormal 选择无外部网络的沙箱配置，理由只有两条但都很硬：其一是可复现性——沙箱无外网访问，外部任何东西都无法影响该会话中 Agent 的行为，环境被设计成完全确定性的；其二是防数据外泄——威胁情报数据要进入沙箱分析，即使 Agent 因 prompt injection 或随机行为变坏，也设计成无法把数据外传。^[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security.md]

配套的三项安全实践：受控数据摄入（刻意决定哪些数据进入 Code Interpreter、允许哪些写操作）；沙箱隔离叠加在既有的网络隔离 harness 之上形成纵深防御；以及让 Code Interpreter 运行在既有 AWS 子处理者关系下以降低合规开销。^[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security.md]

## 四条工程经验（可迁移）

1. **给 Agent 它想要的东西**——Agent 在轻量、通用的 harness 下表现更好，而不是刚性的 step-by-step 工作流；提供高层原则让它自己判断方法。
2. **每个 Agent 都需要暂存区**——Code Interpreter 不只是编码 Agent 的工具，安全 Agent 同样受益于用计算暂存区做数据聚合、模式分析与校验。
3. **用程序化 verifier 作护栏**——当 Agent 拥有单元测试、集成测试、lint 等程序化验证工具时输出质量更高，因为它能在沙箱内自测后再交付结果。
4. **文件系统作为长任务的恢复点**——超过会话时长的操作（如模型训练）用文件做 checkpoint：用 Code Interpreter 做计算、把状态持久化到文件、在外部执行长任务、再重新调用 Code Interpreter 处理结果；文中 analyst agent 的跨天训练即用此模式。

^[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security.md]

## 与 wiki 既有覆盖的关系

- [[concepts/agent-sandbox|Agent 沙箱]] 与 [[entities/agent-harness-architecture-design-production-guide|Agent Harness 生产架构指南]] 给出沙箱与 harness 的通用框架；本篇补的是**生产规模下的三条具体设计约束**：按难度分级把 Agent 算力留给最难样本、无出口沙箱在"确定性 vs 可用性"上的取舍、以及文件系统检查点作为跨会话恢复点。
- [[entities/amazon-bedrock-agentcore-data-persistence-file-system-session-storage-efs-s3|AgentCore 会话文件系统持久化]] 提供文件系统持久化能力，本篇给出"何时该用它做恢复点"的用法。
- [[concepts/verifier-driven-development|Verifier-Driven Development]] 与 [[concepts/agent-orchestration-patterns|Agent 编排模式]] 是四条经验中第 3 条与整体编排的上位概念。
- 平台绑定部分（Code Interpreter 的 TTL 15 分钟～8 小时、MicroVM 会话隔离、AgentCore 产品能力）不可迁移，仅作实现示例；可迁移的是分级调度、沙箱出口策略与检查点模式。

^[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security.md]

## 相关实体

- [[concepts/agent-sandbox|Agent 沙箱与执行容器]]
- [[concepts/verifier-driven-development|Verifier-Driven Development]]
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]
- [[entities/agentcore-managed-harness|AgentCore 托管 Harness]]
- [[entities/amazon-bedrock-agentcore-data-persistence-file-system-session-storage-efs-s3|AgentCore 文件系统持久化]]
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 生产架构设计指南]]
- [[entities/agent-observability-5-layer-architecture|Agent 可观测性 5 层架构]]

→ [[raw/articles/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security|原文存档]]
