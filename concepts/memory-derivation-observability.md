---
title: 记忆影响谱系可观测性
created: 2026-09-10
updated: 2026-09-10
type: concept
tags: [agent-memory, observability, derivation, lineage, governance]
confidence: 0.6
provenance_state: inferred
---

# 记忆影响谱系可观测性（Memory Derivation Observability）

> 2026-09 跨簇迁移第二对（[[drafts/wiki-emergent-viewpoints-2026-09-crosscluster-memory|涌现稿]]）检验 [[concepts/channel-enumeration-criterion|通道枚举判据]]时的空白产出：记忆簇对 I/O 通道有认证语义，对**衍生通道**（原文→摘要→偏好→行为的影响链）零可观测性设计。本页是这个空白的正式登记与设计方向。

## 核心问句

**"Agent 这次为什么这样做？哪条记忆影响了它？"——当前任何库内记忆架构都回答不了。**

[[concepts/agent-memory-lifecycle-philosophies|生命周期哲学]]自认删除是影响链清算且"从来不是一键"；[[raw/articles/skilljack-persistent-skill-backdoor-tencent-mindchain-2026-09|SkillJack]] 证明影响链可以被投毒者利用（删源记录后污染仍存续）。两者的共同前提是：影响链**存在但不可见**。不可见的影响链无法清算（治理失能）、无法审计（安全失能）、无法调试（工程失能）。

## 设计方向：把衍生边变成一等记录

| 机制 | 要回答的问题 |
|------|-------------|
| 衍生边显式化 | 每条摘要/偏好记录其源记录 id（写记忆时自动登记 `derived_from` 链） |
| 影响查询 | 决策日志反向索引本次上下文注入了哪些记忆条目（注入即可观测） |
| 失效传播视图 | 删除/作废一条记忆时，列出其全部衍生后代与受影响行为类（清算的前置） |
| 谱系完整性告警 | 衍生边断裂（源被删、子代仍在生效）= 大声失败而非默默放行 |

## 与相邻机制的分工

- **写入前**：[[concepts/when-not-to-memory-architecture|何时不要建记忆系统]]的衍生预算门槛——声明这条信息允许影响什么。
- **写入后**：本页——让实际发生的影响可查询、可清算、可审计。
- **通用判据**：[[concepts/channel-enumeration-criterion|通道枚举判据]]第三簇扩展（通道清单 = I/O + 衍生）。

## 现状

全库零覆盖：无任何已摄入架构实现衍生边记录或影响查询（此为本页的空白登记来源）。评测通路亦缺：现有记忆基准（如 CL-Bench 类）度量召回与利用，不度量影响可解释性。

## 参见

- [[concepts/agent-memory-lifecycle-philosophies|记忆生命周期哲学]] — 影响链清算的治理正题
- [[concepts/when-not-to-memory-architecture|何时不要建记忆系统]] — 写入侧门槛
- [[queries/lens-proposals|提案队列]] — 本页来源（卡 #19）
