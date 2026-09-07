---

title: "OPD 重新审视失败模式与简单修复"
created: 2026-06-10
updated: 2026-09-07
tags: [agent, code, fine-tuning, llm, memory, mlops, observability, open-source, prompt, rl]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/opd-revisiting-failure-modes-simple-fixes-storm
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Opd Revisiting Failure Modes Simple Fixes Storm

→ [[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm|原文存档]] ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]

## 深度分析

Opd Revisiting Failure Modes Simple Fixes Storm ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
### 核心观点
1. 大模型智能｜分享 ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
来源 | 知乎 ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
作者 | storm ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
在最近的大模型后训练中，On-Policy Distillation已经成为默认选项之一。 ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
2. 但研究者们在分析训练日志、实验曲线和对比不同 OPD 方法实现时，反复碰到同一个问题：理论上很自然的 sampled-token OPD，实际运行起来并不稳定，甚至会把模型往一些局部上“看起来合理”、整体上却越来越差的方向推。 ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
3. 论文:Revisiting On-Policy Distillation: Empirical Failure Modes and Simple Fixes ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
链接:https://arxiv. ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
4. 25562 ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
代码:https://github. ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
5. com/hhh675597/revisiting_opd ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]
在这篇文章中，我们并不打算再次讲解 OPD (已经有很多不错的入门材料)，而是想集中回答三个更具体的问题：这个方法到底在优化什么；常见实现为什么容易出问题；以及有没有一个代价不高、但更稳定的实现路径。 ^[raw/articles/opd-revisiting-failure-modes-simple-fixes-storm.md]

### 关联实体

- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/两万字详解claude-code源码核心机制]]
- [[entities/scale-robot-reinforcement-learning-with-nvidia-isaac-lab-on-]]

## 相关实体

- [[moc/reinforcement-learning-rlhf|MOC]]
