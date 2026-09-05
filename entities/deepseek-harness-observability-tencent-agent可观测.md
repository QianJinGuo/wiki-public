---

title: "DeepSeek Harness 规模化踩坑实录：Agent 可观测方案"
created: 2026-08-30
updated: 2026-09-06
type: entity
tags: [harness, observability, deepseek, agent, monitoring, tracing]
sources: [raw/articles/deepseek-harness-observability-tencent-agent可观测]
confidence: 0.8
---

# DeepSeek Harness 规模化踩坑实录：Agent 可观测方案

## 摘要

腾讯云日志服务团队分享了 DeepSeek Harness（DSH）规模化使用中的可观测经验，提供一套以"插件采集 + 结构化调用链 + 控制台分析"为核心的全景 Agent 可观测方案。核心洞察是：DSH 原生能力解决的是**本机、单会话、实时**的调试问题，规模化运维需要补充**跨会话聚合、跨机汇聚、长期留存与告警**能力。^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

## 核心要点

- **DSH 架构**：Cordis 律内核 + 全插件化——内核只负责按 profile 装配插件与管理生命周期，模型适配器、工具集、沙箱策略、会话持久化均为可插拔插件；web / tui / headless 三种形态共用同一内核 ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]
- **规模化三大痛点**：耗时分析（模型推理 vs 工具执行）、成本追踪（token 集中在哪些会话/模型）、失败回溯（失败与中断发生在哪一步、是否可回溯）^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]
- **DSH 插件的形态**：以原生插件形态挂载，将执行过程还原为结构化调用链，覆盖任务与推理轮次、模型调用、工具调用、会话关联 ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]
- **多种上报方式**：DSH 插件之外，还支持通用 Agent 框架（LangChain、OpenAI Agent SDK 等）与 Coding Agent（CodeX、Claude Code 等），全部写入同一套数据模型，可在同一控制台横向对比 ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]
- **控制台五类分析能力**：应用列表、仪表盘、调用链、会话、告警 ^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

## 深度分析

### 1．"单机会话实时"到"跨机长期留存"：Harness 规模化运维的本质跃迁

DSH 自带会话轨迹视图、Session 事件流落盘与工具调用检索，但这些能力的边界清晰：**本机、单会话、实时**。一旦 Agent 从个人开发机走向团队协作、多机部署，问题就变了：一个失败发生在哪台机器、哪个会话、哪个环节？token 成本集中在哪个团队、哪个模型？这些都需要把分散在单机上的运行时事件汇聚到统一的数据模型。腾讯云方案的切入点正是如此——通过原生插件把 DSH 的运行时事件还原为结构化的五层调用链，再叠加跨会话聚合、跨机汇聚、长期留存与告警，完成从"调试工具"到"运维体系"的跃迁。这是 Harness 工程从单机开发走向团队生产的关键分水岭。^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

### 2．Agent 可观测的分层模型：调用链是骨架，会话是单位，告警是闭环

控制台的分析能力呈现了一个完整的 Agent 可观测分层：**应用列表**回答"谁在跑、跑多少"（接入应用、上报状态、昨日写入量与 token 总数）；**仪表盘**回答"整体表现如何"（请求/错误数、模型调用次数、Input/Output Tokens、Agent 与模型 Top10、平均 TTFT、P50/P90/P99 耗时分位）；**调用链**回答"这一次调用哪里出了问题"（Traces/Span 双视图，按状态、错误类型、Trace ID、Session ID 与耗时筛选，详情页提供调用树、节点 Input/Output 与错误根因定位）；**会话**把 Trace 按 Session 聚合，还原多轮对话上下文；**告警**对耗时、失败率等异常指标主动触发。五层能力从总览到根因逐步下钻，构成"发现→定位→复盘→预警"的完整闭环。^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

### 3．插件化是 Harness 可观测接入的最优形态

与 langchain、OpenAI Agent SDK 等通用框架的接入方式不同，DSH 的插件形态直接复用其"全插件化"架构：模型适配器、工具集、沙箱策略都是插件，可观测采集同样以原生插件挂载，无需侵入内核。这意味着采集点与框架的执行路径天然对齐——能拿到任务与推理轮次、模型调用、工具调用的精确边界，而不是靠外部代理猜测。同时，多接入方式写入同一套数据模型的设计，让自研框架与生态框架（CodeX、Claude Code）的观测数据可横向对比，避免为每个框架搭建孤立的观测栈。^[raw/articles/deepseek-harness-observability-tencent-agent可观测.md]

## 实践启示

1. **先划清"单机调试"与"规模化运维"的边界**：如果 Agent 还在个人开发阶段，DSH 原生的轨迹视图与事件流落盘就够用；一旦进入团队协作或多机部署，就要立刻补齐跨会话聚合、跨机汇聚、长期留存与告警能力，而不是继续依赖单机会话视图。
2. **可观测采集点优先选插件形态**：能原生挂载就原生挂载，让采集与执行路径对齐；对外部无法插桩的框架，再考虑通用 SDK 上报，并保证写入同一套数据模型以便横向对比。
3. **按"应用→仪表盘→调用链→会话→告警"五层建设观测体系**：先有应用总览与仪表盘（发现异常），再靠调用链与会话下钻（定位根因），最后用告警把重复的排查变成主动预警。
4. **错误分类要贴合 Agent 语义**：工具调用失败、LLM 调用失败、Root span 状态码异常、Agent 执行失败——这些错误类型是 Agent 场景特有的，比通用的 HTTP 状态码更能快速定位失败环节。
5. **规模化前先定义成本与耗时的聚合口径**：token 集中在哪些会话、哪些模型，耗时花在推理还是工具执行——这两类问题在单会话里看不出价值，必须在接入第一天就确定聚合维度，否则后续无法追溯。

## 相关实体

- [[entities/agent-harness-production|Agent Harness 生产化]]
- [[entities/harness-engineering-core-patterns-claude-code|Harness Engineering 核心模式]]
- [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- [[entities/on-device-harness-qwen38-27b-portable-computer|端侧模型专用 Harness]]

→ [[raw/articles/deepseek-harness-observability-tencent-agent可观测|原文存档]]