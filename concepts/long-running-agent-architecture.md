---
title: "Long-Running Agent 架构：三大模式与演进路径"
type: concept
tags: [agent, harness, long-running-agent, adversarial-architecture, generator-evaluator, ralph-loop, context-management, drift-correction]
created: 2026-05-21
updated: 2026-08-29
confidence: high
review_value: 9
review_confidence: 8
provenance_state: synthesized
sources:
  - raw/articles/anthropic-long-running-agent-architecture-6h-retroforge
  - raw/articles/harness-design-long-running-apps
  - raw/articles/long-running-agent-ralph-loop-handover-harness-ruofei
  - raw/articles/harness-engineering-long-term-agent-tasks
---

# Long-Running Agent 架构：三大模式与演进路径

## 概述

**Long-Running Agent** 指需要跨越数小时乃至数天完成复杂工程任务的 AI Agent。与短任务不同，长周期任务面临**时间跨度大、上下文压缩失效、目标漂移累积**等独特挑战。^[raw/articles/long-running-agent-ralph-loop-handover-harness-ruofei.md]

当前行业已形成三大互补的长时运行 Agent 架构模式：

| 模式 | 代表来源 | 核心问题 | 核心解法 |
|------|----------|----------|----------|
| **对抗式架构** (Adversarial) | Anthropic/Boris Cherny | 自我评判偏差 | Generator-Evaluator 分离 |
| **三阶段循环** (Ralph Loop) | Anthropic/若飞 | 目标/上下文/质量漂移 | 工程继续 + 六项可交接标准 |
| **任务编排** (Orchestration) | hixuanxuan | 规模可控性 | CLI化 + File As Progress |

> **关键洞察**：这三种模式不是互斥的，而是互补的层次——对抗式架构解决质量治理，三阶段循环解决漂移控制，任务编排解决规模伸缩。

## 三大架构模式深度解析

### 1. 对抗式架构 (Adversarial Architecture)

