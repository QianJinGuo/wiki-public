---
title: Harness Engineering 的核心原则与最佳实践？
created: 2026-05-15
updated: 2026-05-21
type: query
tags: [topic-map, navigation, agent-harness-engineering, research]
---
# Harness Engineering 的核心原则与最佳实践？

## 研究问题

Harness Engineering 作为让 AI 从"聪明"到"可靠"的第三代工程范式，其核心设计原则是什么？有哪些最佳实践可以指导 Agent 可靠性工程的落地？

## 核心定义

**Harness Engineering**：AI Agent 的可靠性工程范式，通过结构化的约束层（Prompt/Context/Harness）让不确定的 LLM 输出变得可预测、可控制。

核心理念：
- **Thin Harness, Fat Skills**：轻量级 Harness + 丰富 Skills 实现系统灵活性
- **Build to Delete**：组件应有明确的保质期，不合适就删除而非无限扩展
- **Memory as Governance**：记忆系统是治理驱动的闭环，而非简单的存储

## 核心设计原则

### 1. 分离关注点

| 组件 | 职责 | 设计要点 |
|------|------|----------|
| **Prompt** | 指令定义、任务分解 | 清晰、无歧义、可验证 |
| **Context** | 状态传递、信息供给 | 高效压缩、关键信息突出 |
| **Harness** | 可靠性保障、约束执行 | 最小化干预、最大透明度 |

### 2. 可观测性优先

- 每次修改应有明确的预期行为变化
- 支持回滚和 A/B 测试
- 监控 harness 干预频率和效果

### 3. 组件保质期意识

- 定期评估组件必要性
- 建立"删除候选"机制
- 避免 harness 层的累积复杂性

### 4. 长程任务支撑

- 状态机设计（Plan-Execute, ReAct 等）
- 中断恢复机制
- 进度检查点和回退策略

## 最佳实践清单

### 设计阶段
- [ ] 明确任务类型（短任务 vs 长程任务）
- [ ] 评估 Sub-Agent vs Agent Team 适用场景
- [ ] 设计状态持久化策略

### 实现阶段
- [ ] 遵循最小干预原则
- [ ] 实现完整的 Context Reset 机制
- [ ] 建立工具调用的超时和重试策略

### 验证阶段
- [ ] 每次只移除一个组件进行验证
- [ ] 设置 WIP（Work In Progress）限制
- [ ] 记录干预率和成功率

## 七层框架（参考）

1. **任务理解层**：意图识别、任务分解
2. **上下文管理层**：信息压缩、状态维护
3. **工具选择层**：MCP、API、CLI 路由
4. **执行监控层**：进度跟踪、异常捕获
5. **记忆管理层**：短期/长期记忆分层
6. **安全治理层**：权限控制、审计日志
7. **自进化层**：经验沉淀、策略更新

## 相关概念

- [[concepts/agent-backend-unification|Agent 与后端统一架构]]
- [[concepts/agent-memory-lifecycle-philosophies|Agent Memory 生命周期哲学]]
- [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- [[concepts/coding-harness-engineering|Coding Harness 工程本质]]
- [[concepts/sdd-specification-driven-development-harness|规格驱动开发与 Harness]]

## 相关实体

