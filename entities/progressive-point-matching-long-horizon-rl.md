---
title: "Progressive Point Matching (PPM): Long-Horizon LLM RL"
created: 2026-09-10
updated: 2026-09-10
type: entity
tags: [rl, reinforcement-learning, long-horizon, credit-assignment, post-training, berkeley, arxiv]
sources: [raw/articles/progressive-point-matching-long-horizon-rl]
confidence: 0.75
---

# Progressive Point Matching (Long-Horizon LLM RL)

Berkeley 团队（Preston Fu, Kevin Frans, Oleh Rybkin, Sergey Levine, Aviral Kumar，2026-09）提出 Progressive Point Matching (PPM)：一个简单且渐近无偏的 partial-credit 框架，用于长程 LLM RL 训练（任务轨迹可长达数百万 token）。论文与代码均已发布。^[raw/articles/progressive-point-matching-long-horizon-rl.md]

## 问题：稀疏 outcome reward 随 horizon 指数失效

标准 RL 做法是对完整轨迹采样并给 0/1 稀疏结果奖励。文章理论上证明：稀疏结果奖励产生的 policy gradient 信噪比随任务 horizon **指数衰减**——一条在几十个子任务上有进展但最后一步失败的轨迹，与毫无进展的轨迹拿到同样奖励。既有方案（learned value functions、process rewards、self-distillation）都引入渐近 bias：surrogate objective 下的最优策略可能不是 outcome reward 下的最优策略（如 process reward 奖励"逻辑正确但与最终成功无关"的语句）。^[raw/articles/progressive-point-matching-long-horizon-rl.md]

## 核心机制

关键洞见：推理问题可看作在 Markovian state space 中寻路。长推理链可压缩为中间结果（reasoning points，如定理证明中已证明的 lemma），后续推理只需条件化在这些点上。状态 = 当前轨迹前缀访问过的 reasoning point 集合 → reasoning MDP，progress（集合大小）永不回退。

朴素优化 progress 有偏：与参考轨迹（如人工证明）同策略的成功轨迹 progress 低，紧跟参考轨迹的失败轨迹反而高。PPM 用 **shortcutting** 机制修正：某点依赖的所有点都到达后该点才算到达——任何成功轨迹获得全额 credit，且允许学到偏离参考轨迹的新策略（对提升 pass@k 关键）。论文证明 shortcutting 学到的策略在 outcome reward 下最优（无偏）。^[raw/articles/progressive-point-matching-long-horizon-rl.md]

## 实验结果

合成受控实验显示相对 GRPO 指数级加速收敛；在 hard-math 任务上有强结果。reasoning points 从参考轨迹（如人工 proof）提取成本低，不需要完整 LLM reasoning traces。^[raw/articles/progressive-point-matching-long-horizon-rl.md]

## 关联

- 长程 agentic RL 实践：[[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Long-Horizon Agentic RL Frameworks]]、[[entities/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528|Agent Lightning (Agentic RL)]]
- RL 算法谱系：[[entities/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|LLM RL 算法演进]]、[[entities/aws-grpo-rlvr-sagemaker-math-reasoning|GRPO/RLVR]]
- 概念层：[[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]]、[[concepts/reinforcement-fine-tuning-rft|RFT]]

→ [[raw/articles/progressive-point-matching-long-horizon-rl|原文存档]]
