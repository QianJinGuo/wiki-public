---
title: "组织级第二大脑：构建从专家处学习的 AI（agent 反馈归因）"
created: 2026-09-05
updated: 2026-09-05
type: entity
tags: [agent, feedback, learning, expert-knowledge, attribution, meta]
sources: [raw/articles/facebook-org-second-brain-agent-learns-from-experts-2026]
confidence: 0.65
---

# 组织级第二大脑：构建从专家处学习的 AI（agent 反馈归因）

> Meta Engineering 介绍如何构建「组织级第二大脑」——一个从领域专家处学习的 AI。核心问题：如何把专家的原始对话反馈转化为可诊断、可执行的 agent 学习信号。^[raw/articles/facebook-org-second-brain-agent-learns-from-experts-2026.md]

## 反馈归因问题

原始专家反馈来自对话 traces——领域 SME 与 agent 交互并提供纠正。诊断阶段需要从这些对话中提取结构化信号。^[raw/articles/facebook-org-second-brain-agent-learns-from-experts-2026.md]

## 两种归因方法的演进

**第一种（失败）**：按对话形式分类反馈——若专家提供信息则是「知识缺口」，若重定向 agent 则是「流程问题」。这个启发式失败，因为**对话形式是根因的差代理**：专家纠正一个结论，可能暴露的是知识缺口、配方缺陷、或真正的歧义。^[raw/articles/facebook-org-second-brain-agent-learns-from-experts-2026.md]

**可用方法（拆分提取与分类）**：^[raw/articles/facebook-org-second-brain-agent-learns-from-experts-2026.md]

1. 先提取专家携带的每个实质性信号 + agent 的完整知识 manifest（加载了哪些文件、何时、如何使用）
2. 再读实际知识文件，应用单个归因测试：**「agent 能否从其源材料得到正确结论？」**
3. 分类：
   - 材料含正确答案但 agent 仍出错 → **配方问题（recipe problem）**
   - 材料不含正确答案 → **知识缺口（knowledge gap）**
   - 专家彼此对正确答案有分歧 → **歧义（ambiguity）**，标记交人类讨论

## 可迁移洞察

「从对话形式判断根因」在 agent 反馈学习中是普遍陷阱——形式（信息 vs 重定向）与根因（知识缺口 vs 配方 vs 歧义）不是一一对应。**归因测试（attribution test）：检查 agent 源材料是否包含正确答案**，是比对话形式更可靠的分诊信号，可迁移到任何基于专家反馈学习的 agent 系统。^[raw/articles/facebook-org-second-brain-agent-learns-from-experts-2026.md]

## 相关

- [[concepts/agent-self-improvement-loops|Agent 自我改进循环]]
- [[concepts/agent-memory-architecture|Agent 记忆架构]]
- [[entities/karpathy-llm-wiki-second-brain-awkthole|第二大脑：知识库]]

→ [[raw/articles/facebook-org-second-brain-agent-learns-from-experts-2026|原文存档]]