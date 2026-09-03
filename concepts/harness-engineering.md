---
title: "Harness Engineering 规范主页（Hub）"
created: 2026-08-29
updated: 2026-08-30
type: concept
tags: [concept, harness, hub, agent-engineering, meta, navigation]
sources: [concepts/harness-engineering-framework, concepts/agentic-engineering-paradigm, entities/harness-engineeringai-能在真正出事会炸的后端系统里写代码吗, entities/agent-harness-engineering-survey-2026]
confidence: 0.85
provenance_state: merged
---

# Harness Engineering 规范主页（Hub）

> 本页是 harness 概念族的**规范主页**：`concepts/` 下 20+ 个 harness/agentic 页面长期 orbiting 一个不存在的中心（[[drafts/wiki-emergent-viewpoints-2026-07|2026-07 涌现观点·观点四]] 的诊断），本页补上这个中心，并为「模型能力 vs 工程体系」之争提供一个家。

## 一句话定义

**Harness = 除模型外的所有东西**。核心方程：`Agent = Model + Harness`。它决定模型看到什么（context）、能做什么（tools）、按什么规则做（权限/纪律）、做错了怎么纠偏（verifier/loop），以及如何把能力稳定交付出来。三次工程重心的演进是 **Prompt ⊂ Context ⊂ Harness**——从「怎么说」到「给什么」到「别跑偏」，详见 [[concepts/harness-engineering-framework|Harness Engineering 框架]]。

## 概念族地图

**范式与定位**
- [[concepts/agentic-engineering-paradigm|Agentic Engineering 范式]] — Karpathy 2026：harness + context + verifier 三件套，问责性优先
- [[concepts/agent-harness-engineering-paradigm|Agent Harness Engineering 范式]] / [[concepts/harness-engineering-paradigm-shift|范式迁移]] — 从写代码到运维 agent 的重心转移
- [[concepts/harness-engineering-7-layers-framework|七层框架]] — 分层解构；[[concepts/ahe-agentic-harness-engineering|AHE（Agentic Harness Engineering）]] — 复旦/北大提出的学科化命名
- [[concepts/harness-as-product-surface|Harness 作为产品表面]] — harness 不只是工程件，是用户接触面
- [[entities/agent-harness-engineering-survey-2026|Agent Harness Engineering 综述（2026）]] — 实体侧全景

**结构与权衡**
- [[concepts/100-line-vs-managed-harness-tradeoff|100 行循环 vs 托管 Harness]] — 自建最小循环与生产级平台的权衡轴
- [[concepts/coding-harness-engineering|Coding Harness]] — 编码场景的 harness 细节
- [[concepts/harness-loop-architecture|Loop 架构]] / [[concepts/harness-tool-design-evolution|Tool 设计演化]] / [[concepts/harness-context-window-management|Context 窗口管理]] — 三大内部子系统
- [[concepts/harness-long-running-task|长时任务]] — 数小时级任务的断点/恢复；姊妹篇 [[entities/harness-design-long-running-apps|长时应用设计]]
- [[concepts/sdd-specification-driven-development-harness|Spec 驱动开发 Harness]] / [[concepts/ai-team-knowledge-harness|团队知识 Harness]]

**生命周期与治理**
- [[concepts/harness-component-expiry-build-to-delete|组件过期与 Build-to-Delete]]（及其 [[concepts/harness-component-expiry-and-build-to-delete|合并版]]、[[concepts/harness-component-expiry-evidence|证据版]]——本族自身就是碎片化的活标本）— harness 组件按预期寿命设计，到期即删
- [[concepts/routa-harness-visualization|Routa 可视化]]

**验证与评测**（harness 的质量面）
- [[concepts/verifier-driven-development|Verifier 驱动开发]] — verifier 必须存在的正面陈述
- [[concepts/verifier-paradox|Verifier 悖论]] — 同一原则的边界条件：验证能力跟不上生成能力时会发生什么
- [[concepts/eval-optimizer-firewall|评测防火墙]] — optimizer 存在时 verifier 的加固层
- [[concepts/evaluation-harness-design|评估 Harness 设计]]

**瓶颈迁移**
- [[concepts/ai-r-and-d-when-ai-builds-itself-bottleneck-shift-r-d-harness|AI R&D 瓶颈迁移]] — 当 AI 自己做研究时，瓶颈从模型转移到 harness
- [[entities/self-harness-shanghai-ai-lab-agent-improves-harness|Self-Harness]] — harness 自改进的范式化（2026-06）

## 中央矛盾：模型能力 vs 工程体系

本库被引用最高的 harness 实证页 [[entities/harness-engineeringai-能在真正出事会炸的后端系统里写代码吗|LEGO Harness Engineering]] 结尾有一条**长期悬空的矛盾注记**——反方观点（「AI Coding 的核心瓶颈不在工程体系而在模型能力本身」）从未有过页面可指。本页收留这条线程：

- **正方（工程体系是瓶颈）**：同一基模在不同 harness 下表现差距巨大（本库 harness 实证页群的共同结论）；Self-Harness 证明固定权重模型靠 harness 自进化在 Terminal-Bench 上最高 +138% ^[raw/articles/self-harness-shanghai-ai-lab-agent-improves-harness-hyman-2026-06-12.md]。
- **反方（模型能力是瓶颈）**：harness 无法弥补模型不会的东西；AI 产出超过人类理解能力时，整套验证体系退化为合规仪式（见 [[concepts/verifier-paradox|Verifier 悖论]]）。
- **2026-08 的综合**（[[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|phd 透镜涌现]]）：争论正在从二选一变为分工——模型能力决定上限，harness 决定下限与方差；固定基模后竞争全部下沉到 harness 层，且 harness 层竞争的焦点是经验资本化与验证防腐蚀。

## 导航

- 学习路径入口：[[moc/agent-engineering-guide|Agent Engineering Guide]] · [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
- 循环专项：[[moc/loop-engineering|Loop Engineering]]
- 记忆专项：[[moc/memory-context-systems|Memory & Context Systems]]
