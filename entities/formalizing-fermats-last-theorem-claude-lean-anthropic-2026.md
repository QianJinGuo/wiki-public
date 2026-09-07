---
title: "Formalizing Fermat's Last Theorem：Claude 用 Lean 端到端形式化证明费马大定理"
created: 2026-09-07
updated: 2026-09-07
type: entity
tags: [ai4math, formal-verification, lean, prove2me, multi-agent, anthropic, claude, theorem-proving]
sources: [raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026]
confidence: 0.9
---

# Formalizing Fermat's Last Theorem：Claude 用 Lean 端到端形式化证明费马大定理

Anthropic 于 2026-09-04 宣布首个完整的计算机可校验的费马大定理（FLT）证明：Claude 在 11 天内、以高度自主的方式，用 **Lean** 证明助手写完了 Wiles 证明的形式化版本。这是 AI 自动形式化（autoformalization）领域的一个里程碑式成果。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 背景：为什么 FLT 如此难形式化

费马在 1637 年于《算术》页边写下"我发现了真正绝妙的证明，但页边太窄写不下"，其后 350 余年无人证出。1908 年悬赏 10 万德国金马克，仅第一年就出现 621 份错误尝试。1993 年 Wiles 三场讲座后被发现存在关键缺口，又与 Taylor 苦战一年才在 1995 年发表正确证明——篇幅 129 页，依赖远超 1637 年时代的现代数学技术。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

形式化（把数学推理转成计算机可自动校验的形式）是一项公认难题：人类证明会跳过大量显然步骤，而 Lean 需要看到每一步；且形式化只能从已被形式化的少数数学出发，无法直接借力几百年的出版物积累。仅社区项目（2024 年 Kevin Buzzard 在帝国理工发起的 multi-year 努力）的形式化 blueprint 就有 86 页，原预计整个形式化需数年。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 关键成果与技术数据

Claude 用 11 天完成 FLT 形式化，沿途产出 30,300 个定理（最终证明用到 29,500 个），共 13 百万行 Lean 代码——是 Mathlib（社区主要数学定理库）规模的 5 倍以上，也是迄今构建的最大 Lean 证明。约 6 billion output tokens 消耗于一个接近 Claude Fable 5.1 的通用内部研究模型。证明只用了 Lean 的三个标准公理，并经 comparator 确认定理表述与 Mathlib 对 FLT 的表述一致。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

Claude 的证明遵循 Darmon-Diamond-Taylor 对 Wiles 证明的简化版本。人类输入仅限 Tianyi Peng（Anthropic 研究员，哥伦比亚大学课题组）偶尔的高层指令，如"Jacobian as a scheme 优先级高""尽快推 Mazur 定理"。多位 Claude agent 分工协作定义概念、证明中间定理、再用这些定理证明更难命题。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 为什么能成功：Prove2Me 平台与多 Agent Harness

值得注意：Claude 的数次初始尝试都失败了——agent 虽有早期成功，但很快丢失项目状态、停止有效协作，失败尝试贡献了最终证明中约 7% 的非样板代码行。**转用 Prove2Me 后才成功**。Prove2Me 是 Peng 与其哥伦比亚合作者设计的开放协作形式化平台，其设计要点：^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

- **定理语句 DAG**：维护定理陈述的有向无环图（DAG），供 agent 决定下一步证明什么，缓解记忆退化（memory degradation）并允许多 agent 并行。
- **编译提速**：把定理陈述与证明拆分到不同文件、独立维护链接，加速 Lean 编译、最小化资源消耗。
- **搜索复用**：为每个定理维护自然语言描述，简化证明路径。

配合基于 Claude Code 的多 Agent harness，一个 agent 团队在不到两周内完成证明（11 天主体工作）。这与 [[entities/claude-code-multi-agent-harness-source-analysis|Claude Code Multi-Agent Harness]] 的多 agent 编排模式高度相关——DAG 化任务分配是这群 agent 不"跑偏"的关键。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 对形式化验证与 AI 数学的意义

Anthropic 与评论者（Kevin Buzzard）认为这证明"自动形式化当代数学文献已成为可能"——可用来找出数学共同语料中的错误、减轻审稿/referee 负担，并以可校验方式核查 LLM 生成的数学。形式化也被视为人类对 AI 生成数学结果建立信心的主要途径：AI 产出越来越多"声称的证明"后，形式化可能是数学界跟上的唯一可行方式。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

一个低成本信号：用三个个人 Claude Max 订阅、完全经 Prove2Me 协作，agent 在三天内完成了 Vinogradov 三素数定理的形式化——说明在合适 scaffold 下，用消费级 AI 订阅协作形式化重大结果可达。这也呼应 [[entities/lean-scaling|Lean 软件扩展定律]] 所关注的形式化规模与成本问题。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 相关实体

- [[entities/lean-scaling|Lean Software Scaling Laws]]
- [[entities/formatheoria-cfsg-ai-formal-verification-lean-2026|FormaTheoria：AI 辅助有限单群分类形式化]]
- [[entities/agent-formal-verification-ai-code|Agent 形式验证]]
- [[entities/claude-code-multi-agent-harness-source-analysis|Claude Code Multi-Agent Harness]]

→ [[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026|原文存档]]
