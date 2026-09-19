---
title: "第 11 篇 · 上下文压缩：让 Agent 在长会话中持续工作"
created: 2026-07-27
updated: 2026-09-19
type: entity
tags: [agent, context, memory, compression, harness-engineering]
sources:
  - raw/articles/agent-context-compression-yexiaochai-11
confidence: 0.65
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 第 11 篇 · 上下文压缩：让 Agent 在长会话中持续工作

> Author: 叶小钗 | Source: 微信公众号

本文是叶小钗 Agent 系列教程的第 11 篇，聚焦于 Agent 上下文压缩的原理与实现。介绍了上下文窗口概念、Agent 上下文增长的原因、上下文过长带来的问题，以及上下文压缩的三种策略（消息丢弃、消息摘要、滑动窗口 + 摘要），并结合 mini-openclaw 项目源码说明了压缩触发时机、消息切分、摘要生成和上下文重建的实现。^[raw/articles/agent-context-compression-yexiaochai-11.md]

## 上下文窗口与 Agent 上下文增长

每个大模型都有一个上下文窗口（context window），当前主流模型已普遍提升到 100 万 token 左右（DeepSeek V4 Flash/Pro: 1M, GPT-5.6: 1.05M, Claude Sonnet 5: 1M, Gemini 3.5 Flash: 1M）。每次模型调用时，上下文占用 = system prompt + user 消息 + assistant 历史消息 + tool definitions + tool 调用结果 + 多模态内容 + 模型本轮输出。^[raw/articles/agent-context-compression-yexiaochai-11.md]

Agent 上下文增长迅速的原因在于：大模型是无状态的，每次调用需要重新组装全部历史消息；而工具调用结果（文件全文、搜索结果、命令行输出、API/MCP 返回值）通常比用户输入和模型输出占用更多 token。^[raw/articles/agent-context-compression-yexiaochai-11.md]

上下文超过模型上限后，API 请求会直接失败，这是长会话 Agent 必须解决的工程问题。^[raw/articles/agent-context-compression-yexiaochai-11.md]

## 三种压缩策略

### 1. 消息丢弃
对消息按重要性分级，保留关键消息（工具调用结果、关键回复），丢弃低优先级消息（问候语、确认性回复）。实现简单但可能丢失有用信息。^[raw/articles/agent-context-compression-yexiaochai-11.md]

### 2. 消息摘要
对多轮历史消息生成摘要，仅保留摘要文本。优点是压缩比高（多轮→一段话），缺点是摘要会丢失具体细节（代码片段、API 返回值）。^[raw/articles/agent-context-compression-yexiaochai-11.md]

### 3. 滑动窗口 + 摘要
综合方案：保留最近的 K 轮完整消息（滑动窗口），对更早的历史进行摘要压缩。兼顾了近期细节和远期上下文。^[raw/articles/agent-context-compression-yexiaochai-11.md]

## mini-openclaw 实现

在 mini-openclaw 项目中，上下文压缩在每次收到新的消息时进行判断。如果当前会话占用接近上下文窗口上限，则触发压缩流程：^[raw/articles/agent-context-compression-yexiaochai-11.md]

1. **消息切分**：将历史消息分为保留区（最近的 N 轮对话）和压缩区（更早的消息）
2. **摘要生成**：对压缩区的消息逐段生成摘要，使用独立 LLM 调用进行压缩
3. **上下文重建**：将保留区完整消息 + 压缩区的摘要合并为新的 messages 数组，替换原有的全部历史消息

核心实现在 `context_compressor.py` 的 `compress()` 方法中，采用 LLM 驱动的摘要压缩策略。代码逻辑包括消息长度统计、压缩触发阈值判断、保留轮次配置和摘要 prompt 模板。^[raw/articles/agent-context-compression-yexiaochai-11.md]

## 深度分析

### 压缩即选择损失函数：被丢弃的历史无法在生成时召回

RAG 假设被删内容仍在向量库里、生成前可检索；压缩没有这条退路 —— `compress()` 返回新数组后原文即被替换，模型只能靠摘要里剩下的字面内容推理。因此压缩是在选择损失函数：滑动窗口最小化「位置加权损失」（越新越保），消息丢弃最小化「显著性加权损失」（越重要越保），摘要最小化「任务状态重建误差」。失败模式随之不同：前者丢掉早期否定式约束，中者丢掉低频但关键的错误栈，后者丢掉需逐字复现的文件路径与 API 签名。^[raw/articles/agent-context-compression-yexiaochai-11.md]

压缩的收益与风险在消息类型上不对称：工具结果（文件全文、搜索结果、命令输出、MCP JSON）动辄数千 token，占掉绝大部分窗口，但大多可重新推导 —— 文件能按路径重读、命令能重跑、MCP 能重查，压缩它只损失一次 I/O；用户轮次与已做出的决策不可重新推导，被摘要掉即永久丢失。取舍轴不是「位置」（谁更旧），而是「可重新推导性 × 时效性」：优先压缩可重建的大块文本，并把坐标（路径、命令、参数）留在摘要里供取回（[[concepts/harness-context-window-management]]、[[entities/agent-harness-context-management-working-set]]）。^[raw/articles/agent-context-compression-yexiaochai-11.md]

