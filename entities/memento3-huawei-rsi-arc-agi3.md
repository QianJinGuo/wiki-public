---

title: "Memento 3：华为 RSI 游戏智能体（ARC-AGI-3 通关）"
created: 2026-10-13
updated: 2026-10-13
type: entity
tags: [agent, rsi, self-evolving, game-agent, memory]
sources: [raw/articles/memento3-huawei-rsi-arc-agi3-2026-10-12]
confidence: 0.75
provenance_state: extracted
---

# Memento 3：华为 RSI 游戏智能体（ARC-AGI-3 通关）

## 系统概况

Memento 3 是华为诺亚方舟实验室联合 UCL 等团队提出的 AI 智能体系统，打通了 ARC-AGI-3 公开集的 **25 款游戏、183 个关卡**，以 Claude Opus 5 为底模，**全程不更新模型权重、不做游戏微调** → [[raw/articles/memento3-huawei-rsi-arc-agi3-2026-10-12|原文存档]]。 ^[raw/articles/memento3-huawei-rsi-arc-agi3-2026-10-12.md]

## 关键数字

- 人类完成同样关卡需 **17135 次操作**，Memento 3 只用 **7518 次——比人类少约 56%**，RHAE（Relative Human Action Efficiency）拿下满分 100
- 对比：同样的 Claude Opus 5 在 ARC Prize 官方标准框架下平均 RHAE 仅约 40.7——换一套智能体系统成绩提升 **59.3 分**
- Atari Pong：AI 自主摸索出控制策略后**不再向大模型请求推理**，连续三局 21:0

## 核心机制：Code-as-Model

系统边玩边给自己升级：自己上手玩、自己琢磨规则、自己写代码，玩错了反思修改。其核心是**迭代修正的规则手册 + 可执行世界模型**（Code-as-Model）——把学到的游戏规则沉淀为可执行的代码化模型，而非微调底模，与「从记忆到自进化 agent」的 RSI 路线一致。 ^[raw/articles/memento3-huawei-rsi-arc-agi3-2026-10-12.md]

## 与 Memento-Skills 的关系

Memento 3 是 Memento 系列的延续（前代 Memento-Skills 让 Agent 通过技能外部记忆持续进化），但本作的领域是**游戏型 RSI**（操作效率 + 规则自学习），与技能设计的侧重点不同，属同一自进化路线的两个分支。 ^[raw/articles/memento3-huawei-rsi-arc-agi3-2026-10-12.md]

## 关联

- → [[entities/memento-skills-agent-self-evolving|Memento-Skills（前代）]]
- → [[concepts/agent-memory-system-design|Agent 记忆系统设计]]
- → [[concepts/world-models|世界模型]]（Code-as-Model 的可执行世界模型路线）
