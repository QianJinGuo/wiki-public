---
title: "Can AI automate Epoch?"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [agent, ai-evals, research-automation, benchmark]
sources: [raw/articles/can-ai-automate-epoch-agent-case-study]
confidence: 0.75
---

# Can AI automate Epoch?（GPT-6 Astra 自动化研究分析案例）

## 概述

Epoch AI 的自指实验：让 GPT-6 Astra agent 自动化 Epoch 自身的研究分析工作流，并复盘其中暴露的实验设计缺陷。该案例的价值不在"agent 能不能自动化研究"，而在它给出一个具体、可复现的 agent 跑实验翻车样本：**agent 自己设计的实验有 setup flaw，agent 自己发现并修复了它，但在 write-up 中把修复效应错误归因为核心科学结论**。^[raw/articles/can-ai-automate-epoch-agent-case-study.md]

## 关键细节：pilot 实验的 token budget 缺陷

Astra 设计的 pilot 实验测模型能否通过有限次输入试错猜出隐藏规则（经典 2-4-6 范式变体），对比"自选输入"vs"给定信息性输入"两组条件。缺陷：模型输出预算只有 4096 tokens，导致 280 个响应中 61 个在说出输入之前就被截断，且这些截断响应仍被计为有效尝试——被测模型被迫在信息不足下猜测规则，测的不是"证据获取能力"而是"输出预算容忍度"。Astra 自己识别出该错误并提高 token budget，各被测模型成绩显著回升。^[raw/articles/can-ai-automate-epoch-agent-case-study.md]

## 核心洞察：修复 ≠ 解释权

提升预算后的结论本应是"4096-token 输出预算不足以引出完整响应，这与核心研究问题无关"。但 Astra 在 write-up 中把该修复包装成被测模型"sensitivity to the acquisition budget"（acquisition budget 指模型选输入时的 token 预算）——把 harness 工程缺陷的修复效应误归因为被测系统的科学属性。这是 agent 自动化研究的一个通用失败模式：**实验修复发生在循环内，解释归因发生在循环外，两者脱节时 agent 会产出看起来自洽但结论错位的报告**。^[raw/articles/can-ai-automate-epoch-agent-case-study.md]

## 关联

- [[concepts/scientific-method-ai-research|AI 研究的科学方法]]
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]
- [[entities/ai-evals-methodology|AI Evals 方法论]]
- [[entities/arbor-hypothesis-tree-research-agent-2026|Arbor 研究假设树 agent]]

→ [[raw/articles/can-ai-automate-epoch-agent-case-study|原文存档]]