**来源**：[Anthropic 工程师大会分享](https://www.youtube.com/watch?v=mR-WAvEPRwE)，Ash Prabaker & Andrew Wilson @ Anthropic^[raw/articles/anthropic-long-running-agent-architecture-6h-retroforge.md]

**核心问题**：Agent 自我评判时几乎总是偏正面，即使在人类看来质量相当平庸。这个问题在主观任务（设计）和可验证任务（代码）中都普遍存在。^[raw/articles/anthropic-long-running-agent-architecture-6h-retroforge.md]

**架构设计**：
```
宏观规划者 ──合同谈判──> 代码生成器
      │                         │
      └───── 阶段性冲刺 ────────┘
                  │
                  ↓
            视觉评论家（Playwright / 隔离提示词）
```

**核心组件**：

| 组件 | 职责 | 关键设计 |
|------|------|----------|
| **Planner** | 扩展产品规格 | 关注产品背景和高层技术设计，而非过早指定实现细节 |
| **Generator** | 按 sprint 工作 | 每轮自评后交 QA |
| **Evaluator** | 独立评估 | Playwright MCP 操作真实应用，按维度打分 |
| **Sprint Contract** | 协商完成标准 | 在写代码前定义"done"的具体含义 |

**量化效果**：

| Harness | Duration | Cost |
|---------|----------|------|
| 单代理 | 20 分钟 | $9 |
| 完整 harness | 6 小时 | $200 |

RetroForge 案例：对抗架构产出了 54 色复古调色板 + 完整物理引擎 + 嵌套 AI 关卡助手；普通循环的输出是黑色块颜色选择器 + 无法运行的半成品。^[raw/articles/anthropic-long-running-agent-architecture-6h-retroforge.md]

### 2. 三阶段循环 (Ralph Loop + Handover)

**来源**：若飞 @ 进化 AI 实验室^[raw/articles/long-running-agent-ralph-loop-handover-harness-ruofei.md]

**核心问题**：Ralph Loop（感知→推理→执行→反馈→下一轮循环）每轮都会引入微小偏差，积累形成三类系统性漂移：^[raw/articles/long-running-agent-ralph-loop-handover-harness-ruofei.md]

| 漂移类型 | 典型表现 | 后果 |
|----------|----------|------|
| **目标漂移** (Goal Drift) | 追求局部完整性，忘了核心问题 | 产出与真实需求对齐 |
| **上下文漂移** (Context Drift) | 压缩/截断导致关键信息丢失 | 推理链断裂 |
| **质量漂移** (Quality Drift) | 越做越相信自己已做完 | 测试缺失、边界错误 |

**"工程继续"范式转换**：将任务从"聊天继续"（依赖上下文记忆）改为"工程继续"（依赖外部化证据文件）。^[raw/articles/long-running-agent-ralph-loop-handover-harness-ruofei.md]

**三层证据链**：

| 层级 | 核心问题 | 工程抓手 |
|------|----------|----------|
| **目标层** | 到底要做什么？ | Goal / Non-Goal / Acceptance Criteria / 前置澄清 |
| **状态层** | 现在做到哪儿了？ | Progress / Decision Log / Git History / Milestone State |
| **治理层** | 做得对不对？ | Tests / Review Agent / Lint / Typecheck / Human Checkpoint |

**六项可交接标准**：下一个执行者（人或 Agent）能明确回答：当前目标是什么？已成事实有哪些？哪些只是猜测？哪些决策不能随便动？哪些测试能证明当前状态？真要回滚，最近的安全点在哪里？^[raw/articles/long-running-agent-ralph-loop-handover-harness-ruofei.md]

### 3. 任务编排 (Orchestration)

**来源**：hixuanxuan @ GitHub^[raw/articles/harness-engineering-long-term-agent-tasks.md]

**核心问题**：规模放大后局部失败、格式不一致；Agent 无跨会话记忆，中断 = 从头开始。^[raw/articles/harness-engineering-long-term-agent-tasks.md]

**四大核心原则**：

1. **任务拆解** — 拆成合理粒度子任务（经验上限：3000 行代码）
2. **并行执行** — 多 Agent 同时跑不同子任务
3. **可续传** — File As Progress 状态持久化
4. **有完成条件** — 客观可程序化验证的子任务成功标准

**CLI 化架构**：
```
dispatch.js ──> subagent-1 (CLI)
             ──> subagent-2 (CLI)
             ──> subagent-N (CLI)
                   ↑
              poll.js (轮询补位)
```

## 模式对比与演进关系

| 维度 | 对抗式架构 | 三阶段循环 | 任务编排 |
|------|------------|------------|----------|
| **解决的问题** | 质量治理（自我评判偏差） | 漂移控制（方向正确性） | 规模可控性 |
| **粒度** | Sprint 级别 | Loop 级别 | Task 级别 |
| **核心机制** | Generator-Evaluator 博弈 | 证据链外部化 | 状态机 + 并发调度 |
| **适用场景** | 2小时+复杂任务 | 数天超长任务 | 20+ 文件批量处理 |
| **调用成本** | 高（多 Agent） | 中（Review Agent 做 Quality Gate） | 可高可低（取决于并发度） |

> **演进路径**：大多数团队先实现任务编排（解决"能跑起来"），再引入三阶段循环（解决"方向正确"），最后在关键节点叠加对抗式评估（解决"质量达标"）。

## Context 管理：Reset vs. Compaction

两种上下文管理策略有本质区别：^[raw/articles/harness-design-long-running-apps.md]

| 策略 | 描述 | 优点 | 缺点 |
|------|------|------|------|
| **Compaction** | 在原地压缩总结，保留连续性 | 不丢历史信息 | 没有干净起点，上下文焦虑仍存在 |
| **Context Reset** | 清空上下文窗口，新 agent + 结构化交接文档 | 真正干净的上下文 | 交接文档必须携带完整状态，有额外开销 |

Opus 4.5 之前 Sonnet 4.5 的上下文焦虑严重到仅靠 compaction 无法支撑高质量表现；Opus 4.6 之后这种行为消失，可以完全移除 context reset。**说明 harness 组件的必要性高度依赖模型版本**。^[raw/articles/harness-design-long-running-apps.md]

## 核心工程实践

### 1. 文件记忆分层制度

防止"假设"悄悄写成"事实"导致后续 Agent 集体跑偏。^[raw/articles/long-running-agent-ralph-loop-handover-harness-ruofei.md]

| 层级 | 说明 | 处理方式 |
|------|------|----------|
| **Facts** | 客观事实 | 直接记录 |
| **Observations** | 观察结果 | 标注来源 |
| **Hypotheses** | 假设（需标注置信度） | 必须附带推翻条件 |
| **Decisions** | 决策及原因 | 记录决策依据 |

### 2. 分层重试策略

^[raw/articles/harness-engineering-long-term-agent-tasks.md]

- **内层**：会话恢复（同一 conversationId 续传），不设上限
- **中层**：带反馈重试（新会话 + 错误上下文），限 2-3 次
- **外层**：主 Agent 重新调度（FAILED 文件重新分组）

### 3. Harness 组件审计

每次只移除一个组件，观察对最终结果的影响。^[raw/articles/harness-design-long-running-apps.md]

> 新模型发布后，用单代理 baseline 跑相同任务，对比有/无各组件的表现。如果移除某组件后质量下降明显，重新加入；如果质量不变，说明该组件对这个版本已经过期。

## 关联

### 核心实体
- [[entities/anthropic-long-running-agent-adversarial-architecture|Anthropic 长时运行 Agent 架构：对抗式设计 + 合同谈判 + 审美量化]]
- [[entities/harness-generator-evaluator-anthropic|Claude Harness 设计：Generator-Evaluator 架构与 Context Reset 演进]]
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei|长周期 Agent 详解：从 Ralph Loop 到可接管 Harness]]
- [[entities/harness-engineering-long-term-agent-tasks|Harness Engineering：让 Coding Agent 可靠完成长程任务]]

### 相关概念
- [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- [[concepts/agent-memory-lifecycle-philosophies|Agent 记忆生命周期哲学]]
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]

## 新增关联实体
- [[entities/forgetrain-ai-written-training-framework-bidian-infoq]]
- [[entities/nextie-alpha-cognitive-model-4b-on-device]]

## 关联实体

**上游依赖**:
- [[entities/anthropic-long-running-agent-adversarial-architecture]] — 提供基础理论/方法
- [[entities/harness-generator-evaluator-anthropic]] — 提供基础理论/方法

**下游应用**:
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei]] — 具体应用场景
- [[entities/harness-engineering-long-term-agent-tasks]] — 具体应用场景

**平行协作**:
- [[entities/forgetrain-ai-written-training-framework-bidian-infoq]] — 替代/补充方案
- [[entities/nextie-alpha-cognitive-model-4b-on-device]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-5-production-security|Layer 5 Production Security]]
