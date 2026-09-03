---
title: "研究前沿图 — 2026 Q2"
created: 2026-06-10
updated: 2026-06-10
tags: [frontier, research, dashboard]
type: query
---

# 研究前沿图 — 2026 Q2

> 基于 2,187 实体 + 74 概念的知识图谱分析，识别 6 大前沿集群及其开放问题。

---

## 前沿 1: Harness Engineering — 从范式到工程

**核心论点**: AI Agent 的可靠性不取决于模型能力，而取决于 harness 质量。Harness Engineering 正在从"有趣的观察"进化为"可执行的工程学科"。

**知识基础**:
- [[concepts/ahe-agentic-harness-engineering]] — 36 实体支撑
- [[concepts/harness-engineering-framework]] — 31 实体
- [[concepts/harness-engineering-paradigm-shift]] — 三次范式跃迁
- [[concepts/coding-harness-engineering]] — Coding Harness 本质
- [[concepts/harness-component-expiry-build-to-delete]] — Build to Delete

**开放问题**:
1. Harness 的组件保质期如何量化？Model-Harness Fit 的度量标准是什么？
2. 从"vibe coding"到"agentic engineering"的过渡点如何检测？
3. 长程 Agent（6h+）的 harness 侵蚀率是多少？如何防腐蚀？
4. Harness 的最小可用检查清单（MVP Harness）需要哪些组件？

**关键实体**:  (68 refs) ·  (62 refs) ·  (47 refs)

---

## 前沿 2: Agent Memory — 存储学派与架构选择

**核心论点**: Agent Memory 不是"加一个向量数据库"，而是一个需要工程权衡的存储系统设计问题。六大存储学派代表了不同的架构选择。

**知识基础**:
- [[concepts/agent-memory-system-design]] — 32 实体
- [[concepts/agent-memory-lifecycle-philosophies]] — 生命周期哲学
- [[concepts/agent-memory-systematic-framework]] — 系统性框架
- [[concepts/catastrophic-forgetting]] — 灾难性遗忘

**开放问题**:
1. 六大存储学派（文件/向量/图/键值/关系/混合）的选择标准是什么？
2. 记忆的工程税（存储成本、检索延迟、一致性维护）如何量化？
3. 长期记忆的过期策略：时间衰减 vs 使用频率 vs 显式标记？
4. 多 Agent 共享记忆的一致性协议如何设计？

**关键实体**:  (53 refs) ·  (52 refs)

---

## 前沿 3: Claude Code 架构 — 从源码到实践

**核心论点**: Claude Code 是目前最深入研究的 AI Coding Agent 实现，其架构决策（工具设计、上下文管理、harness 模式）代表了行业最佳实践。

**知识基础**:
- [[concepts/claude-code-deep-architecture-analysis]] — 架构深度分析
- [[concepts/claude-code-source-leak-lifecycle]] — 源码级生命周期
- [[concepts/claude-code-tool-design-evolution]] — 工具设计演化
- [[concepts/kairos-claude-code-paradigm]] — KAIROS 常驻协作
- [[concepts/openclaw-architecture]] — 800行实现

**开放问题**:
1. Claude Code 的 task boundary 检测机制如何泛化到非编程任务？
2. Context window 的 working set 管理策略在 >1M token 场景下如何演进？
3. Skills 封装与 MCP 工具调用的边界在哪里？
4. OpenClaw 的 800 行实现是否代表了"最小可用 Agent 架构"？

**关键实体**:  (53 refs) ·  (52 refs) ·  (51 refs)

---

## 前沿 4: Agent 安全 — 从攻击到治理

**核心论点**: Agent 安全正在从"prompt injection 防御"升级为"全生命周期安全体系"，核心挑战是身份验证和权限管理。

**知识基础**:
- [[concepts/agent-security-architecture]] — 安全架构
- [[concepts/agent-security-full-lifecycle-system]] — 全生命周期
- [[concepts/agent-security-threat-models]] — 威胁模型

**开放问题**:
1. Prompt injection 的缓解率上限是多少？是否存在"不可能完全防御"的证明？
2. 多 Agent 系统中的身份冒用攻击如何检测？
3. Agent 的权限膨胀（privilege creep）如何自动检测和回收？
4. 供应链攻击（xz-utils 模式）在 Agent 生态中的风险倍增效应如何量化？

**关键实体**:  · 

---

## 前沿 5: 多 Agent 协作 — 从单体到群体智能

**核心论点**: 多 Agent 系统的核心挑战不是技术实现，而是协作模式的选择和协调成本的控制。

**知识基础**:
- [[concepts/multi-agent-systems]] — 31 实体
- [[concepts/multi-agent-collaboration-patterns]] — 协作模式
- [[concepts/agent-orchestration-patterns]] — 编排模式

**开放问题**:
1. 中心化编排 vs 去中心化协商：什么场景选什么模式？
2. Agent 间的通信成本（token/延迟/错误传播）如何优化？
3. 群体决策中的"共识陷阱"（过早收敛到次优解）如何避免？
4. 多 Agent 的调试和可观测性如何设计？

**关键实体**:  (47 refs) ·  (43 refs)

---

## 前沿 6: 开源 AI 生态 — 从开放权重到开放治理

**核心论点**: "开放权重"不等于"开源"，开源 AI 需要类似 Linux Foundation 的治理结构来保证长期可持续性。

**知识基础**:
- [[concepts/open-source-ai-ecosystem]] — 35 实体
- [[concepts/transformer-architecture]] — 35 实体
- [[concepts/inference-optimization]] — 31 实体

**开放问题**:
1. 开放模型联盟（Open Model Consortium）是否会出现？由谁主导？
2. 模型蒸馏（distillation）的法律和伦理边界在哪里？
3. 开源 AI 的可持续商业模式是什么？
4. 推理优化的天花板：算法优化 vs 硬件优化，哪个杠杆更大？

**关键实体**: [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]] ·  (48 refs)

---

## 跨前沿连接

```
Harness Engineering ←→ Agent Memory (harness 管理记忆工程税)
Harness Engineering ←→ Agent Security (harness 是安全第一道防线)
Claude Code 架构   ←→ Harness Engineering (最完整的 harness 实现)
多 Agent 协作      ←→ Agent Security (身份验证是协作基础)
开源 AI 生态       ←→ Harness Engineering (开放 harness 标准化)
```

---

*Frontier map generated 2026-06-10 · Based on 2,187 entities, 74 concepts, 10,460 cross-references*
