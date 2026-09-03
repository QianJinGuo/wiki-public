---
title: "RLVR 基于可验证奖励的强化学习"
created: 2026-06-30
updated: 2026-08-01
type: concept
tags: [concept, rl, rlvr, reinforcement-learning, training, llm, reasoning]
sources: []
---

## 定义

RLVR（Reinforcement Learning with Verifiable Rewards）是一种 LLM 训练范式，使用可程序化验证的奖励信号（而非人类偏好标注）来训练模型的推理能力。核心思路：对于数学、代码、逻辑推理等有明确正确答案的任务，用程序自动判断模型输出是否正确，将判断结果作为 RL 训练的奖励信号。RLVR 是 DeepSeek-R1、Qwen-2.5-Math 等推理模型的核心训练方法。

## 核心范式

- **可验证奖励**：奖励信号来自程序化验证（数学答案对错、代码是否通过测试），不需要人类标注
- **自我进化**：模型通过大量采样 + 验证 + RL 更新循环，逐步学会更长的推理链（chain-of-thought）
- **涌现推理**：RLVR 训练的模型会自发涌现"自我纠错""回溯探索"等高级推理行为，无需显式标注
- **GRPO/DPO 为核心算法**：GRPO（无 critic）和 DPO（直接偏好优化）是 RLVR 最常用的 RL 算法
- **冷启动问题**：base model 需要先通过 SFT 学会基本的指令遵循和输出格式，再进入 RLVR 阶段

## 背景与提出

RLVR 的概念在 2024-2025 年随着 DeepSeek-R1、OpenAI o1 等推理模型的兴起而被广泛讨论。传统 RLHF 依赖人类偏好标注（"回答 A 比回答 B 好"），标注成本高且主观性强。RLVR 的关键洞察是：对于有标准答案的任务，程序化验证比人类标注更便宜、更一致、更可扩展。

DeepSeek-R1（2025）是 RLVR 的里程碑：模型仅通过 RL 训练（无额外人类标注）就在数学和代码推理上达到了 GPT-4 级别。更令人惊讶的是，模型在训练过程中自发涌现了长 chain-of-thought、自我反思、回溯探索等行为——这些推理策略从未被显式教授，纯粹通过 RL 探索发现。

## 局限与反对声音

- **任务范围受限**：仅适用于有可验证答案的任务，开放式对话、创意写作、主观判断等场景无法使用
- **奖励黑客（Reward Hacking）**：模型可能学会绕过验证器的"捷径"（如格式化输出骗过正则匹配）
- **训练不稳定**：RL 训练本身不稳定，加上大规模采样，超参敏感度高
- **涌现不可控**：自发涌现的推理策略有时是低效的（如冗长的 self-talk），需要额外约束

## 实践启示

1. **奖励设计是核心**：验证器的准确性直接决定模型质量——宁可漏判也不要误判
2. **SFT 冷启动**：不要从 base model 直接 RLVR，先用高质量 SFT 数据教会模型输出格式
3. **渐进式训练**：从简单任务（单步推理）到复杂任务（多步推理）逐步增加难度
4. **防 reward hacking**：用多个验证器交叉验证，定期审查模型的"成功"输出是否真正正确
5. **与 RLHF 互补**：RLVR 处理可验证任务，RLHF 处理主观任务，两者在训练流程中交替使用

## 相关实体

- [[concepts/grpo-policy-optimization-2026]] — GRPO 是 RLVR 的核心训练算法
- [[concepts/transformer-architecture-2025]] — Transformer 架构是 RLVR 训练的基础模型
- [[entities/deepseek-v3-moe-architecture]] — DeepSeek 系列是 RLVR 的代表性成果
- [[entities/rubrics-survey-llm-evaluation-ruc-nlpir-2026]] — LLM 评估方法与 RLVR 奖励设计相关

## 所属 MOC

- [[moc/llm-core-technology|Llm Core Technology]]
