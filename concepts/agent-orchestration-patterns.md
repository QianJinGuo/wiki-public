---
title: Agent Orchestration Patterns
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [orchestration, workflow, multi-agent, sub-agent, harness, patterns]
sources: [raw/articles/four-sub-agent-patterns-2026, raw/articles/harness-engineering-long-term-agent-tasks, raw/articles/aws-agent-orchestration-workshop, raw/articles/inngest-ai-in-production-the-2026-benchmark-report-2]
confidence: high
provenance_state: merged
---

# Agent Orchestration Patterns

Agent 编排是让多个 AI Agent 协同完成复杂任务的核心能力。本概念页整合当前主流的编排模式、工作流框架和生产级实践，形成对 Agent 编排的系统性认知。

## 四种 Sub Agent 模式：控制粒度与状态保留

[[entities/four-sub-agent-patterns|四种 Sub Agent 模式]]按主 Agent 对子 Agent 生命周期的控制程度由低到高排列：

| 模式 | 主 Agent 角色 | Agent 生命周期 | 结果收集 | 适用场景 |
|------|-------------|--------------|---------|---------|
| **内联工具** | 调用方 | 单次任务 | 工具返回值 | 自包含独立任务 |
| **Fan-Out** | 调度方 | 单次任务 | wait_agent 批量 | 多个独立并行任务 |
| **Agent Pool** | 协调方 | 多轮持久 | 逐消息增量 | 多步骤协作工作流 |
| **Teams** | 监督方 | 持久化 | Agent 主动汇报 | 超出单 Agent 协调能力上限的大型任务 |

^[raw/articles/four-sub-agent-patterns-2026.md]

**模式升级的决策树**可概括为三个问题：

1. 任务是否需要子 Agent 完成后才能继续？（否 → 考虑 Fan-Out；是 → 继续）
2. 任务中途是否可能需要补充信息或纠偏？（否 → 内联工具足够；是 → 继续）
3. 协作是否跨越多个独立步骤且需要跨轮次状态？（否 → Fan-Out；是 → Agent Pool）
4. 任务规模是否已超出单个 Agent 的协调能力上限？（否 → Agent Pool；是 → Teams）

**Teams 模式的工程挑战**尤为突出：死锁检测（A 等 B，B 等 A）需要额外的健康检查机制；冲突解决（两 Agent 同时修改同一文件）需要锁或版本控制；关闭协调（graceful shutdown 所有 Agent）比单 Agent 系统复杂得多。

## Harness Engineering：长程任务的可靠执行框架

[[entities/harness-engineering-long-term-agent-tasks|Harness Engineering]] 的核心目标是让 Agent 能够可靠完成涉及成百上千文件、跨越多个会话、消耗数千万 Token 量级的长程任务。四大核心原则：

1. **任务拆解** — 拆成合理粒度子任务，控制单次执行复杂度
2. **并行执行** — 多 Agent 同时跑不同子任务，压缩整体耗时
3. **可续传** — File As Progress 状态持久化，消除中断沉没成本
4. **有完成条件** — 客观可程序化验证的子任务成功标准

^[raw/articles/harness-engineering-long-term-agent-tasks.md]

**三大困难点**的根因与对应原则：

| 困难 | 根因 | 对应原则 |
|------|------|---------|
| 上下文耗尽 | 上下文窗口有限，压缩叠加导致细节丢失 | 任务拆解 |
| 中断要重来 | Agent 无跨会话记忆，中断=从头开始 | 可续传 |
| 规模行为不可控 | 规模放大后局部失败、格式不一致 | 完成条件 |

**Meta-Skill：Skill for Skill** 是将长程任务编排经验本身做成可复用框架的思路。骨架结构包含 discover.js（扫描目标）、dispatch.js（并发调度）、poll.js（状态轮询补位）、merge.js（结果合并）、status.js（进度查询）。

## AWS 编排工具矩阵：确定性 vs 推理驱动

[[entities/aws-agent-orchestration-workshop|AWS Agent Orchestration Workshop]] 揭示了编排工具选型的核心二分法：

**确定性工作流编排**（AWS Step Functions）适合结构化、可预测的流程——精确性、可审计性、事务性是核心优势。**推理驱动型 Agent 协调**（Amazon Bedrock Agents）适合需要动态路由和工具调用的场景——灵活性、自适应能力是核心优势。前者保证精确性，后者提供灵活性。^[raw/articles/aws-agent-orchestration-workshop.md]

Marketplace 上的编排工具矩阵：

| 工具 | 定位 | 适合场景 |
|------|------|---------|
| **Temporal Cloud** | Durable execution | 长时间运行、有状态的工作流 |
| **Orkes Cloud** | Conductor 商业版 | 企业级工作流治理 |
| **Prefect Cloud** | Python-first | 数据管道和 ETL |
| **Amazon MWAA** | Managed Airflow | 已有 DAG 资产的团队 |
| **Step Functions** | AWS 原生状态机 | AWS 深度集成的确定性工作流 |

**Human-in-the-loop 是生产级 Agent 系统的必备能力**：在金融、医疗等合规要求严格的领域，关键决策必须有人工审批节点。AWS Step Functions 的 approval 步骤支持暂停执行等待人工确认。

## Inngest 生产基准：编排+可观测性+Evals 三层协同

[[entities/inngest-ai-in-production-the-2026-benchmark-report]] 基于 130 名工程师的调研，揭示了高信心 AI 团队的共同特征：

