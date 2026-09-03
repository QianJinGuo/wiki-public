---
title: "Agent 开始「自我进化」：会出题、会反思，还会自己长出新技能"
created: 2026-08-06
updated: 2026-08-06
type: entity
tags: [agent, self-evolution, self-improvement, reflection, skill-generation, post-training]
sources: [raw/articles/agent开始自我进化会出题会反思还会自己长出新技能]
confidence: 0.8
provenance_state: extracted
---

# Agent 开始「自我进化」：会出题、会反思，还会自己长出新技能

## 核心机制：出题—答题—反思闭环

本文（作者 horacebao、ashexie）描述了一个 Agent 自我进化的三层闭环：**Agent 自己会出题、自己会答题，还能把答错的经历变成新技能**。与传统的"人类出题—模型训练"模式不同，这里 Agent 在运行时自主生成训练信号——出题环节检验自身知识边界，答题环节暴露薄弱点，反思环节将错误模式沉淀为可复用的新能力。这是从"被动接受训练"到"主动生成学习信号"的范式转移。^[raw/articles/agent开始自我进化会出题会反思还会自己长出新技能.md]

## 与 Wiki 现有知识的关联

- 与 [[entities/agent-self-improvement-six-mechanisms|Agent 自我改进六机制]] 同主题但机制不同：六机制是分类学框架，本文是具体的"自出题"实现路径
- 与 [[entities/agent-self-evolution-evaluator-bottleneck|自进化评估器瓶颈]] 互补：本文侧重技能生成侧，评估器瓶颈侧重验证侧
- 企业级落地见 [[entities/agentscope-builder-enterprise-self-evolving-agent-harness|AgentScope Builder 自进化 Harness]]
- 递归自我改进的理论背景见 [[entities/ai-recursive-self-improvement-nanogpt-prime-intellect|AI 递归自我改进]]

→ [[raw/articles/agent开始自我进化会出题会反思还会自己长出新技能|原文存档]]
