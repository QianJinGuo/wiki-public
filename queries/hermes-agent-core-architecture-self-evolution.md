---
title: "Hermes Agent 的核心架构设计与自进化机制是什么？"
created: "2026-05-21"
updated: "2026-05-21"
type: query
tags: [hermes-agent, architecture, self-evolution, memory, skill-system]
---

# Hermes Agent 的核心架构设计与自进化机制是什么？

## 核心问题

Hermes Agent 的核心架构是如何设计的？其自进化机制如何驱动 Agent 持续改进？记忆系统和 Skill 管理在其中扮演什么角色？

---

## 一、9 模块系统架构

Hermes Agent 采用 9 模块架构，各模块各司其职：

|| 模块 | 职责 | 关键设计点 |
||------|------|-----------|
| 1 | 任务理解 | 意图识别、任务分解 | 支持多轮对话上下文 |
| 2 | 上下文管理 | 信息压缩、状态维护 | 高效压缩、关键信息突出 |
| 3 | 工具选择 | MCP/API/CLI 路由 | 动态工具发现机制 |
| 4 | 执行监控 | 进度跟踪、异常捕获 | 实时状态反馈 |
| 5 | 短期记忆 | 会话级状态管理 | 零成本压缩 |
| 6 | 长期记忆 | 经验沉淀、知识检索 | 向量检索 + 结构化存储 |
| 7 | Skill 管理 | 工具链编排、生命周期 | 动态加载/卸载 |
| 8 | 安全治理 | 权限控制、审计日志 | 最小权限原则 |
| 9 | 自进化层 | 自我评估、策略更新 | 闭环反馈机制 |

---

## 二、记忆系统架构

### 双层记忆设计

|| 层 | 存储形式 | 生命周期 | 压缩机制 |
||---|---------|---------|---------|---------|
| 短期记忆 | 会话消息 | 当前会话 | 零成本压缩（有内容则用，无则跳） |
| 长期记忆 | 向量数据库 | 跨会话 | 增量更新、定期合并 |

### 记忆治理原则

- **Memory as Governance**：记忆系统是治理驱动的闭环，而非简单存储
- 第 1 层（工具结果磁盘存储）和第 2 层（微压缩）几乎零成本，优先利用
- 做梦机制（四阶段：标定→收集→合并→整理）用于长期知识整理

---

## 三、Skill 系统

### Skill 生命周期

```
加载 → 执行 → 监控 → 卸载
```

### Skill 分类

| 类型 | 示例 | 加载策略 |
|------|------|---------|
| 核心 Skill | 记忆、工具路由 | 常驻内存 |
| 领域 Skill | 代码编写、搜索 | 按需加载 |
| 临时 Skill | 一次性任务 | 用完即删 |

### Skill 管理最佳实践

- Thin Harness, Fat Skills：轻量级 Harness + 丰富 Skills 实现系统灵活性
- Build to Delete：组件有明确保质期，不合适就删除而非无限扩展
- Skill 应该有清晰的 description 作为路由契约

---

## 四、自进化闭环（Self-Improving Loop）

### 四阶段闭环

```
执行 → 评估 → 反思 → 更新
```

1. **执行**：Agent 完成特定任务
2. **评估**：对比预期结果与实际结果
3. **反思**：分析失败原因，更新策略
4. **更新**：修改记忆、调整 Skill 组合、更新 Harness

### winty 版 Self-Improving 闭环详解

- 从真实失败案例中学习，而非从成功案例中学习
- 负面样本（Gotchas）比正面样本更有价值
- 建立"失败模式库"驱动系统进化

---

## 五、可观测性方案

| 维度 | 指标 | 告警阈值 |
|------|------|---------|
| 上下文使用率 | Token 使用率 | > 80% |
| Skill 加载频率 | 每小时加载次数 | 异常突增 |
| 任务完成率 | 成功率 | < 90% |
| 工具调用稳定性 | 工具调用失败率 | > 5% |

---

## 六、Kubernetes 部署与 Worker 运行时

HiClaw v1.1.0 提供了 Hermes Worker 在 K8s 集群的部署方案：

- Worker 作为无状态 Sidecar 运行
- ConfigMap 管理 Harness 配置
- Secret 管理 API 密钥
- HPA 支持自动扩缩容

---

## 七、MemOS 记忆插件

MemOS Hermes 记忆插件实现：

- 结构化记忆的持久化存储
- 跨会话记忆检索
- 记忆过期与自动清理策略

---

## 相关概念

- [[concepts/hermes-agent|Hermes Agent 核心概念]]
- [[concepts/hermes-agent-onboarding|Hermes Agent 上手指南]]
- [[concepts/hermes-agent-skill|Hermes Agent Skill 系统]]
- [[concepts/agent-memory-lifecycle-philosophies|Agent Memory 生命周期哲学]]

## 相关实体

