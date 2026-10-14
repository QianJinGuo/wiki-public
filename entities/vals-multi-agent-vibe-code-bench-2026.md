---
title: "Vals AI Multi-Agent Vibe Code Bench — 多智能体 vs 单智能体全栈编码基准"
created: 2026-10-13
updated: 2026-10-13
type: entity
tags: [multi-agent, evals, benchmark, coding-agent, vals-ai]
sources: [raw/articles/vals-multi-agent-vibe-code-bench-2026]
confidence: 0.7
---

# Vals AI Multi-Agent Vibe Code Bench

Vals AI 发布受控基准 + 统计显著性检验，比较多智能体团队与单智能体在全栈 web app 编码任务上的表现。核心数字：团队成本是单 agent 的 1.8-5.1 倍，而四个模型×强度对比中仅一个分数差异统计显著——Sol 的 medium-effort 团队比其单 agent 高 7.3 分（p=0.005），其余三个差异（−0.3 到 +3.4 分）均不显著^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md]。

成本-时间权衡出现反直觉形态：对 Sol 而言提高单 agent effort 比组团队收益更大——max-effort 单 agent 得分提升更多，成本只有 medium-effort 团队的 1/1.6（$4.82 vs $3.07/app），但耗时 3 倍；对 Opus 而言提高 effort 与组团队均无显著差异。团队只在 Sol 上更快完成，Opus 的团队耗时最高达单 agent 的 2.3 倍^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md]。

编排风格的质性差异：Sol 的 lead 在最初几分钟把 app 拆给并行 subagent 并亲自测试结果（max effort 下运行时从 38 分钟降到 28 分钟）；Opus 的 lead 通常是先写一份"app 各部分如何拼合"的共享契约，再按 builder/tester 波次串行委派。作者明确提示结论来自单一全栈 web app 基准，不一定迁移到其他工作类型^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md]。

对 wiki 的价值：这是"多智能体 ≠ 更好"命题的少数学术级量化证据之一（成本 1.8-5.1x、显著性 1/4），与 vals 自家 terminal-bench 作弊率研究同属其 agent 评测产品线。

## 关联

- [[entities/vals-ai-room-temperature-luttinger-magnet-agent-design|Vals AI Luttinger 磁体 agent 设计]]
- [[entities/vals-ai-cheating-on-the-rise-terminal-bench|Vals AI Terminal-Bench 作弊率研究]]
- [[entities/we-ran-250-ai-agent-evals-to-find-out-if-skills-beat-docs-th|250 次 Agent Evals 技能 vs 文档]]
- [[entities/ai-evals-methodology|AI Evals 方法论]]
- [[concepts/multi-agent-collaboration-patterns|多智能体协作模式]]

→ [[raw/articles/vals-multi-agent-vibe-code-bench-2026|原文存档]]
