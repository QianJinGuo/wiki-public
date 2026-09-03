---

title: "从Prompt、Context到Harness，工程的三次进化与终局之战"
created: 2026-06-10
updated: 2026-08-29
tags: [agent, code, evaluation, harness-engineering, llm, memory, prompt, prompt-engineering, rag, tool-use, loop-engineering, skills, paradigm-shift]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/prompt-context-harness-three-evolutions-tencent
  - raw/articles/agent-paradigm-2024-to-2026-skills-harness-loop-parsevolve
  - raw/articles/prompt-to-harness-enterprise-agent-evolution-qwen-ai-2026-07-20
---

# 从Prompt、Context到Harness，工程的三次进化与终局之战


## 相关实体

- [[entities/didi-ibg-customer-experience-llm-quality-inspection-3-pipelines|滴滴 ibg 智能客服质检系统：3 管线（意图 86% / 合规 90%+ / voc）+ 企业 llm 落地方法论]]
→ [[raw/articles/prompt-context-harness-three-evolutions-tencent|原文存档]] ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]

## 深度分析

从Prompt、Context到Harness，工程的三次进化与终局之战 涉及agent领域的核心技术议题。 ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]
### 核心观点
1. # 从Prompt、Context到Harness，工程的三次进化与终局之战 ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]
OpenAI 内部 3-7 人小团队，在五个月内让 AI 生成了将近 100 万行生产级别代码。 ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]
2. 全程没有一个工程师亲手写过一行业务逻辑代码。 ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]
3. 三个关键概念：Prompt Engineering、Context Engineering、Harness Engineering。 ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]
4. ## 第一进化：Prompt Engineering ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]
### 核心本质
LLM 底层逻辑是一个极其擅长续写的系统。 ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]
5. 给它一段输入，它预测接下来最有可能出现的内容，不断生成，直到任务完成。 ^[raw/articles/prompt-context-harness-three-evolutions-tencent.md]

### 关联实体

- [[entities/一文带你弄懂-ai-圈爆火的新概念harness-engineering]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/你不知道的-agent原理架构与工程实践-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道]]
- [[entities/ethan-he-cosmos-grok-imagine-latent-space-video-agent-20260606]]

## [MERGE 新增] 2024→2026 范式图谱：旧零件如何被新底座吸收

*新增来源：raw/articles/agent-paradigm-2024-to-2026-skills-harness-loop-parsevolve*

### 演化主线

完整的演化路径：**Prompt Engineering → Context Engineering → Harness Engineering → Loop Engineering**。每一阶段并不取代前一阶段，而是将其"吃进去"作为实现细节。^[raw/articles/agent-paradigm-2024-to-2026-skills-harness-loop-parsevolve.md]

### 旧概念与新底座映射

| 2024 零件 | 解决的问题 | 2026 如何被吸收 |
|-----------|-----------|----------------|
| **RAG** | 模型不知道你的私有知识 | 降级为 Agent 手里一个按需调用的检索工具（如 Claude Code 用 grep 而非 RAG 检索代码库）|
| **ReAct** | 模型只会一次性作答 | 变成 Loop 内部的一圈"心跳"，包进更大的自驱循环中 |
| **Function Call** | 模型碰不到真实世界 | 标准化为 MCP 协议，成为工具调用的统一接口 |
| **Prompt Engineering** | 模型不理解你的意图 | 变成 Context Engineering 的一个子集 |

### 三个新底座

1. **Skills（技能底座）** — 接管"知识怎么给"。渐进式披露：只给模型看能力目录，用了才现取。技能图谱正在取代向量检索成为默认底座。^[raw/articles/agent-paradigm-2024-to-2026-skills-harness-loop-parsevolve.md]

2. **Harness（驾驭底座）** — 接管"怎么不让模型跑偏"。套上工具、记忆、权限边界、反馈回路、出错恢复，让长链路运行可控。公式：Agent = 模型 + Harness。注意：Harness 策略会随每次模型升级被重新定价。^[raw/articles/agent-paradigm-2024-to-2026-skills-harness-loop-parsevolve.md]

3. **Loop（循环底座）** — 接管"谁来按回车"。你从"亲自接电话的接线员"升级为"设计工单系统的人"。设计那个替你给 agent 打 prompt 的循环。ReAct 被包进这个更大的自动循环里。^[raw/articles/agent-paradigm-2024-to-2026-skills-harness-loop-parsevolve.md]

### 核心洞察

旧地图没死，只是颗粒度不对了。旧地图是"零件级"（一个问题配一个零件），新地图是"系统级"（三个底座管三大类问题）。零件会被不断重新打包，但"知识、稳定、自驱"这三个底座要解决的问题会一直在。^[raw/articles/agent-paradigm-2024-to-2026-skills-harness-loop-parsevolve.md]

→ [[raw/articles/agent-paradigm-2024-to-2026-skills-harness-loop-parsevolve.md|原文存档]]

## 补充：企业级实战案例（千问AI平台 2026-07-20）

千问AI平台储旭(槿柏)提供了该演进路径的企业级实战案例，从大模型四个结构性约束（上下文有效容量、数据搬运失真、注意力自恶化、无状态）出发，详述了从 Prompt→Context→Harness 每个阶段的落地细节。^[raw/articles/prompt-to-harness-enterprise-agent-evolution-qwen-ai-2026-07-20.md]

### 结构性约束的具体表现

| 约束 | 表现 | 工程方案 |
|------|------|----------|
| 上下文容量 ≠ 有效容量 | 128K 窗口 70% 是噪音数据 | 渐进式披露 + 信息生命周期管理 |
| 数据搬运失真 | LLM 搬运 UUID 时截断/混淆 | parameterBindings 声明式绑定 |
| 注意力自恶化循环 | 上下文膨胀→参数错误→更多重试 | 逐步骤 dump 上下文诊断 |
| 无状态 | 跨执行遗忘，重复犯错 | 信息生命周期（产生→压缩→索引→恢复） |

### 核心工程原则

> 不要用更大的模型掩盖工程层面的问题。让 LLM 做它擅长的事（理解、规划、推理），让系统做它擅长的事（数据搬运、格式转换、精确传递）。

### 五层 Harness 架构

1. 原始证据层 → 2. 状态管理层 → 3. 技能层 → 4. 治理层 → 5. 运行时层

每层解决前一阶段遇到的天花板，与本文"Prompt→Context→Harness"的三次进化叙事一致。

→ [[raw/articles/prompt-to-harness-enterprise-agent-evolution-qwen-ai-2026-07-20|原文存档]]