### 摘要漂移：自我摘要会产生复合误差

若下一轮压缩拿上一版摘要当输入再摘一次，误差会逐轮复利：每层都按自己的判断丢细节，模型补全的猜测在下一层被当成事实传递，几轮后摘要既短又自信地偏离真实历史。它不报错，只表现为 Agent 重做工作、忘记约束或引用不存在的文件。两条硬规则：摘要必须从原文（verbatim 消息）重新生成，摘要链深度恒为 1，而非在摘要之上继续叠加；「发生了什么」的叙述与「现在什么是真的」的状态必须分开，后者应是结构化字段（目标、已完成、当前进度、未决问题、用户显式约束）。全量历史落盘（JSONL）是重建摘要的前提，[[entities/openai-skills-shell-compaction-agent-primitives]] 正是把这件事标准化。^[raw/articles/agent-context-compression-yexiaochai-11.md]

### 触发时机与迟滞：过早压缩既烧钱又毁缓存

mini-openclaw 在每条新消息到达时检查占用率，但触发有两个隐性代价：一次额外的 LLM 调用（摘要生成），以及 prompt prefix cache 全部失效 —— 压缩改变消息数组的字节前缀，之后每轮请求都要重算被改动的部分，直到前缀重新稳定。阈值贴得太近会高频抖动：刚释放的空间被几轮工具调用吃回去，于是再压一次，缓存永远命中不了。合理做法是双水位迟滞控制：高水位（75%–80%）触发、压缩后落到低水位（50% 以下）、两次之间强制间隔若干轮；触发判定还要包含 tool definitions 与为本轮输出预留的 token。^[raw/articles/agent-context-compression-yexiaochai-11.md]

### 重建步骤的静默失败模式

`compress()` 最后一步是把保留区原文加摘要拼成新 messages 数组整体替换历史，其正确性依赖几条隐式不变量：system prompt 仍在首位且未被摘要；每条 `tool` 消息都能找到发起它的 `assistant.tool_calls`；没有悬空的 tool_calls 等待结果。失败往往静默：切分点落在一轮对话中间时，保留区可能只剩结果而缺发起方，部分 API 直接报错，另一些接受但行为退化 —— 模型忽略工具结果或重复发起同一调用。工程上应做「边界吸附」：只从完整用户轮次边界开始保留，发送前对重建数组做结构断言（配对加角色检查），失败则回退到上一个安全边界。同源的失败模式参见 [[entities/agent-reliability-context-drift-tool-hallucination]]。^[raw/articles/agent-context-compression-yexiaochai-11.md]

## 实践启示

1. **边界吸附到轮次，而不是 token 位置**：保留区从完整用户轮次开始；重建后先断言 tool_call/tool_result 配对与角色合法性再发请求，失败就回退到上一个安全边界。
2. **优先压缩可重新推导的工具结果**：文件全文、搜索结果、命令输出、MCP JSON 先落盘再压缩，摘要里保留取回坐标（路径、命令、参数），用户消息与已确定的决策不进删除区。
3. **摘要用状态字段而非叙事**：模板覆盖任务目标、已完成、当前进度、未决问题、后续步骤与用户显式约束，否定式约束原样引用。
4. **摘要链深度恒为 1，全量历史持久化**：会话原文写入 JSONL 或事件日志，作为摘要的唯一事实来源，使摘要可重建、可审计。
5. **双水位迟滞触发，并把缓存代价计入阈值**：高水位触发、压缩后落回低水位、两次压缩间设最小轮次间隔；尽量安排在任务阶段完成或切换任务的天然断点（参见 [[entities/anthropic-prompt-caching-claude-code]] 与 [[entities/openclacky-harness-prompt-cache]]）。
6. **度量并回归测试压缩**：记录压缩次数、释放的 token、摘要长度与压缩后首个请求的失败率；用真实长会话回放做对照测试，对关键约束与文件路径建立必须保留的断言集。

## 相关实体
- [[entities/langchain用agent做销售获客3个月转化率提升25倍看完后我发现国内-agent-落地的方法都错了]]
- [[entities/meituan-longcat-vitabench-20-long-term-dynamic-agent-benchmark]]
- [[entities/ai-coding-agent-quality-defense-five-control-mechanisms-tutu-agi]]
- [[entities/tencentdb-agent-memory-long-term-pyramid]]
- [[entities/hermes-agent-loop-architecture]]

## 相关概念
- [[concepts/context-engineering]]
- [[concepts/context-management-framework]]
- [[concepts/context-management-agent-systems]]
