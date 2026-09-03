---
title: "GRPO 群组相对策略优化"
created: 2026-06-30
updated: 2026-08-01
type: concept
tags: [concept, rl, grpo, reinforcement-learning, training, llm, deepseek]
sources: []
---

## 定义

GRPO（Group Relative Policy Optimization）是一种面向 LLM 训练的强化学习算法，由 DeepSeek 团队在 DeepSeek-Math（2024）中提出。GRPO 的核心创新是**去掉 critic/value model**，用同一批次内多个采样结果的相对奖励作为 baseline，直接用 group-level 归一化的优势估计来更新策略。相比 PPO 需要训练一个与策略模型同等规模的 value model，GRPO 显著降低了训练的内存和计算开销。

## 核心范式

- **无 Critic**：去掉 PPO 中的 value model，用 group 内采样结果的均值奖励作为 baseline
- **Group 采样**：对同一 prompt 生成 G 个候选回答（通常 G=8-64），计算每个回答的奖励
- **相对优势**：优势 = (个体奖励 - 组均值) / 组标准差，天然支持对比学习
- **KL 惩罚**：与参考策略的 KL 散度约束防止策略退化，保持生成多样性
- **兼容 Verifiable Reward**：GRPO 天然适合有明确正确性判断的任务（数学、代码），奖励信号清晰

## 背景与提出

GRPO 由 DeepSeek 团队在 2024 年的 DeepSeek-Math 论文中首次提出，后在 DeepSeek-R1（2025）中成为核心训练方法。动机源于 PPO 在 LLM 训练中的实际困难：训练一个与策略模型同等规模的 critic model 需要巨大的 GPU 内存，且 critic 本身的训练不稳定会传导到策略更新中。

DeepSeek-R1 的成功证明了 GRPO 在大规模 LLM 训练中的有效性：模型在数学推理和代码生成任务上取得了显著提升，且训练成本比 PPO 方案低约 40%。GRPO 随后被广泛采用于 RLHF/RLVR 训练流程中。

## 局限与反对声音

- **奖励稀疏性**：group 内所有采样共享同一 prompt，如果奖励信号太稀疏（全部回答得 0 分），group 归一化无效
- **采样成本**：每个 prompt 需要生成 G 个候选回答，推理成本是 PPO 的 G 倍
- **仅适用于可验证任务**：对于开放式对话、创意写作等缺乏明确奖励信号的任务，GRPO 优势不明显
- **Group 大小敏感**：G 太小归一化不稳定，G 太大推理成本过高，需要仔细调参

## 实践启示

1. **数学/代码任务首选**：GRPO 在有 verifiable reward 的场景下效果最佳，是 RLVR 训练的默认选择
2. **Group 大小**：G=8-16 是常见起点，GPU 内存允许时增大到 32-64 可提升稳定性
3. **奖励设计**：二值奖励（对/错）+ 格式奖励（是否遵循输出格式）组合效果好
4. **与 SFT 配合**：先 SFT 建立基础能力，再 GRPO 微调——不要从 base model 直接 RL
5. **参考策略更新**：每隔 N 步更新一次参考策略，避免 KL 散度计算的参考模型过时

## 相关实体

- [[concepts/rlvr-reinforcement-learning-verified-reasoning]] — GRPO 是 RLVR 的核心算法之一
- [[entities/deepseek-v3-moe-architecture]] — DeepSeek 系列模型广泛使用 GRPO 训练
- [[concepts/moe-mixture-of-experts-2025]] — MoE 架构降低 GRPO 训练的推理成本
- [[entities/rubrics-survey-llm-evaluation-ruc-nlpir-2026]] — 评估方法与 GRPO 奖励设计相关

## 所属 MOC

- [[moc/llm-core-technology|Llm Core Technology]]
