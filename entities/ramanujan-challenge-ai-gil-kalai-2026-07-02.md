---
title: "The Ramanujan Challenge for AI"
type: entity
tags: [ai-evaluation, benchmark, ai-mathematics, research, formal-verification, ai-reasoning]
created: 2026-07-04
updated: 2026-08-29
sources: [raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02]
---

# The Ramanujan Challenge for AI

## 摘要

吉尔·卡莱（Gil Kalai）分享了由伊多·卡米纳（Ido Kaminer）发起的"拉马努金 AI 挑战赛"（Ramanujan Challenge for AI），该挑战于 2026 年 7 月 2 日启动，将持续至 2026 年 8 月 1 日。挑战旨在系统测试 AI 系统在数学研究中的能力，特别是从具体公式出发生成有效证明或符号推导的能力。^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]

## 核心要点

- **十个研究级问题**：基于数学常数显式公式方向，专门设计用于测试 AI 的证明生成与符号推导能力^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]
- **多种提交形式**：接受形式化验证证明、计算机代数系统（CAS）推导、以及附带可复现代码的人可读证明^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]
- **评估从"答案"转向"过程"**：核心目标是检验 AI 能否以可结构化验证的方式生成推导过程，而非仅仅输出正确答案^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]
- **兼容现有工具链**：规则设计兼容形式化验证系统（如 Lean、Coq）、CAS 系统（如 Mathematica、SymPy）及可复现代码辅助证明

## 深度分析

### 数学 AI 评估的范式转变

拉马努金挑战代表了 AI 数学能力评估从"答案导向"向"过程导向"的深刻转变。传统的数学基准（如 MATH、GSM8K）主要测试 AI 对已知问题的答案生成能力，而本挑战要求 AI 从给定公式出发，构建完整的推导链条。这与当前 AI 评估领域从"输出正确性"向"推理可验证性"转变的大趋势一致。^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]


这种评估方式的转变触及了 AI 数学能力的本质：数学不仅仅是找到正确答案，更重要的是展示出可被同行验证的推理过程。挑战的规则设计——允许形式化证明、CAS 推导或可复现代码辅助证明——实际上是在测试 AI 是否具备"数学家的思维方式"，而不仅仅是"计算器的输出能力"。^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]


### 与形式化验证的深层关联

该挑战兼容形式化验证系统（如 Lean、Coq），这一设计选择极具前瞻性。近年来，[[entities/leap-agentic-formal-theorem-proving-google-2026|LEAN 在形式化定理证明]]领域的进展表明，AI 辅助的形式化数学正在成为现实。拉马努金挑战将形式化证明作为可接受的提交形式，实际上是在推动 AI 数学研究向"可机器验证"的方向发展——这不仅降低了人工审校的成本，也为 AI 生成的数学内容建立了一条质量控制基线。^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]


### 拉马努金机器项目背景

该挑战由拉马努金机器（Ramanujan Machine）项目发起。拉马努金机器项目是一个致力于利用 AI 发现新的数学常数公式的开源研究项目，其核心理念与拉马努金（Srinivasa Ramanujan）的直觉式数学发现精神一脉相承。该项目此前已在数学常数公式发现方面取得了多项成果，本挑战是其将研究重心从"公式发现"向"证明生成"拓展的重要一步。^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]


### 对 AI 推理能力评估的启示

拉马努金挑战的设计对更广泛的 AI 推理能力评估具有启示意义：^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]


1. **结构化验证**：通过要求可结构化验证的推导过程，挑战建立了一个可自动检查的评估框架，避免了传统评估中的人工主观偏差
2. **多路径求解**：允许形式化证明、CAS 推导、代码辅助证明等多种提交路径，反映了数学研究本身的多元化方法论
3. **时间压力**：一个月的挑战周期为 AI 系统提供了充足的多轮迭代空间，这与 METR、AISI 等机构对 AI 自主任务完成能力的长时间评估方法一致^[raw/articles/the-twilight-of-the-chatbots.md]

### 数学 AI 的未来方向

拉马努金挑战预示了数学 AI 的三大发展方向：从答案生成向证明生成演进、从封闭测试向开放挑战演进、以及从单一模态向形式化系统集成演进。这些方向与 [[concepts/harness-engineering-framework|Harness Engineering]] 的核心理念——系统性地评估和提升 AI 能力——高度一致。^[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02.md]


## 实践启示

1. **重视过程验证**：在评估 AI 的数学能力时，应优先关注推导过程的可验证性而非答案的正确性——前者才是衡量"理解"的真实标尺
2. **拥抱形式化工具**：将形式化验证系统（Lean、Coq 等）纳入 AI 数学研究的评估工具链，可以大幅降低结果验证成本并提高可靠性
3. **跨学科合作**：拉马努金挑战的跨学科性质（数学、AI、形式化验证）提示我们，最前沿的 AI 突破往往发生在学科交叉点
4. **时间维度的评估**：给 AI 系统充足的时间进行多轮尝试和迭代（如本挑战的一个月窗口），往往比单次评估更能反映真实能力——这一原则同样适用于 Agent 评估基准 设计
5. **从"发现"到"证明"**：AI 在数学领域的应用正从"公式发现"（如拉马努金机器项目的早期工作）向"定理证明"拓展，这一演进路径与 AI 从"模式匹配"向"推理理解"的整体进化趋势一致

## 相关实体

- [[entities/leap-agentic-formal-theorem-proving-google-2026|LEAP: Agentic Formal Theorem Proving]]
- [[entities/coda-bench-code-agent-data-benchmark-renmin-2026|CODA Bench: Code Agent Data Benchmark]]
- [[entities/karpathy-llm-wiki-obsidian-tutorial-shuge-2026|Karpathy LLM Wiki Knowledge Management]]

→ [[raw/articles/ramanujan-challenge-ai-gil-kalai-2026-07-02|原文存档]]