**信心团队的三个基础设施层**：

1. **编排层（Orchestration）**：持久化状态、处理故障的工作流编排
2. **可观测性层（Observability）**：嵌入工作流内部的监控与调试能力
3. **Evals 层（Evals）**：连接到实际故障点的评估机制

当这三个层共享上下文时，信心随之而来。最强的正向组合包括：持久化执行 + 使用 evals + 报告可靠性开销下降。

**可观测性是 #1 未解决难题**（19% 的受访者），高于任何其他主题。这个挑战在 AI 团队（18%）和非 AI 团队（21%）中基本一致，说明可观测性挑战并非 AI 独有，而是异步工作流本身的固有难题。

**可靠性税**：20% 的 AI 团队将高达一半的工程时间投入到可靠性工作中——这一「可靠性税」的本质是 AI 系统固有的不确定性与生产环境稳定性要求之间的结构性矛盾。

## 深度分析：编排模式的本质

### 控制粒度与状态保留的两个维度

四种 Sub Agent 模式的核心差异在于**控制粒度**与**状态保留**的两个维度。

**控制粒度**指主 Agent 对子 Agent 任务执行过程的话语权。内联工具模式几乎没有任何过程控制能力；Fan-Out 稍进一步，允许主 Agent 在等待期间穿插其他工作，但仍无法干预运行中的子 Agent；Agent Pool 赋予主 Agent 完整的生命周期管理权；Teams 模式则将控制权进一步下放给子 Agent 自身。

**状态保留**指子 Agent 的上下文是否能跨任务持续存在。内联工具和 Fan-Out 模式中，子 Agent 每次执行都是独立的、单次性的；Agent Pool 和 Teams 模式则维持持久化 Agent 实例，跨多轮交互保持工作记忆。

### File As Progress 的双通道设计

Harness Engineering 中，进度状态同时写两套载体：给人看的高可扫终端摘要（数字、比例、异常），给 Agent 读的结构化状态文件（任务 ID、状态、产出路径、失败原因）。深层逻辑在于：人恢复时需要「现在跑到哪了」的直观感知；Agent 恢复时需要「从哪继续」的精确指令，两者数据结构完全异构。

### IN_PROGRESS 残留处理的判断依据

Agent 在执行过程中被中断时，恢复逻辑的关键不是「状态是什么」，而是「产出物是否存在且合法」。具体判断分三步：文件是否存在、内容能否通过合法性校验、内容完整则状态直接更新为 DONE，否则清理工作区后重置为 TODO。

## 实践启示

1. **从模式 1 开始，在遇到真实瓶颈时才升级**。过早引入 Agent Pool 或 Teams 模式会引入不必要的系统复杂度——每个模式都带来额外的工程负担

2. **编排层、可观测性层、Evals 层必须协同设计**。孤立地优化任何一个层次效果有限；真正有效的是三个层次的协同设计，三层共享上下文时才能形成闭环的可靠性保障体系

3. **小模型/便宜模型强烈建议待在模式 1 或 2**。当子 Agent 需要调用前沿级别模型才能正确工作时，Teams 模式的成本会快速叠加

4. **并行任务优先考虑 Fan-Out**。很多独立任务（多个文件的代码审查、多个主题的资料查询）天然适合并行处理，Fan-Out 在保持简单性的同时提供了启动和收集的解耦

5. **Human-in-the-loop 流程要在概念验证阶段就引入**。不要等系统上线后才考虑审批节点——从第一个生产级 Agent 应用开始就设计人工确认步骤，形成可审计的操作记录

6. **建立配置版本管理机制**。「为当前模型写的指令，下一代模型可能适得其反」——企业需要把 agent 配置视为「一等公民」来管理，每 3-6 个月做一次完整配置审查

## 相关概念

- [[entities/four-sub-agent-patterns|四种 Sub Agent 模式]] — 控制粒度与状态保留的完整框架
- [[entities/harness-engineering-long-term-agent-tasks|Harness Engineering]] — 长程任务的四大原则和 Meta-Skill
- [[entities/aws-agent-orchestration-workshop|Agent Orchestration Workshop]] — AWS 编排工具矩阵
- [[entities/inngest-ai-in-production-the-2026-benchmark-report]] — 信心团队的三个基础设施层
- [[entities/anthropic-multi-agent-research-system|Anthropic Multi-Agent Research]] — 多 Agent 评估方法
- [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- [[concepts/multi-agent-systems|Multi-Agent Systems]]

## 关联实体

**上游依赖**:
- [[entities/four-sub-agent-patterns]] — 提供基础理论/方法
- [[entities/harness-engineering-long-term-agent-tasks]] — 提供基础理论/方法
- [[entities/aws-agent-orchestration-workshop]] — 提供基础理论/方法

**下游应用**:
- [[entities/inngest-ai-in-production-the-2026-benchmark-report]] — 具体应用场景
- [[entities/four-sub-agent-patterns]] — 具体应用场景
- [[entities/harness-engineering-long-term-agent-tasks]] — 具体应用场景

**平行协作**:
- [[entities/aws-agent-orchestration-workshop]] — 替代/补充方案
- [[entities/inngest-ai-in-production-the-2026-benchmark-report]] — 替代/补充方案
- [[entities/anthropic-multi-agent-research-system]] — 替代/补充方案

## 所属 MOC

- [[moc/amazon-aws-ai|Amazon Aws Ai]]
