---
title: "Harness Engineering - 让 Coding Agent 可靠完成长程任务"
type: entity
created: 2026-05-10
updated: 2026-09-07
tags: [harness, engineering, coding-agent, long-term, reliability]
sources:
  - raw/articles/harness-engineering-让-coding-agent-可靠完成长程任务
review_value: 7
review_confidence: 7
review_recommendation: strong
review_stars: 4
provenance_state: inferred
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.85: 长程任务同题3741字版，留6698字rv9; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Harness Engineering - 让 Coding Agent 可靠完成长程任务

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.85**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/harness-engineering-reliable-long-term-agent.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/code-as-agent-harness-survey|Code as Agent Harness 综述]] — 102页综述
- [[entities/agent-production-harness-engineering|Agent生产级Harness工程指南]] — 四支柱审查框架
- [[entities/agent-memory-architecture-ruofei|Agent Memory 架构解析]] — memory架构解析
- [[entities/agent-reliability-context-drift-tool-hallucination|Agent Reliability: Context Drift & Tool Calling Hallucination]] — 漂移与幻觉分层解法
- [[entities/agent-harness-6-runtime-patterns-sdb|Agent Harness 6 种运行模式与 SDB 方法论]] — SDB边界形式化
- [[entities/prime-agent-self-improving-rlm-agent|Prime Agent — 以 RLM + Continual Harness 双抽象为核心的自改进编码 Harness]] — RLM+Continual Harness：harness状态可CRUD
- [[entities/icml-2026-open-reproductions-agent-audit|ICML 2026 Open Reproductions — 大规模 Agent 驱动的论文复现审计]] — 2226篇论文35908个claim的agent复现审计
- [[entities/databricks-glm-52-3000-engineer-benchmark-coding-agent|Databricks CEO用3000名程序员真实任务测试GLM 5.2 — Harness选择比模型更重要]] — 3000工程师实测
- [[entities/semaplc-verification-gated-agent-harness-plc-codegen-2026-08-26|SemaPLC：验证门控的 PLC 代码生成 Agent Harness]] — 验证门控PLC代码生成harness

## 工程实践
- [[entities/harness-engineering|'Harness Engineering：AI 从]] — 六层架构+七大反模式+分级决策树19712字rv9
- [[entities/cursor-复盘-harness模型决定能力上限harness-决定生产下限|Cursor 复盘 Harness：模型决定能力上限，Harness 决定生产下限]] — Cursor复盘主版
- [[entities/deepseek-code-harness|DeepSeek Code Harness]] — DSH 28k主版
- [[entities/vivo-agent-brain-body-icu-harness-evolutionary-framework-2026|vivo Agent 系统分析：大模型是大脑不是马，Harness 是 ICU 不是马鞍]] — 大脑身体ICU隐喻框架
- [[entities/ai-dlc-zixun-ai-native-development-lifecycle|AI-DLC：紫讯落地 AI 原生研发新范式的实践]] — AI-DLC四件套
- [[entities/microsoft-fastcontext-coding-agent-explore-subagent-vibecoder|FastContext（微软开源 Coding Agent 仓库探索子代理）]] — Explore子Agent只读三工具，证据bundle
- [[entities/bedrock-agentcore-coding-agent-hosting|It’s safe to close your laptop now: Hosting coding agents on Amazon Bedrock AgentCore]] — 远端工作站即服务
- [[entities/baidu-comate-coding-agent-feedback-loop-wanpeng|Coding Agent在百度的落地实践：从反馈闭环到工程范式重构]] — 双层loop实践
- [[entities/harness-engineering-systematic-explainer|Harness Engineering 系统性解读]] — 李宏毅课程解读7933字最全版
- [[entities/devin-fusion-multi-model-harness-cognition|Devin Fusion: 多模型路由 Harness 实现 35% 成本降低]] — Sidekick双agent路由
- [[entities/skill-hell-agent-skill-engineering-ruofei|Skill Hell：Agent Skill 工程方法论]] — Skill Hell四失效模式治理
- [[entities/从vibe-coding到harness-一套大仓ai工程化实战|从Vibe Coding到Harness—— 一套大仓AI工程化实战]] — TAB大仓实战2293字版
- [[entities/claude-code-multi-agent-harness-source-analysis|Claude Code 多 Agent Harness 源码拆解：留纸条、抠上下文、抠缓存、捆手脚]] — 留纸条抠上下文
- [[entities/qunar-ai-coding-large-core-system-refactor-2026|去哪儿 AI Coding 驱动大型核心系统重构 — Harness+Loop+Task 工程化方法论]] — 15万行重构提效70%：Harness约束+Loop推进
- [[entities/ai-native-sdlc-playbook-anthropic|AI Native SDLC Playbook：Anthropic 应用 AI 团队的软件开发生命周期重构方法论]] — SDLC六阶段重构
