---
title: "ReViSQL: Task Expertise into RL Achieves Human-Level Text-to-SQL"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: [text-to-sql, reinforcement-learning, fine-tuning, rlvr, database, benchmark]
sources: [raw/articles/revsql-task-expertise-rl-text-to-sql]
confidence: 0.8
---

# ReViSQL: Task Expertise into RL Achieves Human-Level Text-to-SQL

> **Background**：Thinking Machines Lab联合UIUC和Bridgewater AIA Labs的研究，通过RLVR（Reinforcement Learning with Verifiable Rewards）微调模型ReViSQL-K2.6，在BIRD text-to-SQL benchmark上达到人类水平准确率（92.96%）。

## 问题背景

人类在BIRD benchmark上得分92.96%，但LLM从2024年的~70%到目前最高82%。GPT-5.6 Sol Ultra和Claude Fable 5在mid-80s，但成本过高不适合高容量应用。^[raw/articles/revsql-task-expertise-rl-text-to-sql.md]

**核心洞察**：人类通过重复实践获得技能，而不是通过接收指令列表——LLM也应该如此。任务经验应该用来训练更好的推理能力，而不是仅仅更新scaffold的prompt。

## 方法创新

标准text-to-SQL scaffold（schema linking → generation → self-correction → selection）的问题：模型固定，通过增加调用数量和结构来提升性能，但仍有11个百分点的人类差距。

ReViSQL的两个关键改进：

1. **专家验证的训练集**：清除可能毒化RLVR的标签错误
2. **Reward shaping**：针对RLVR在该领域两个常见失败模式的奖励塑形技术

## 成果

- ReViSQL-K2.6在SC-16（16个样本自一致性选择）下超过人类92.96%基准
- 无需scaffolding——直接模型推理
- 使用Tinker平台进行RL训练

## 实践启示

- Text-to-SQL的瓶颈不在训练数据（SQL在预训练中广泛存在），而在处理歧义问题和高上下文schema的能力
- RLVR在可验证任务（SQL执行结果可验证）上的有效性
- 专家验证训练集对RLVR性能的关键影响——标签错误会直接毒化奖励信号
- Scaffolding vs end-to-end training的范式之争：增加调用结构 vs 训练更强的内化推理

→ [[raw/articles/revsql-task-expertise-rl-text-to-sql|原文存档]]
