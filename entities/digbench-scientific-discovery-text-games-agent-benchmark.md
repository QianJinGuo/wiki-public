---
title: "dig.bench — 文本游戏科学发现智能体基准"
created: 2026-08-19
updated: 2026-11-24
type: entity
tags: [agent-benchmark, scientific-discovery, text-games, agent-evaluation, experimentation, rule-discovery, harness]
sources: [raw/articles/digbench-scientific-discovery-text-games-agent-benchmark]
confidence: 0.78
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# dig.bench — 文本游戏科学发现智能体基准

## 概述

dig.bench 是一个以**科学发现**为目标的智能体基准：70 个交互式文本游戏，**每个游戏衡量智能体能否通过实验发现游戏自身的未知规则**（Discovering unknown rules in text-based games）。配备 21 个公开 API 文档，并提供 Paper / GitHub / Leaderboard。^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]

## 设计要点

- **科学发现的实验能力**：不只测「答对问题」，而测智能体能否主动设计实验、从反馈中归纳未知规则——把 LLM 的科学假设-检验循环压缩进文本游戏环境^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]
- **人类可基准**：每个游戏都至少被一位人类测试者在首次尝试中击败，难度分层——所有游戏都是人类首试可达的^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]
- **得分机制**：胜率 = 每个游戏 runs 均值再跨 tier 的 10 个游戏平均；Tier 划分覆盖从 Basic Harness 到 Agentic Harness（基础模型 vs 带 Agent 框架的完整编排）^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]

## 深度分析

### 为什么文本游戏是廉价却严格的发现试验台

dig.bench 最关键的设计决策是**纯文本载体**：原文明确指出，文本环境「没有视觉混淆横在模型与发现之间」，被测的就是发现本身^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]。这使「人类可解而前沿模型难以攻克顶层游戏」的差距成为关于科学推理能力的干净信号，而非感知或模态适配的副产物——综合基准的分数往往混合多种因素，dig.bench 用载体约束换来了归因清晰度。严格性还来自另外两个工程选择：交互式文本游戏的构建成本远低于真实实验室环境，规则由设计者完全掌握，因此「未知规则」可被精确设置、任意次重复——用可控的廉价模拟替代了真实科研中实验昂贵、真值未知的天然劣势，换取了发现过程的可复现性。

### 「发现」在 dig.bench 中的操作化定义

原文对胜利条件的规定把「科学发现」压成一个可判定闭环：智能体必须先**发现未知规则**，再**把规则应用于解决挑战**，并在有限步数内通关。这意味着发现不只是「说出规则」，还必须在行为上兑现——假设要被行动验证。人类与模型通过相同接口、相同游戏状态、相同动作集合与步数预算游玩^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]，把「发现」从叙事能力还原为决策能力。配合 7 个难度 tier 的分层（tier 1 最易、tier 7 最难，人类首试皆可解而最佳模型在顶层受阻），tier 曲线本身成为诊断证据：模型是在假设生成、实验设计还是规则应用环节失败，可沿难度轴定位短板。胜率定义（单游戏先对 runs 取均值、再对 tier 的 10 个游戏取均值）则把随机探索噪声与真实能力差异区分开。

### 奖励可验证性与探索预算的平衡

dig.bench 的奖励信号是游戏可通关性：通关与否二值且完全可验证，无需人工评分或模型裁判——环境即裁判，这与 [[concepts/rlvr-reinforcement-learning-verified-reasoning|可验证奖励强化学习]] 的思路同构，也解释了为何该基准不需要昂贵的评估基础设施。**步数预算与 creative mode 是对实验经济学的显式建模**：胜负由有限步数判定，部分游戏允许进入不计步数的 creative mode 关卡自由实验^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]，把「安全探索」与「承诺行动」的分离变成机制：高分奖励先在低代价环境试错、再回到计分环境验证假设的智能体，即真实科研中先导实验与正式实验的分工。这一结构与 [[entities/sciagentgym-benchmark-multi-step-scientific-tool-use|SciAgentGym]] 等多步科学工具基准共同指向一条评测设计原则：探索的代价必须计入得分，否则基准奖励的是穷举而非发现。

### 双层 harness 测法与残余局限

**双层测法分离模型与 harness 的贡献**：同一组游戏按 Basic Harness（裸模型）与 Agentic Harness（完整编排框架）两个 tier 集合分别计分^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]，使「增益来自模型还是脚手架」成为可观测对比而非争论；这与 [[concepts/evaluation-harness-design|评测 harness 设计]] 的核心关切一致。局限同样值得记录。其一，70 局仅公开 21 局，隐藏的 49 局不可由外部复现，独立验证存在盲区；其二，「发现」被限定为设计者预置的封闭规则系统——奖励的可验证性恰恰来自规则的封闭性，它能干净地测出封闭世界内的实验推理，却无法衡量真实科研中开放式的问题定义与真值不明场景；其三，通关这一单一二值信号压缩了发现的中间质量（实验方法是否优雅、规则是否被真正理解还是偶然碰到），与 [[concepts/agent-evaluation-benchmark-frameworks|智能体评测基准框架]] 中多维评估的思路形成对照——解读 dig.bench 分数时应当作能力下界而非全面画像。封闭规则发现与开放科研发现之间的桥接，可参考 [[entities/science-discovery-tree-search-rsi-research-code-2026|科学发现树搜索]] 等面向真实研究代码的工作。

## 实践启示

1. **用载体约束隔离被测能力**：剥离感知混淆后，「输在哪一环」才有答案；自建 eval 应先想清楚测的到底是哪种单一能力。
2. **用人类首试可解校验任务合法性**：任务若对人类在无先验、有限资源下不可行，基准就无法区分「任务真难」与「基准坏了」。
3. **模型层与 harness 层分开计分**：同一任务集在裸模型与完整框架下各测一遍，引入 agent 框架前先建立裸模型对照，增益归因才有依据。
4. **把探索预算管理当作一等能力**：先低成本试错、再消耗正式预算的两阶段行为可直接转化为得分，训练与评估时应显式奖励。
5. **公开子集 + SDK 降低采纳门槛**：70 局公开 21 局并提供 API 文档^[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark.md]，外部研究者无需全量数据即可对齐排行榜，基准影响力与可复现基础设施成正比。
6. **优先选「环境即裁判」的可验证奖励**：通关与否二值自校验，无需人工评分或模型裁判；自建 eval 应优先寻找这类自校验信号，把评估基础设施成本压到最低。
7. **发现必须落地为行为而非陈述**：仅「说出规则」不得分，规则要被用于解决挑战并限时通关；自建评测同样应要求假设被执行验证，区分「嘴上会」与「真会用」。
8. **保留一个永不公开的 holdout 层级**：21/70 公开、49 局隐藏的做法区分了「背题」与「真会发现」；自建基准至少留一层私有变体用于最终判定，公开层只作社区对齐。


## 意义

dig.bench 把「智能体实验/发现能力」从通用 QA 基准中独立出来，与 [[entities/autoresearch-ai-scientific-discovery-l0-l4-challengehub|AutoResearch]] 同属 AI 科学发现评估前沿。其「基础模型 vs Agentic Harness」双层测法为 [[entities/agent-evaluation-systematic-guide-metrics-to-closed-loop|Agent 评测]] 提供了实验驱动的基准范式。

→ [[raw/articles/digbench-scientific-discovery-text-games-agent-benchmark|原文存档]]
