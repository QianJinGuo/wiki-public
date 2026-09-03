---
title: Agent Memory System Design 的核心设计原则？
created: 2026-04-30
updated: 2026-05-21
type: query
tags: [agent, memory, architecture, research]
sources: ['entities/agent-memory-architecture', 'entities/agent-memory-modular-framework', 'entities/hermes-agent-memory-system', 'entities/agent-harness-context-management-working-set']
confidence: high
---
# Agent Memory System Design 的核心设计原则？

## 研究问题

设计或评审一个 Agent Memory System 时，应遵循哪些核心设计原则？如何构建一个治理驱动的闭环记忆系统，决定哪些历史信息跨时影响未来决策？

## 核心判断准则

> **Memory 不是"聊天记录存储"，而是一个治理驱动的闭环系统**

关键问题：哪些历史信息被允许跨时影响未来决策？这些信息何时应被修正、降权或遗忘？

## 记忆层次模型

| 层次 | 设计问题 | 推荐载体 |
|------|----------|----------|
| **工作记忆** | 当前任务需要哪些即时上下文 | Context window / working set |
| **情景记忆** | 完整轨迹如何可回放 | JSONL session log / raw traces |
| **语义记忆** | 哪些信念可跨任务复用 | MEMORY.md / SQLite / vector+keyword index |
| **程序性记忆** | 哪些做法应沉淀成操作资产 | Skills / playbooks / validators |

## 四类建模对象及常见失败

| 对象 | 应记录什么 | 常见失败 |
|------|------------|----------|
| **用户模型** | 偏好、风险边界、沟通习惯、长期目标 | 把一次性口味误写成永久偏好 |
| **任务模型** | 已确认结论、被否决方案、当前真版本、未完成承诺 | 把过期 TODO 当作仍有效事实 |
| **世界模型** | 环境约束、系统边界、组织规则、数据新鲜度 | 不记录来源与有效期 |
| **自我模型** | 工具稳定性、失败路径、验证经验 | 只记成功，不记失败条件 |

## 写入-管理-读取原则

### 写入原则
> **写入是预算分配，不是"有价值就存"**

高价值信号来源：
- 冲突、反复出现的行为证据
- 影响未来决策的约束
- 可复用的失败教训

### 管理原则
管理层负责：
- 整合、冲突处理
- 衰减、来源追踪
- 用户可控删除

> **旧 belief 被新证据反复否定时，应降权或归档，而不是继续静默影响检索**

### 读取原则
> **读取应由任务约束驱动**

```text
retrieve(query) -> read(task_context, belief_graph, provenance, freshness)
```

语义相似度只是入口，真正的读取函数应结合：
- task context
- belief graph
- 来源可信度
- 时间权重

## 设计检查表

- [ ] 每条记忆是否有类型、来源、作用域、置信度和时间权重？
- [ ] Memory、State、Policy 是否分离，避免记忆动态改写安全策略？
- [ ] 是否同时支持 keyword、vector、graph 或结构化字段检索？
- [ ] 是否有冲突表达机制，而不是简单覆盖旧结论？
- [ ] 是否能从失败回溯到"写错、管错、读错、用错"中的哪一环？
- [ ] 是否有用户查看、编辑、删除记忆的路径？
- [ ] 是否能把高频成功流程沉淀为 Skill 或 playbook？

## 何时不要写入

- 只在单个 session 内有用的临时状态
- 低置信、不可验证、无来源的推断
- 已被更高层抽象吸收的重复 event
- 违反用户隐私边界或无法解释用途的内容

## 相关资源

- [[entities/agent-memory-architecture|Agent Memory 架构本质]]
- [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- [[comparisons/skill-system-design-comparison|Skill 系统设计对比]]
