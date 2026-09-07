---
title: "Agent 记忆生命周期管理：TTL / 相关度衰减 / 合并（AgentCore Memory 架构）"
created: 2026-09-06
updated: 2026-09-07
type: entity
tags: [agent-memory, memory-lifecycle, episodic-memory, semantic-memory, procedural-memory, agentcore, memory-management]
sources: [raw/articles/designing-lifecycle-policies-for-agentcore-memory]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agent 记忆生命周期管理：TTL / 相关度衰减 / 合并（AgentCore Memory 架构）

长时间运行的 agent 会从每次对话持续生成记忆，若不主动管理，会积累过期上下文，降低响应质量并带来合规风险（实测案例：客服 agent 引用四个月前已解决的账单纠纷、重复过时的部署建议）。本架构提出 memory lifecycle management：系统性地对 agent 记忆进行评分（scoring）、合并（consolidating）、剪枝（pruning）。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

## 记忆分类学（Memory Taxonomy）

三类记忆具有不同的保留需求：^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

- **情景记忆（Episodic）**：记录发生了什么——带时间戳、会话绑定、高容量；短期连续性。最优先过期。
- **语义记忆（Semantic）**：从交互中提炼的事实与偏好（如「用户偏好 us-east-1 部署」），持久、高价值、紧凑；保留时间更长，是合并的优先候选。
- **程序记忆（Procedural）**：习得的工作流与工具使用模式（如「问成本先查 Cost Explorer API」），容量低但价值最高，保留最长、剪枝门槛最高。

## 三个生命周期策略

每个策略针对无界记忆的一个失败模式：^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

**1. TTL 过期（Policy 1）**：删除超过配置 TTL 的记忆，提供硬性累积上限。建议按类型差异化：summary 30–60 天、semantic 6–12 个月、procedural 不设 TTL。TTL 先于评分/合并运行，避免浪费计算。实现上用 `ListMemoryRecords` 的 `BEFORE` 过滤 + `createdAt` 时间戳。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

**2. 相关度衰减评分（Policy 2）**：记忆老化速率不同——昨天访问的记忆比数周未动的更相关。三项加权公式组合创建新近度、最近访问新近度、访问频率。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

**3. 合并（Policy 3）**：将多条情景观察合并为一条权威事实（semantic consolidation），减少冗余、提升检索质量。

## 部署架构

运行 nightly workflow：AgentCore memory + AWS Step Functions + Amazon Bedrock，提供 CDK stack 实例，代码开源于 aws-samples。记忆作为托管资源管理，所有阈值可配置，适配不同容量 agent（低容量可仅用 TTL + GDPR 合规）。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

## 关联

- 概念：[[concepts/agent-memory-lifecycle-philosophies|Agent 记忆生命周期哲学]]、[[concepts/episodic-vs-semantic-memory-agent|情景 vs 语义记忆]]、[[concepts/memory-consolidation-decay|记忆合并与衰减]]
- 实体：[[entities/agent-memory-architecture-essence|Agent 记忆架构本质]]、[[entities/structured-memory-filtering-metadata-agentcore-memory|AgentCore Memory 结构化过滤]]、[[entities/when-ai-agents-learn-to-forget-amazon-bedrock-agentcore-memory-philosophy|Agent 何时学会忘记]]

→ [[raw/articles/designing-lifecycle-policies-for-agentcore-memory|原文存档]]