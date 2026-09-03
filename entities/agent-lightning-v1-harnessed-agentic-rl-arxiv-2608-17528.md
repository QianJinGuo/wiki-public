---
title: "Agent Lightning v1.0：面向 Harnessed Agentic RL 的轻量框架"
created: 2026-08-20
updated: 2026-08-20
type: entity
tags: [agent, harness, agentic-rl, post-training, arxiv, framework]
sources: [raw/articles/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528]
confidence: 0.8
provenance_state: extracted
---

# Agent Lightning v1.0：面向 Harnessed Agentic RL 的轻量框架

## 概述

Agent Lightning v1.0（arXiv:2608.17528）提出了 **harnessed agentic RL**（受驾驭的智能体强化学习）范式：部署时的 agent harness 直接参与模型后训练，harness 而非训练引擎拥有环境交互循环，训练器只观察 LLM request-response 序列。该范式源自原版 Agent Lightning 提出的**解耦架构**——通过 LLM 端点代理把任意 agent 连接到 RL 训练，已被 verl Uni-Agent、AReaL 2.0、slime、Polar 等框架采纳。^[raw/articles/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528.md]

## 核心贡献

v1.0 是一个约 3500 行代码的轻量框架，支持任意 agent harness，作为研究 harnessed agentic RL 挑战的测试台。作者指出传统 agentic RL 与 harnessed agentic RL 的根本差异，并列出其中的关键挑战：^[raw/articles/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528.md]

- **Retokenization**（重分词）
- **Sample merging**（样本合并）
- **Advantage calculation**（优势计算）
- **Loss normalization**（损失归一化）
- **Backend scheduling**（后端调度）

这些环节会显著影响训练稳定性与有效性。

## 关键结果

框架在指令跟随、搜索、coding 三类 agent 上评估，并提供完整的 coding-agent RL 可复现流水线。仅用 **6K 训练样本**与中等算力，RL 将 Qwen3.5-9B 在 SWE-bench Verified 上从 **41.8% 提升到 56.4%**（+14.6 点绝对增益）。作者公开完整工作流与训练脚本以支持可复现研究。^[raw/articles/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528.md]

## 与既有知识的关联

该框架属于 Agentic RL 后训练领域，可对照 [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 框架与实践]]、[[entities/areal-2-agentic-rl-online-learning-self-evolving|AReaL 2.0]]、[[entities/agentenv-agentic-rl-execution-environment|AgentEnv 执行环境]] 等既有积累；其「harness 拥有环境循环、trainer 只观察 request-response」的解耦思路与 [[entities/nvidia-polar-agentic-rl-harness-proxy-2026-07-22|NVIDIA Polar Agentic RL Harness Proxy]] 高度一致。harness 作为训练参与方的视角可对照 [[entities/agent-harness-architecture-deep-dive-aksahy|Agent Harness 架构]] 与 [[entities/cuhk-slim-skill-lifecycle-agentic-rl-arxiv-2605-10923|CUHK Slim Skill Lifecycle]]，范式层面纳入 [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演进]] 与 [[concepts/agent-harness-engineering-paradigm|Agent Harness 工程范式]]。

→ [[raw/articles/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528|原文存档]]
