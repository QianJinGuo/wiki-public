---
title: Agent 工程全景指南
type: moc
created: 2026-06-18
updated: 2026-11-25
tags: [agent, harness, loop-engineering, memory, sub-agents, architecture]
description: Agent 工程从入门到精通：harness 设计、循环工程、记忆系统、子 Agent 编排、自改进闭环。
---

# Agent 工程全景指南

> 阅读顺序：先建立 harness 视角，再理解 loop 与 memory，最后进入多 Agent、自改进和生产案例。所有链接均为当前活跃页面。

## 核心概念

- [[concepts/harness-engineering-framework|Harness Engineering 框架]] — 模型之外的运行时、工具、状态与验证层。
- [[concepts/loop-engineering-methodology|Loop Engineering 方法论]] — 从一次性 prompt 到持续运行的控制循环。
- [[concepts/agent-memory-architecture|Agent 记忆架构]] — 上下文、外部记忆与遗忘机制。
- [[concepts/subagent-spawning-pattern|Subagent 派生模式]] — 上下文隔离与并行分工的基本原语。
- [[entities/four-sub-agent-patterns|四类子 Agent 模式]] — 研究、实现、评审与运维的职责拆分。

## Harness 工程

- [[entities/harness-engineering-14-step-roadmap|Harness 工程 14 步路线图]] — 从单 Agent 到自改进系统。
- [[entities/harness-engineering-core-patterns|Harness Engineering 核心模式]]。
- [[entities/agent-harness-12-components-7-decisions|Agent Harness 的 12 个组件与 7 个决策]]。
- [[entities/agent-harness-architecture-deep-dive-aksahy|Agent Harness 架构深度拆解]]。
- [[entities/agent-harness-architecture|Agent Harness 生产架构]]。
- [[entities/agent-harness-observability-production|Agent Harness 可观测性实践]]。
- [[entities/一文带你弄懂-ai-圈爆火的新概念harness-engineering|一文弄懂 Harness Engineering]]。
- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道|Claude Code 源码中的 Agent Harness]]。

## 循环工程

- [[entities/loop-engineering-feedback-control-system|Loop Engineering：把反馈循环放进工程现场]]。
- [[entities/loop-engineering-addy-osmani-challengehub|Addy Osmani：Loop Engineering 深度解读]]。
- [[entities/agent-self-improvement-six-mechanisms|Agent 自我改进的六条路径]]。
- [[entities/harness-engineering|Harness Engineering：Loop 与 Harness 的关系]]。

## 记忆与上下文

- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进|Agent 记忆系统的工程实践与演进]]。
- [[entities/agent-memory-architecture-past-influence-future-ruofei|Agent Memory：过去、影响与未来]]。
- [[concepts/context-management-agent-systems|Agent 上下文管理系统]]。
- [[entities/memory-vs-rag-agent-memory-systematic-framework|Memory 不是 RAG：Agent 记忆的系统性框架]]。

## 架构范式

- [[entities/你不知道的-agent原理架构与工程实践|Agent 原理、架构与工程实践]]。
- [[concepts/ahe-agentic-harness-engineering|Agentic Harness Engineering]]。
- [[entities/karpathy-vibe-coding-agentic-engineering|从 Vibe Coding 到 Agentic Engineering]]。
- [[entities/owner-worker-verifier-architecture|Owner-Worker-Verifier 架构]]。
- [[entities/agent-workflows|Agent Workflows]]。
- [[entities/agent-开发范式演进从环境工程出发|Agent 开发范式：从环境工程出发]]。
- [[entities/guide-ai-agents-models-apps-harnesses-mollick|AI Agent 选型指南：模型、应用与 Harness]]。

## 多 Agent 与团队编排

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw 完全指南]]。
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验|OpenClaw 多智能体团队搭建经验]]。
- [[entities/构建基于多智能体架构的深度思考交易系统-v2|多 Agent 深度思考交易系统]]。
- [[entities/claude-code-dynamic-workflows-multi-agent-orchestration|Claude Code 动态工作流与多 Agent 编排]]。

## 生产案例

- [[entities/anthropic-95pct-data-analysis-skill-stack-architecture|Anthropic 95% 数据分析技能栈]]。
- [[entities/agent-architecture-harness-new-backend|Harness 正在成为新的后端]]。
- [[entities/harness-pilot-claude-code-plugin-yangtong-2026-06-17|Harness Pilot：Claude Code Plugin]]。
- [[entities/ssd-spec-driven-development-harness-asd-shuge-2026-06-17|SSD：Spec-Driven Development Harness]]。

## 相邻 MOC

- [[moc/memory-context-systems|记忆与上下文系统]]。
- [[moc/loop-engineering|Loop Engineering]]。
- [[moc/multi-agent-coordination|多 Agent 协作]]。
- [[moc/coding-agent-practice|编程 Agent 实践]]。
- [[moc/evaluation-and-benchmarks|评测与基准]]。

## 使用说明

- 初学：先读“核心概念”，再看一个具体 harness 拆解。
- 设计系统：从“Harness 工程”进入，补齐 loop、memory 与 eval。
- 做多 Agent：先确认任务真的需要并行，再读“多 Agent 与团队编排”。
- 生产落地：直接看“生产案例”，回到相应 entity 查证据与边界。
