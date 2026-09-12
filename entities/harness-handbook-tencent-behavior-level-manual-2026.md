---
title: "Harness Handbook — 行为级 Agent Harness 手册：可理解、可审计、可编辑"
created: 2026-07-18
updated: 2026-09-11
type: entity
tags: [agent, harness, harness-engineering, behavior, manual, tencent, research]
sources: [raw/articles/harness-handbook-tencent-ruhan-wang-2026, raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026]
confidence: 0.9
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Harness Handbook — 行为级 Agent Harness 手册：可理解、可审计、可编辑

> **Background**：本文档基于腾讯 HY LLM Frontier 与 Indiana University 联合发布的 Harness Handbook 研究项目建立。该项目提出用三层行为级手册（L1-L3）组织 agent harness 代码，使复杂 system 行为可浏览、可验证、可修改。参考了项目官网、GitHub 仓库、Handbook Studio Demo 多源信息。

## 核心问题

Agent harness 的 behavior 散布在数千个文件中（Codex 有 2,267 文件、34,000+ 函数、160,000 代码连接），传统文件树展示"代码在哪里"但不展示"这些代码如何协作产生行为"。搜索 `delete`、`permission`、`confirm` 只返回散落的片段，无法重建完整的 behavior chain。^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]

Harness Handbook 的核心洞见：**问题不是缺少代码，而是缺少从 behavior 到 implementation 的路径**。需要一个组织概念——behavior——来连接 execution 和 code。^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]

## 三层结构（L1-L3）

| 层级 | 回答的问题 | 产出 |
|------|-----------|------|
| L1 · System Overview | 这个 harness 整体怎么运行？ | 架构、执行流、主要阶段、状态流 |
| L2 · Behavior-Unit Overview | 有哪些 behavior unit，它们怎么连接？ | 职责、输入/输出、依赖、关键状态 |
| L3 · Behavior-Unit Detail | 这个 behavior unit 如何执行？ | 触发条件、状态变化、异常路径、代码证据 |

每层保留可验证的代码证据链接，读者可以检查每个解释并在源码中验证。^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]

## Behavior-Guided Progressive Disclosure (BGPD)

BGPD 将 behavior question 转化为可追踪的证据路径：^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]


```
行为问题 → L1 系统上下文 → L2 定位相关 behavior unit → L3 打开展示实现细节 → 代码证据
```

每个步骤只揭示当前决策需要的信息。理解、审计、修改共享同一个证据路径。^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]

## Handbook 生成流程

采用 facts-first 方法，而非让模型逐文件总结：^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]


1. **提取事实** → 程序图（静态分析：文件、函数、调用关系、状态读写、配置边界）
2. **按行为组织** → 行为图（proposer-reviewer 循环，直至收敛）
3. **合成手册** → L1-L3 渲染（每段 prose 锚定在提取的程序事实上）

> "prose explains; facts anchor"——每一个 claim 都链接到可验证的代码证据。^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]

## 评估结果（论文原文完整数据，2026-09-11 SUPP 补强）

使用相同 coding agent 在 Terminus-2 和 Codex 两个生产 harness 上对比，仅变化是否提供 Handbook。三个独立 judge（GPT-5.5、Opus 4.8、DeepSeek-V4-Pro）。每个框架 30 个行为驱动修改请求（Q/CF/SH × Easy/Medium/Hard）。judge 胜负阈值 δ=3 分（0-100 量表，Localization 权重最大）^[raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026.md]

**发现一：质量更高、token 更少**——总体胜率 Codex 38.3% vs 基线 28.3%（+10.0pp，三 judge 方向一致）；Terminus-2 45.6% vs 26.7%（+18.9pp，gap 13.3-26.7pp）。规划 token 反而下降：Codex 0.102M→0.089M（-12.7%）、Terminus-2 0.058M→0.053M（-8.6%）——证明增益不是靠更多 token 预算 ^[raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026.md]

**发现二：弱规划器+手册 ≈ 强模型**——DeepSeek-V4-Pro 作规划器，对 Opus 4.8 与 GPT-5.5 两套独立参考计划，24 项 file/symbol 级 Recall/Precision/F1 对比全部占优（F1 增益 +5.0～+18.8pp）。完全定位失败（Wrong=与参考零重叠）最多降 25.9pp（Codex symbol Wrong 44.4%→18.5%）。Terminus-2 上手册辅助的弱规划器 file-F1 达 84.7%/89.3%、symbol Precision 对 GPT-5.5 达 93.3% ^[raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026.md]

