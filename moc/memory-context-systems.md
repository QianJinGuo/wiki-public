---
title: "Agent 记忆与上下文系统"
created: 2026-06-18
updated: 2026-06-18
type: moc
tags: [memory, context-engineering, context-management, long-memory]
---

# Agent 记忆与上下文系统

> 自动生成的 MOC，覆盖 40 个 entity 页面。

## 核心实体

- [[entities/17-agent-architectures-evolution|17种Agent架构演进：控制流设计的完整演化史]]
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|800行代码实现 Open Claw 的 Tool、消息总线、子Agent管理架构]]
- [[entities/acker-agent-evolution-three-routes-convergence|Agent演化：三条路线汇聚框架]]
- [[entities/agent-loop-engineering-handbook-8-questions-chen-jin-tencent-self-2026|Agent Loop 工程手册 8 个未解问题 + SELF Protocol 治理薄壳：腾讯陈进的二手解读与单 Agent 实验]]
- [[entities/agent-memory-architecture|Agent Memory 架构本质]]
- [[entities/agent-memory-architecture-ruofei|Agent Memory 架构解析]]
- [[entities/agent-memory-engineering-tax-aws-china-2026|Agent 记忆系统工程税：写入纪律·Prompt Cache 冲突·跨模型容量·Embedding 迁移·自产 Skill 治理]]
- [[entities/agentic-ai-infrastructure-practice-series-nine-context-engineering|Agentic AI Infrastructure Practice Series 9: Context Engineering]]
- [[entities/agentic-ai-system-architecture-harness-skill-mcp|MCP · Skill · Agent · LLM · Harness — 一张图讲清：Agentic AI 系统如何真正落地]]
- [[entities/agentmemory-source-analysis-coding-agent-local-memory|AgentMemory 源码分析：给 Coding Agent 装上本地长期记忆]]
- [[entities/agentos-minimax-forge-model-adaptation-yaoge|MiniMax Token调用第一后：AgentOS现实与模型厂商的系统适配挑战]]
- [[entities/agentscope-java-harness-framework|AgentScope Java Harness Framework — 企业级 Agent 分布式场景的 Harness 实现]]
- [[entities/ai-agent-engineer-capability-map|AI Agent 工程师能力地图]]
- [[entities/ai-coding-agent-memory-system|AI Coding Agent 记忆系统]]
- [[entities/ai-coding-入门指南-如何更好地让ai真正帮你干活|AI Coding 入门指南：如何更好地让 AI 真正帮你干活]]
- [[entities/ai-context-layer-kgc-2026|AI Context Layer 框架]]
- [[entities/ai-memory-architecture-deep-dive|AI Memory Architecture: Deep Dive]]
- [[entities/ai-native-startup-cyberfund-guide|How to Build an AI-Native Startup]]
- [[entities/alibaba-eventhouse-enterprise-agent-context|阿里云 EventHouse 企业级 Agent 上下文构建五维框架]]
- [[entities/aliyun-end-to-end-business-requirements-agent-multica-2026|阿里云端到端业务需求专家 Agent：Multica 平台 + superai-* 技能集群 + TDD/pre-push 质量门禁]]
- [[entities/anthropic-prompt-caching-claude-code|Prompt Caching 工程实践 — Anthropic Claude Code 经验总结]]
- [[entities/baixing-ontoz-enterprise-ontology-xinzhiyuan|百型智能 OntoZ：企业本体论 + 群智能体协同体系，新一代企业级 AI 基础设施]]
- [[entities/chatgpt-dreaming-v3-long-term-memory-openai|ChatGPT 的'失忆症'终于被治好了！Dreaming V3 让大模型拥有长期记忆]]
- [[entities/chatgpt-dreaming-v3-long-term-memory-xinzhiyuan|ChatGPT记忆大升级，十亿人免费用！]]
- [[entities/cheriot-ibex-memory-safety-hardware-enforcement|CHERIoT-Ibex: Closing the door on memory safety vulnerabilities with hardware-enforced protection]]
- [[entities/claude-code-7-layer-memory-architecture|claude-code-7-layer-memory-architecture]]
- [[entities/claude-code-agent-teams-xingxiaozhao|看完 Claude Code Agent Teams，我更确定接下来拼的是 Agent Runtime，技术拆解：Lead、Task List、Mailbox 和 Hooks 是什么东西]]
- [[entities/claude-code-and-what-comes-next|Claude Code and What Comes Next]]
- [[entities/claude-code-context-engineering-anthropic-thariq|Claude Code 上下文工程 —— Anthropic 团队的工程实践]]
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]]
- [[entities/claude-code-demo-to-production-8-gates-huang-jia-csdn-2026|Claude Code 从 Demo 到产线 · 企业 Harness 工程化的 8 道关卡（黄佳/咖哥 CSDN）]]
- [[entities/claude-code-dynamic-workflows-jiagoux-architect-perspective|Claude Code Dynamic Workflows 第 7 译本（架构师 JiaGouX 视角：任务级 Harness 统一框架）]]
- [[entities/claude-code-dynamic-workflows-jiqizhixin-9th-translation|Claude Code 团队成员亲述：动态工作流该怎么用（机器之心译本）]]
- [[entities/claude-code-dynamic-workflows-zhuge6-yucheng-translation|Claude Code Dynamic Workflows 第 6 译本（玉澄 / 51CTO 视角）]]
- [[entities/claude-code-large-codebase-agent-harness-13-patterns-tuutuiagi|面向大型代码库的 Claude Code 团队落地经验与扩展策略（Agent Harness）]]
- [[entities/claude-code-large-codebase-team-deployment-agent-harness|面向大型代码库的 Claude Code 团队落地经验与扩展策略（Agent Harness）]]
- [[entities/claude-code-memory-setup-obsidian-graphify|Claude Code Memory Setup (Obsidian + Graphify)]]
- [[entities/claude-code-memory-setup-token-71x楠楠自瑜|Claude Code 实践：token 效率提高 71.5 倍的工作流]]
- [[entities/claude-code-openclaw-memory-comparison|claude code openclaw memory comparison]]
- [[entities/claude-code-openclaw-memory-vector-db-doubt|claude code openclaw memory vector db doubt]]

## 待关联概念

- [[concepts/ai-ethics-responsible-ai|AI 伦理与负责任 AI]]
- [[concepts/lost-in-the-middle|Lost in the Middle 长上下文注意力衰减]]
- RAG 框架对比
- [[concepts/ssm-attention-sleep-consolidation-cmu-arxiv-2605-26099|SSM-Attention 睡眠巩固机制：CMU 让 LLM 在 N 次递归前向中「睡一觉「消化长上下文（arxiv 2605.26099）]]
- [[concepts/responsible-ai-governance|负责任 AI 治理体系]]
