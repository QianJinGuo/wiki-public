---

title: "存之有序，治之有矩——Agent 记忆系统的工程实践与演进"
created: 2026-06-10
updated: 2026-07-22
tags: [agent, architecture, aws, code, data, database, evaluation, k8s, llm, memory, open-source, prompt, rag, rl]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进
---

# 存之有序，治之有矩——Agent 记忆系统的工程实践与演进

→ [[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进|原文存档]]^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]

## 深度分析

# 存之有序，治之有矩——Agent 记忆系统的工程实践与演进
摘要：本文是”解决 Agentic AI 应用 Token 爆炸问题”系列的第三篇，系统讨论 Agent 记忆系统在生产环境的工程税:从写入纪律、Prompt Cache 冲突、跨模型容量、Embedding 迁移到 Agent 自产 Skill 治理,以及 S3 Files / S3 Vectors / AgentCore Memory 在亚马逊云科技上的落地路径。 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]

### 核心观点

1. **目录** ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
02 2026 的新变量 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
03 Part 1. ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
2. 记忆系统的五个工程考量 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
04 Part 2. ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
3. 记忆的写入纪律与失效机制 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
05 Part 3. ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
4. 记忆写入与 Prompt Cache 的冲突 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
06 Part 4. ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
5. Token ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
07 Part 5. ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]

### 关联实体

- [[entities/headroom-context-compression-agent-vibecoder]]
- [[entities/构建无服务器kiro调度平台用kiro-cli-eventbridge-ecs-fargate实现定时ai任务]]
- [[entities/hermes-agent-v014-architecture-shugex]]
- [[entities/extending-mcp-support-for-amazon-bedrock-agentcore-gateway]]
- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation]]
- [[entities/腾讯混元新里程碑hy3-preview-发布开源agent-表现全面提升.md]]

## 补充：Agent 经验变更系统视角（若飞/架构师 2026-07-19）

若飞从"经验变更系统"视角补充了 Agent Memory 的治理维度：记忆一旦参与决策就不再只是存档，需要管理证据、状态、作用域、验证、晋升和回滚。^[raw/articles/agent-memory-self-improvement-verification-ruofei-2026-07-19.md]

### 六层经验层级

| 层次 | 解决的问题 | 默认状态 |
|------|-----------|----------|
| 原始证据 | 当时发生了什么 | 只追加、可追溯 |
| 当前状态 | 任务现在走到哪里 | 一个权威版本 |
| 候选经验 | 哪条做法可能有用 | 未验证、可过期 |
| 已验证记忆 | 哪类场景可以复用 | 有来源和作用域 |
| Skill / Harness | 以后默认怎样做 | 有版本、可回归 |
| Policy | 哪些动作允许或禁止 | 外部维护、单独审批 |

### 读取链与写入链分离

- **读取链**：当前任务允许哪些过去进来
- **写入链**：当前结果凭什么影响未来

关键洞察：Memory 的写入是经验获得长期影响力的入口，读取则是历史进入当前任务的准入过程。一次有效离可靠经验还很远——需复现+外部证据+明确作用域才能晋升。Memory 进入 Skill 或 Harness（参考信息→默认行为）时，验证要求随之提高。

→ [[raw/articles/agent-memory-self-improvement-verification-ruofei-2026-07-19|原文存档]]

## 实践启示

1. **Agent 设计**: 关注控制流与上下文工程的平衡，Harness 约束比模型能力更影响成功率 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
2. **可观测性**: Agent 行为调试应优先检查工具定义和上下文质量 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
3. **渐进式部署**: 从简单 ReAct 循环起步，逐步引入多 Agent 编排 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]
4. **验证优先**: 建立完善的测试验证体系，确保 Agent 行为可预测 ^[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进.md]

## 相关实体

- [[moc/agent-engineering-guide|MOC]]
