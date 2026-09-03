---
title: Wiki Topic Map 的结构与导航最佳实践？
created: 2026-05-21
updated: 2026-06-11
type: moc
tags: [meta, navigation, knowledge-management]
sources:
  - wiki/queries/topic-map
confidence: high
---
# Wiki Topic Map 的结构与导航最佳实践？
## 核心问题
如何通过 Topic Map 将"收藏夹式列表"转化为"专题路径"，实现知识库的结构化导航？

## Topic Map 分层结构
### 第一层：主题分组（Navigation Layer）
- **目标**：从每组的第一批页面读起，再进入相关 raw source
- **分组依据**：按能力维度划分（Agent Harness、Memory、Skill、Tools、Models 等）

### 第二层：页面类型
| 类型 | 目录 | 说明 |
|------|------|------|
| Entity | `entities/` | 工具、产品、人物 |
| Concept | `concepts/` | 框架、思想、方法 |
| Comparison | `comparisons/` | 横向对比 |
| Query | `queries/` | 导航、仪表盘、专题图 |
| Raw | `raw/articles/` | 原始来源存档 |

## 导航最佳实践
### 1. 路径设计原则
- **纵向深挖**：从 Topic Map → Entity/Concept → Raw Source
- **横向关联**：通过 wikilink 跨目录连接，如 `[[entities/agent-harness-context-management-working-set|Context 作为 working set`
- **最小跳转**：每个 Topic Map 页面至少包含 2 个跨目录链接

### 2. 分组策略
```
Agent Harness 与工程架构
  ↓
Memory 与 Context 管理
  ↓
Skill、Sub-Agent 与团队编排
  ↓
Claude Code 与 AI Coding
  ↓
Agent 工具与运行时
  ↓
模型、推理与训练
  ↓
知识库与学习系统
```

### 3. 路径设计示例
- Topic Map 页面（导航层）
- → Entity/Concept 页面（知识层）
- → Raw Source 页面（原始来源）

### 3. 质量维护
- **置信度标注**：`confidence: high/medium/low` 反映内容可靠性
- **标签体系**：每个页面 `tags: []]` 便于聚合检索
- **健康度视图**：低置信、冲突、过期、未评分页面的维护视图

## 相关实体

- [[entities/wiki-evolver|Wiki Evolver
- [[entities/agent-harness-architecture-deep-dive-aksahy|Agent Harness 解析：智能体架构深度拆解

## 相关知识
- [[moc/wiki-master-map|Wiki Topic Map 原始页面]]
- [[entities/agent-harness-context-management-working-set|Context 作为 working set
- [[queries/vault-evolution-dashboard|Vault 演化仪表盘]]
