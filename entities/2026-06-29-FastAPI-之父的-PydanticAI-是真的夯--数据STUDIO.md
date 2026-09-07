---
title: "1. 定义你的输出（这是合约，不是建议）"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-29-FastAPI-之父的-PydanticAI-是真的夯--数据STUDIO]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/2026-06-29-FastAPI-之父的-PydanticAI-是真的夯--数据STUDIO.md|原文存档]]

sha256: 48732b4d99aa53fee8fdb93d7d2bc1bf15cfa5bf6c37425e32734eae5b672d3d ^[raw/articles/2026-06-29-FastAPI-之父的-PydanticAI-是真的夯--数据STUDIO.md]

## 摘要

文章解读 FastAPI 之父 Samuel Colvin 的 Agent 框架 PydanticAI：既然 LLM 的概率采样本质决定了 prompt 无法保证输出格式（作者实测 prompt-only 输出结构化 JSON 跑 100 次完全合规率仅 73%，27 次失败中字段漂移 11 次、类型不对 9 次、嵌套缺失 7 次），正确解法是把 FastAPI 的"类型声明可执行"哲学搬进 Agent——用 result_type 声明 Pydantic 输出模型，框架自动校验、失败即发 ModelRetry 让模型重来。核心设计是泛型 Agent[DepsT, OutputT]：每次调用编译为三节点有向图——UserPromptNode（拼装 system prompt、注入输出 schema 与工具定义）、ModelRequestNode（封装 25+ provider，优先用 native structured output API）、CallToolsNode（两阶段 validator pipeline：Pydantic 校验失败发 ModelRetry 控制流信号，默认重试 1 次最多可配 5 次，配套 UsageLimits 硬截断防 token 失控）。文章给出踩坑经验（schema 过严让模型变笨、缺多 Agent 协作、UsageLimits 易被忽略）与选型对比（LangChain 哲学是"链"、CrewAI 是"团队"但多耗 15-20% token、PydanticAI 是"合约"），并提到 2026 年 1 月发布的 Monty Python-in-Rust VM 冷启动 6 微秒、任务成本从 2 美元降到 4 美分，PydanticAI v2.0 正在 beta（6 月 10 日 beta 7）。^[raw/articles/2026-06-29-FastAPI-之父的-PydanticAI-是真的夯--数据STUDIO.md]

## 关键要点

- 问题根源：temperature + 概率采样 = 非确定输出，prompt 只能调概率分布、无法 guarantee 字段名与类型；实测 prompt-only 结构化 JSON 合规率 73%，每 4 次调用约 1 次炸下游
- 核心抽象 Agent[DepsT, OutputT]：DepsT 依赖注入进 RunContext（同 FastAPI 的 Depends，无全局状态）；OutputT 支持 Pydantic model、TypedDict、primitive、Union
- 三节点执行图：UserPromptNode（拼 prompt + 注入 schema/工具定义）→ ModelRequestNode（25+ provider 优先 native structured output，切换模型只改一个字符串）→ CallToolsNode（tool_call 循环 + final output 两阶段校验）
- ModelRetry 是控制流信号而非异常：把校验失败信息作为 RetryPromptPart 注回模型（"confidence 应为 float"），默认重试 1 次、最多 5 次；UsageLimits（request_limit + total_tokens）做硬截断
- 踩坑三条：核心字段用精确类型、辅助字段用 str 容错（过严 regex 让 LLM 分心凑格式）；PydanticAI 无多 Agent 协作（仅 agent_as_tool 串联），CrewAI 角色系统灵活但多耗 15-20% token 且社区报告过单次运行 $414 的失控案例
- 框架选型：单 Agent 输出接下游选 PydanticAI、多 Agent 角色协作选 CrewAI、复杂图编排选 LangGraph、OpenAI 生态绑定选 OpenAI Agents SDK；Pydantic 2026 年 2 月突破 10 亿下载，PydanticAI GitHub 17.6k stars（MIT）

## 来源

- 原文: [[raw/articles/2026-06-29-FastAPI-之父的-PydanticAI-是真的夯--数据STUDIO.md|1. 定义你的输出（这是合约，不是建议）]]
