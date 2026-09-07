---
title: UnityMAS-O
created: 2026-07-15
updated: 2026-09-07
type: entity
tags: [multi-agent, rl, reinforcement-learning, open-source, paper]
sources: [raw/articles/unitymas-o-multi-agent-rl-optimization-framework-2026]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# UnityMAS-O

UnityMAS-O 是一个面向通用多智能体系统（Multi-Agent System, MAS）的强化学习优化框架，由中国人民大学与小红书联合开源。它的核心定位是让用户能够对 MAS 内所有 Agent 进行联合 RL 优化，而不再局限于提示词工程和手工编排。用户可自由定义角色、工作流、模型映射和奖励函数，系统将这些自定义要素转化为可执行、可归因、可训练的多智能体强化学习问题。^[raw/articles/unitymas-o-multi-agent-rl-optimization-framework-2026.md]

该框架的核心创新在于将"逻辑角色"与"物理参数"解耦。用户可为规划、检索、验证等不同角色分配独立的 LLM 映射（role-LLM mapping），定义静态或动态工作流（包括顺序、并行、分支、循环等图结构），并在节点级、回合级或完整轨迹级设定奖励函数。这种设计使得同一个 MAS 可以在全参数共享、部分共享和全独立参数之间自由切换，无需重写工作流本身。^[raw/articles/unitymas-o-multi-agent-rl-optimization-framework-2026.md]

UnityMAS-O 与传统的提示词工程方法有本质区别。传统方法依赖手工编排规则和单模型调优，而 UnityMAS-O 将整个多智能体系统视为一个可优化的整体，通过强化学习实现"工作流级优化"。系统采用中心控制器加模型本地 worker group 的星型架构，将轻量轨迹数据保留在控制器侧，而 log-prob、token buffer 等重数据留在模型本地，从而实现高效的异步 rollout、动态工作流调度和 PPO 风格策略更新。^[raw/articles/unitymas-o-multi-agent-rl-optimization-framework-2026.md]

实验验证方面，UnityMAS-O 在检索增强问答和反思式代码生成任务上展示了显著效果。在 Natural Questions 和 HotpotQA 基准上，多智能体 RL 训练后的验证 F1 均优于训练前，小模型收益尤其明显（如 0.5B agents 在 NQ 上从 0.022 提升到 0.445）。在代码生成任务中，3xQwen3-8B 的 held-out test all-passed rate 从 0.290 提升到 0.738，且无需消耗更多验证轮次。参数共享实验还表明，共享参数方案能以较小训练速度代价接近独立多模型方案的最终效果。^[raw/articles/unitymas-o-multi-agent-rl-optimization-framework-2026.md]

该项目已开源论文和代码（论文：arxiv.org/pdf/2605.26646，代码：github.com/chenyiqun/UnityMAS-O），为多智能体系统从"手工设计 agent"迈向"将整个 MAS 作为系统持续优化"提供了可复用的基础底座。^[raw/articles/unitymas-o-multi-agent-rl-optimization-framework-2026.md]

→ [[raw/articles/unitymas-o-multi-agent-rl-optimization-framework-2026|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

