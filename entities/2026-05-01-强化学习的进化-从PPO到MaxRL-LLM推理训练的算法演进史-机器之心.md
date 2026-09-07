---
title: "强化学习的进化 从PPO到MaxRL LLM推理训练的算法演进史 机器之心"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心.md|原文存档]]

sha256: d1146eef9fdadc0f26576697e604241e2be8d1b39eac6cd90767fdc6e5d70ec2 ^[raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心.md]

## 摘要

本文系统梳理了 2024 至 2026 年用于推理 LLM 的强化学习算法演进，从 REINFORCE 与 PPO 讲起，覆盖 GRPO、RLOO、Dr. GRPO、DAPO、CISPO、MaxRL、DPPO、ScaleRL 等方法 ^[raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心.md]。核心叙事线是 PPO 的四组件（策略、推演策略、参考策略、价值模型）在 LLM 场景下被逐步瘦身：GRPO 用组内相对基线取代 critic 模型，节省约 50% 内存成为大规模推理 RL 的关键 ^[raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心.md]。文章指出几个反复出现的模式：标准差归一化会让模型过度关注几乎已解决的提示词；样本级损失均值会引入偏向冗长错误回复的偏置（Dr. GRPO 与 DAPO 的 token 级聚合修正）；信任域（裁剪机制）是最佳优化切入点——DAPO 非对称放宽、CISPO 只裁剪重要性权重保留全部梯度流、DPPO 用策略散度取代比例掩码 ^[raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心.md]。MaxRL 则从另一视角重构目标：证明最大似然梯度是 pass@k 梯度的无限调和混合，把 RL 重建为不可微采样条件下的近似最大似然训练，实证上提升 pass@k 并更好保留输出多样性 ^[raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心.md]。文末总结了信用分配、样本效率、数学代码之外泛化等开放挑战，并引用 ScaleRL 超过 40 万 GPU 小时的消融说明该领域实证结果的适用范围可能远小于表面看起来的程度 ^[raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心.md]。

## 关键要点

- GRPO（出自 DeepSeekMath、由 DeepSeek-R1 发扬）的成功关键非常朴素：移除 critic 模型，用同提示词下 G 个采样回复的组内归一化做优势基线，大幅减少内存占用
- Dr. GRPO 的两个修正：损失除以固定常量（最大 token 数）而非序列长度，消除"奖励均摊让长错误回复惩罚变弱"的冗长偏置；不再除以奖励标准差，避免低方差提示词上产生极不相称的巨大更新
- DAPO 四项改进：token 级聚合、非对称裁剪（上界 0.28、下界 0.2，让罕见 token 有更大上升空间）、超长奖励软惩罚区域、动态采样（全对/全错提示词持续重采直到出现正负混合）
- CISPO（MiniMax-M1 提出）针对"However、Wait、Aha"等低概率但关键的推理分叉词：只对重要性采样权重做 stop-gradient 裁剪而不硬掩码梯度，单步训练效率比 DAPO 提速两倍
- ScaleRL 发现：异步流水线式 RL 优于先生成后更新；CISPO 和 GSPO 渐近性能优于 DAPO；FP32 精度计算语言模型头可显著缓解训练与推理框架数值不匹配扭曲 IS 比例的问题

## 来源

- 原文：[[raw/articles/2026-05-01-强化学习的进化-从PPO到MaxRL-LLM推理训练的算法演进史-机器之心.md|强化学习的进化 从PPO到MaxRL LLM推理训练的算法演进史 机器之心]]
