---
title: "工程实践积压 — 从理论到可执行产物"
created: 2026-06-10
updated: 2026-06-10
tags: [practice, engineering, backlog, dashboard]
type: query
---

# 工程实践积压 — 从理论到可执行产物

> Wiki 中理论远超实践。此积压将高价值理论转化为 playbook / checklist / template。

---

## P1: Agent Harness 最小检查清单

**来源**: [[concepts/ahe-agentic-harness-engineering]] · 

**产物**: Checklist — 部署任何 Agent 前必须通过的 12 项检查

```markdown
## Harness MVP Checklist v1

### 安全层
- [ ] 工具白名单已定义（agent 只能调用显式授权的工具）
- [ ] 危险操作需人类确认（文件删除、网络请求、数据库写入）
- [ ] 执行超时已设置（单次调用 ≤30s，总任务 ≤1h）

### 上下文层
- [ ] Working set 大小已限制（≤128K tokens）
- [ ] 上下文压缩策略已配置（摘要 vs 裁剪 vs 分区）
- [ ] CLAUDE.md / 系统提示已定义行为边界

### 治理层
- [ ] 成本上限已设置（单任务 $X，日累计 $Y）
- [ ] 审计日志已开启（每次工具调用记录输入/输出/耗时）
- [ ] 错误恢复策略已定义（重试 vs 降级 vs 人工接管）

### 记忆层
- [ ] 短期记忆边界已定义（session scope）
- [ ] 长期记忆存储已选型（文件/向量/键值）
- [ ] 记忆过期策略已配置
```

**优先级**: 🔴 High — 直接影响 Agent 部署安全性

---

## P2: Agent Memory 选型决策树

**来源**: [[concepts/agent-memory-system-design]] · [[concepts/catastrophic-forgetting]]

**产物**: Decision Tree — 根据场景选择记忆架构

```
Agent 需要记忆吗？
├── No → Stateless Agent（无需记忆）
└── Yes → 记忆需要跨 session 吗？
    ├── No → Session Memory（文件/变量即可）
    └── Yes → 数据类型是什么？
        ├── 结构化数据 → 键值存储（Redis/JSON）
        ├── 非结构化文本 → 向量数据库（嵌入+检索）
        ├── 关系型知识 → 知识图谱（三元组+推理）
        └── 混合 → 分层存储（短期文件 + 长期向量 + 元数据图谱）

关键权衡：
- 精确匹配 → 键值（快但无泛化）
- 语义检索 → 向量（有泛化但可能幻觉）
- 推理能力 → 图谱（最强但工程税最高）
```

**优先级**: 🔴 High — 每个多轮 Agent 都面临此决策

---

## P3: Long-Running Agent 防腐蚀模板

**来源**: [[concepts/long-running-agent-architecture]] · 

**产物**: Template — 长程 Agent 的 5 张治理牌

| 治理牌 | 触发条件 | 动作 |
|--------|----------|------|
| 🛑 成本牌 | 累计 token > 预算 80% | 切换到轻量模型或暂停 |
| ⏱️ 超时牌 | 单步骤 > 5min | 强制输出当前状态+移交 |
| 🔄 循环牌 | 连续 3 次相同工具调用 | 提醒+切换策略或人工 |
| 📉 质量牌 | 输出置信度 < 60% | 回退到上一个检查点 |
| 🔐 权限牌 | 请求未授权工具 | 拒绝+记录+请求人工 |

**优先级**: 🟡 Medium — 长程 Agent 场景越来越多

---

## P4: AI-Native 组织转型 Playbook

**来源**:  · 

**产物**: Playbook — 4 阶段渐进转型

```
Phase 1: Tool Layer (月 1-2)
- 引入 AI 编码助手（Claude Code / Cursor）
- 建立代码审查基础设施（AI 审查 → 人类审查）
- 度量：AI 辅助编码占比、审查时间变化

Phase 2: Process Layer (月 3-4)
- AI 驱动的测试生成和 bug 修复
- 自动化 PR 审查和 CI 集成
- 度量：测试覆盖率、bug 修复时间

Phase 3: Org Layer (月 5-8)
- 工程师角色重定义（引导 AI + 审查 + 架构）
- Agent 驱动的 oncall 和运维
- 度量：人均产出、oncall 负载

Phase 4: Paradigm Layer (月 9-12)
- AI-native 产品设计（从 AI 能力出发而非需求）
- 自主 agent 团队协作
- 度量：创新速度、客户价值
```

**优先级**: 🟡 Medium — 组织转型需要时间

---

## P5: Agent 安全测试清单

**来源**: [[concepts/agent-security-architecture]] · 

**产物**: Checklist — Agent 上线前安全测试

```markdown
## Agent Security Test Checklist

### 输入层
- [ ] Prompt injection 测试：已知攻击模式（直接/间接/多轮）
- [ ] 输入长度边界测试：超长输入、空输入、特殊字符
- [ ] 多语言绕过测试：用非英语 prompt 尝试绕过过滤器

### 工具层
- [ ] 未授权工具调用测试：agent 是否能调用未白名单工具
- [ ] 工具参数篡改测试：修改工具调用参数绕过限制
- [ ] 工具链组合攻击：组合多个无害工具执行有害操作

### 输出层
- [ ] 敏感信息泄露测试：agent 是否输出训练数据/系统提示
- [ ] 输出完整性验证：agent 输出是否可被篡改

### 持久层
- [ ] 记忆投毒测试：通过输入污染长期记忆
- [ ] 权限提升测试：agent 是否能获取超出授权的权限
```

**优先级**: 🔴 High — 安全测试是上线的必要条件

---

## P6: Skill 设计模式手册

**来源**: [[concepts/skill-formal-theory-survey]] · [[concepts/skill-framework-writing-patterns]]

**产物**: Pattern Catalog — 8 种 Skill 设计模式

| 模式 | 适用场景 | 复杂度 | 示例 |
|------|----------|--------|------|
| 单步工具 | 确定性操作 | ⭐ | 文件格式转换 |
| 条件分支 | 输入依赖的不同路径 | ⭐⭐ | 代码审查（按语言分支） |
| 迭代优化 | 逐步改进直到满意 | ⭐⭐⭐ | 文本润色/重构 |
| 流水线 | 多步骤顺序执行 | ⭐⭐ | 数据提取→转换→存储 |
| 递归分解 | 复杂任务分解+子任务调度 | ⭐⭐⭐⭐ | 多文件重构 |
| 人机协作 | 关键决策需人类确认 | ⭐⭐ | 部署审批流程 |
| 自适应 | 根据反馈调整策略 | ⭐⭐⭐⭐ | 调试（试错+策略切换） |
| 群体协作 | 多 agent 并行+合并 | ⭐⭐⭐⭐⭐ | 大规模代码迁移 |

**优先级**: 🟢 Low — 理论完善，待实践验证

---

## 积压状态

| ID | 产物 | 优先级 | 状态 |
|----|------|--------|------|
| P1 | Harness MVP Checklist | 🔴 High | ✅ 已产出 |
| P2 | Memory 选型决策树 | 🔴 High | ✅ 已产出 |
| P3 | 长程 Agent 防腐蚀模板 | 🟡 Medium | ✅ 已产出 |
| P4 | 组织转型 Playbook | 🟡 Medium | ✅ 已产出 |
| P5 | 安全测试清单 | 🔴 High | ✅ 已产出 |
| P6 | Skill 设计模式 | 🟢 Low | ✅ 已产出 |

---

*Practice backlog generated 2026-06-10 · All playbooks derived from 2,187 entity knowledge base*
