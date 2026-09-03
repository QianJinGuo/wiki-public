---
title: Paper Backlog
created: 2026-05-01
updated: 2026-05-21
type: query
tags: [meta, paper]
sources:
  - raw/articles/wiki-evolver-skill-system-design-gpt55-copilot-session
  - raw/articles/model-harness-fit-agent-harness
  - raw/articles/agent-harness-12-components-7-decisions
  - raw/articles/anthopic-distillation-behavioural-traits-nature
  - raw/articles/tencent-cdn-lego-harness
  - raw/articles/memento-skills-agent-self-evolving
confidence: high
---

# 哪些 AI/ML 论文最值得深入研读？

基于 vault 中证据密度和实用性，当前最值得研读的 AI/ML 论文/主题分为五个优先级：

## P0 — 必须深入（★★★★★）

**1. Model-Harness Fit 框架（证据最密集）**
- 核心论文支撑：Anthropic Context Reset 机制对不同模型版本的差异性需求 ^、Claude Code 1.6% AI vs 98.4% 基础设施的 Harness 依赖度分析 ^、Terminal Bench 同一模型不同 harness 从 52.8→66.5 的差异 ^
- **研读价值**：理解"模型升级时哪些 harness 组件会过期"——这是 Agent 系统长期维护的核心问题

**2. Skill 质量工程体系**
- 核心论文支撑：Skill-Craft 的 7 类失效模式 × 4 模式框架 ^、技能加载幻觉实证（5400 实例×636 黄金技能） ^、技能作为外部记忆恢复 Markovian 属性（GAIA+13.7pp） ^
- **研读价值**：从"能用的提示词"到"可维护的 Agent 能力单元"的设计范式转变

## P1 — 重要参考（★★★★☆）

**3. Self-Evolving Agents 三大范式**
- 核心论文：Memento Skills ^、Self-Evolving Agents Survey（三范式 taxonomy + Co-Evolution 方向） ^、Anthropic vs OpenAI Skill 设计对比 ^
- **研读价值**：Agent 如何从"固定能力"向"持续进化"演进

**4. Post-Training 管线全景**
- 核心论文：SFT→RL→DPO→GRPO→RLVR→Agentic RL 全链路 ^、百度文心三阶段非重叠训练 ^、MSM 中训练对齐（叛逃率 54%→7%） ^[[concepts/msm-model-spec-midtraining-alignment]]
- **研读价值**：理解当前 LLM 后训练的技术收敛状态

## P2 — 选择性阅读（★★★☆☆）

**5. Agent 执行模式与编排原语**
- 核心论文：12 组件 × 7 决策框架 ^、Sub-Agent vs Agent Team 选择决策 ^、DeerFlow/Hermes/OpenClaw 超栈组合 ^
- **研读价值**：30 种组合中的高杠杆子集识别

**6. 安全盲区：蒸馏行为传递**
- 核心论文：Nature 2026 潜意识行为传递 ^、FP4 静默算错分析 ^、36% 假阳性 + AI 降低审查意愿 ^
- **研读价值**：识别当前行业安全监督的隐身缺口

## 研读优先级判断标准

| 维度 | P0 | P1 | P2 |
|------|-----|-----|-----|
| **实证密度** | 5+ 交叉验证页 | 3-4 页 | 2-3 页 |
| **未解决程度** | 核心问题开放 | 部分解答 | 初期探索 |
| **实践指导** | 直接影响架构选型 | 影响工具选择 | 影响研究方向 |

## See Also
- [[queries/research-frontier-map]] — 当前 AI Research 的前沿方向
- [[queries/llm-wiki-evaluation]] — LLM Wiki 的评测方法
- [[comparisons/agent-skill-evaluation-methods]] — 统一评估语言
