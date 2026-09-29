---

title: "A from-scratch derivation of REINFORCE for language models"
created: 2026-09-29
updated: 2026-09-29
type: entity
tags: ["reinforcement-learning", "policy-gradient", "reinforce", "rlvr", "grpo", "training", "tutorial-deep"]
sources: [raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero]
confidence: 0.7
---

# REINFORCE 从零推导（Tyler Romero）

## 定位

一篇数学推导型教学深文：**PPO 到 GRPO 等当代 LLM RL 算法都是 policy gradient 这一个思想的精化**，文章用单一 prompt（"What is 17 × 24?"）贯穿全程，从 next-token 概率分布一步步推导出使正确答案概率上升的梯度。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md]

## 推导主线

1. **目标函数**：LLM 生成即一段 episode——每个 token 是一个 action，奖励只在结尾由 verifier 给出（RLVR：数学题有标准答案、代码任务有单测），总奖励即最终奖励。目标是最大化期望奖励 J(θ)。
2. **不可直接求导的两个障碍**：(a) 补全空间爆炸——15 万词表下 100-token 补全的枚举不可行；(b) 离散采样不可微——token ID 对参数无平滑梯度，且奖励来自 verifier/单测/人，无 autograd 路径。
3. **REINFORCE 技巧**：核心恒等式 `∇θ pθ = pθ ∇θ log pθ`（log-derivative trick），把期望的梯度改写为"采样补全上 ∇log pθ × R 的平均"——将不可微问题转化为可估计的采样平均。
4. **与 PPO/GRPO 的关系**：三者共享期望奖励目标，但 PPO/GRPO 还**改变更新本身**（clipping/reweighting 保持训练稳定）——这是"精化"而非"另起炉灶"。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md]

## wiki 中的定位

概念链路：[[concepts/rlvr-reinforcement-learning-verified-reasoning]]（RLVR 奖励面）→ 本文（policy gradient 数学面）→ [[concepts/grpo-policy-optimization-2026]] 与 [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]]（算法演化面）。对需要从第一性原理理解 RLHF/RLVR 训练的读者，这是比论文更可及的入口。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md]

## 关联

- → [[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero|原文存档]]
- [[concepts/rlvr-reinforcement-learning-verified-reasoning]] — RLVR 奖励函数与 verifier 概念面
- [[concepts/grpo-policy-optimization-2026]] — GRPO 对 REINFORCE 的精化
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]] — PPO/DPO/GRPO 算法演化谱系

