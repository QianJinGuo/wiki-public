---
title: "LongHorizon-Harness：MEA 循环重构长时程 Agent 任务状态管理"
created: 2026-09-01
updated: 2026-09-03
type: entity
tags: [long-horizon, agent-harness, task-state-management, mea-loop, alibaba, dreamx]
sources: [raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01, raw/articles/longhorizon-harness-mea-alibaba-oscholar-2026-08-05]
provenance_state: extracted
confidence: 0.85
---

# LongHorizon-Harness：MEA 循环重构长时程 Agent 任务状态管理

> -> [[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md|论文原文存档]]
> arXiv:2608.01964 | DreamX Team, Alibaba Group | GitHub: [AMAP-ML/LongHorizon-Harness](https://github.com/AMAP-ML/LongHorizon-Harness)

## 摘要

LongHorizon-Harness 将长时程执行重新定义为**任务状态管理问题**：在执行之外显式维护任务状态，仅用从环境独立验证的事实来更新。通过 Manage-Execute-Audit (MEA) 循环，Manager 定义子任务、Executor 在新鲜上下文中执行、Auditor 独立验证环境变化，实现任务状态的可靠推进。在 WeaveBench 上将 Qwen 3.7-Plus 从 51.8% 提升到 80.7%，Terminal-Bench 2.1 从 69.7% 到 77.2%，OSWorld 2.0 从 2.8% 到 8.3%。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

## 核心设计：Manage-Execute-Audit 循环

### 三大结构性挑战

长时程执行的困难不在单步，而在跨长序列维持连贯进展：**复合错误与目标漂移**（早期错误沿轨迹累积，逐渐偏离原始目标）、**上下文腐烂**（交互历史增长后相关信息难检索，性能过临界阈值急剧下降）、**任务状态丢失**（agent 无法在整个执行过程中恢复、保留和更新任务状态）。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

### 现有 harness 的两大局限

1. 任务执行和任务状态管理共享同一增长上下文——执行历史使状态越来越难追踪
2. 任务执行和完成评估保持耦合——错误判断被记录为状态前提，传播到后续决策^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

### MEA 三角色

**Manager**：拥有持久任务状态，决定任务如何推进。不直接接触计算环境——不能观察应用状态、调用工具或修改环境。决策完全基于任务状态和审计员记录的环境证据。任务状态是结构化记录集合（requirement/artifact/fact），每条标记 completed/pending/blocked/untrusted 并保留审计证据引用。**Manager 不直接改变持久状态——记录仅在有清洁审计证据支持时才标记为 completed**。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

**Executor**：在全新、有预算限制的上下文中执行子任务，是唯一被允许有意修改环境的角色。每轮执行结束后原始交互轨迹被丢弃，仅转发执行报告供审计。GUI 和 CLI 执行者通过不同环境接口操作。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

**Auditor**：只读角色，独立检查环境以确定变化、完成情况和未满足需求。审计报告记录三类发现：完成状态（complete/incomplete/blocked）、完整性状态（clean/suspect/violation）、事实发现。审计结论必须基于直接环境证据，而非执行者声明。检测到工作区变更时记录为完整性违规。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

### AgentAdapter

轻量级适配层，支持可互换的模型和 harness 后端，无需修改原生 agent 循环。支持 Claude Opus、GPT、Qwen 等模型，以及 Codex CLI、Claude Code、OpenClaw、Hermes Agent 等 harness。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

## 实验结果

### WeaveBench（114 任务，GUI+CLI 协调）

| 模型 | Harness | PassRate | Overall |
|------|---------|----------|---------|
| Qwen 3.7-Plus | Claude Code（基线） | 51.8% | 0.702 |
| Qwen 3.7-Plus | LongHorizon-Harness | **80.7%** | **0.835** |
| Claude Opus 4.7 | Claude Code（官方） | 41.2% | — |

几乎翻倍 Claude Opus 4.7 官方最佳结果，八个领域均有提升。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

### Terminal-Bench 2.1

Qwen 3.7-Plus 从 69.7% 提升到 **77.2%**。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

### OSWorld 2.0（108 桌面任务，中位人工时间 ~1.6h）

| 模型 | Binary | Partial |
|------|--------|---------|
| Qwen 3.7-Plus 基线 | 2.8% | 21.5% |
| Qwen 3.7-Plus + MEA | **8.3%** | **35.2%** |

34 任务子集 Claude Opus 4.7：20.0% → **34.3%**。^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

## 关键设计洞察

1. **状态外置**：任务状态在执行之外显式维护，仅用独立验证的事实更新——executor 声明不直接改变持久状态
2. **新鲜上下文执行**：每轮 executor 在全新上下文中运行，不接收前几轮原始轨迹——只有审计报告跨轮持久化
3. **审计隔离**：审计者独立检查环境，结论基于直接环境证据而非执行者声明
4. **可互换后端**：AgentAdapter 证明增益来自框架设计而非特定模型能力^[raw/articles/longhorizon-harness-mea-alibaba-dreamx-arxiv-2026-09-01.md]

## 与已有框架对比

| 维度 | 传统 Harness (Claude Code/Codex) | LongHorizon-Harness |
|------|--------------------------------|---------------------|
| 任务状态 | 隐式（上下文历史中） | 显式结构化记录 |
| 状态更新 | 执行者自行判断 | 仅审计证据驱动 |
| 上下文管理 | 单一增长上下文 | 每轮新鲜上下文 |
| 完成评估 | 执行者自我评估 | 独立只读审计 |
| 跨轮记忆 | 完整交互历史 | 仅审计报告 |

## 实现细节

- Executor 每轮预算 1800 秒，Manager 和 Auditor 各 300 秒
- 支持 GUI 和 CLI 两种执行者角色
- 审计报告是唯一的跨轮记忆，executor 原始交互轨迹每轮丢弃
- GitHub 开源：https://github.com/AMAP-ML/LongHorizon-Harness

## 相关实体

- [[raw/articles/longhorizon-harness-mea-alibaba-oscholar-2026-08-05|Oscholar 解读号版本]]（v×c=30，二手解读）
- 长时程 Agent 综述（towards-long-horizon-agents-survey-mozi-space）
- Terminal-Bench 2.1（lhtb-long-horizon-terminal-bench）