**发现三：全类型全难度普适**——6 组 harness×请求类型全正（+16.3～+33.3pp；Codex 最大增益在 Q 类 +26.7，Terminus-2 最大在 SH 类 +33.3——恰是关键词搜索最弱场景）；6 组 harness×难度全正（+3.7～+33.3pp），且增益与标注难度非单调 ^[raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026.md]

按维度拆解（三 judge 平均）：Localization 增益最大（Terminus-2 +12.2 / Codex +2.2），Scope Control（+6.7 / +1.1），Reasoning（+4.5 / +3.3）^[raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026.md]

早期项目页定性结论（2026-07-18 版）：preference rate 更高、token/case 更低、file/symbol 级 recall/precision/F1 全面上升、wrong cases 急剧下降、跨难度持续。^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]

## 表征的两条铁律与双叶模式（论文原文）

- **Progressive disclosure**：读者仅在任务需要时从 L1 向 L3 深入
- **Behavior–implementation alignment**：每个活跃 L3 locator 必须能在当前仓库 revalidate；无法重验证的条目被 **frozen**（冻结、排除出定位）直到刷新——仓库永远是实现细节的唯一权威
- 两种叶模式（handbook 生命周期内固定）：**function-as-leaf**（中小项目，已知种子骨架自上而下填函数，多角色函数可拆连续代码区分别归属）与 **file-as-leaf**（大型项目如 Codex 2,200+ 源文件，先为每文件生成文件卡片→推断阶段骨架→自下而上分配）

^[raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026.md]

## 自动 Resynchronization（Algorithm 1）

修改工作流四步：BGPD 定位（L1/L2 选阶段 → 沿状态寄存器视图 Z 纳入共享状态耦合的远距阶段 → 选 L3 条目 → 沿调用图扩展候选〔外部边界节点只作上下文、绝不作为编辑点〕→ 打开当前仓库逐点验证，产出 file path+anchor+当前源码摘录的 evidence）→ Planner 转化为 edit plan P 与 action declarations Γ → 独立 executor 应用 P → **任何非空 diff 自动触发 resync**（只更新受影响部分而非重建整个手册），Handbook 与代码永不脱同步 ^[raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026.md]

## 自进化展望

论文结论指出下一步是 **harness self-evolving**：把 Handbook 作为共享行为记忆，Agent 自主闭环"定位–规划–执行–重新同步"，随仓库演化持续自改进。Handbook 作为与代码保持同步的行为中心仓库表征，还可支撑 behavior auditing 与 regression-impact analysis ^[raw/articles/harness-handbook-tencent-full-paper-ruhan-wang-2026.md]

## Handbook Studio

Interactive workbench —— 连接仓库 → 生成三层 Handbook → 在同一行为图上阅读/验证/提议修改。用户以行为级意图发起改动（如"让这个命令携带自己的环境变量"），系统定位所有 affected implementation sites（14 个代码点、10 个文件）并生成可审查的 edit plan 和 diff。^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]

## 与其他 Harness 实体的关系

已有大量 [[entities/agent-harness-architecture.md|agent harness 架构]] 实体覆盖 harness 的设计模式、组件和工程实践。Harness Handbook 提供了不同的角度——**以 behavior 为核心的导航系统**（而非以组件/模块为核心）。它与以下实体互补：^[raw/articles/harness-handbook-tencent-ruhan-wang-2026.md]


- [[entities/harness-engineering.md|Harness Engineering]]、[[concepts/harness-engineering-framework|Harness Engineering Framework]]（工程范式）
- [[entities/agentic-loop-engineering-handbook-empirical-framework.md|Agentic Loop Engineering]]（loop 工程）
- [[entities/agent-harness-12-components-7-decisions.md|Agent Harness 12 Components]]（组件架构）
- [[entities/better-harness-eval-trace-methodology.md|Better Harness Eval]]（评估方法）

→ [[raw/articles/harness-handbook-tencent-ruhan-wang-2026|原文存档]]
