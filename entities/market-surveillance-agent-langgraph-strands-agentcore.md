---
title: "Market surveillance agent with LangGraph and Strands on AgentCore"
created: 2026-07-29
updated: 2026-09-18
type: entity
tags: [agent, multi-agent, langgraph, strands, agentcore, aws, bedrock, market-surveillance, financial-services]
sources: [raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Market surveillance agent with LangGraph and Strands on AgentCore

AWS 官方博客发布的一篇深度技术文章，演示如何将 LangGraph（宏观工作流编排）与 Strands（智能 Agent 推理引擎）结合在 Amazon Bedrock AgentCore 上构建生产级市场监控多 Agent 系统。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md]

## 架构设计

文章提出三层架构：LangGraph 负责宏观编排（状态管理 + 有向图 + checkpoint 恢复），Strands Agent 在单个工作流节点内充当推理引擎，AgentCore 提供生产级部署基础设施。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md]

核心构件包括：
- **市场监控 Agent**（Strands Agent with AgentCore）：分析交易模式、识别可疑活动
- **监管报告 Agent**（Strands Agent with AgentCore）：生成合规报告
- **多 Agent 协调**：LangGraph 管理 Agent 之间的工作流和数据流
- **AgentCore Memory**：通过 `langgraph-checkpoint-aws` 包实现持久化和长短期记忆检索

## 关键技术点

### AgentCore Memory 集成

LangGraph 通过 `AgentCoreMemorySaver` 和 `AgentCoreMemoryStore` 类与 AgentCore 记忆集成，自动保存 checkpoint 到 AgentCore 记忆，实现有状态会话和工作流恢复——无需管理 DynamoDB 表或实现自定义序列化逻辑。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md]

### 可观测性

AgentCore 内置 CloudWatch 和 AWS X-Ray 集成，捕获 Agent 执行轨迹、工具调用和性能指标，结合 LangGraph 的 OpenTelemetry events 提供端到端可见性。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md]

### LangGraph 有向图状态管理

LangGraph 管理所有 Agent 之间的状态共享，通过 checkpoint 系统提供人机协同交互和故障恢复能力。文章演示了如何定义 Agent 节点之间的有向图数据流。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md]

## 互补角度

本文在现有 [[entities/langgraph-state-machine-under-the-hood|LangGraph 状态机]] 和 [[entities/strands-agents|Strands Agents]] 知识基础上贡献了以下独特角度：^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md]

1. **LangGraph + Strands + AgentCore 三件套组合**：现有实体分别覆盖 LangGraph 和 Strands，但本文演示了两者结合部署在 AgentCore 上的完整方案
2. **金融领域应用**：市场监控/合规场景的多 Agent 系统设计
3. **AgentCore Memory 深度集成**：通过 `langgraph-checkpoint-aws` 包将 LangGraph checkpoint 持久化到 AgentCore
4. **记忆存储（MemoryStore）**：AgentCore 自动从对话中提取洞察、摘要和用户偏好，支持跨会话检索
5. **可观测性体系**：CloudWatch + X-Ray + OpenTelemetry 三合一

## 深度分析

### 确定性编排骨架 + 节点内自治推理

金融流程由监管预先规定，"该走哪几步"交给 LLM 的非确定性去赌即是风险，但流程内确有步骤需要敏捷推理。文章把两种智能分层：LangGraph 只做宏观编排（图状态机、共享状态、条件路由、checkpoint 快照、重试与 OpenTelemetry 事件）；Strands 下沉到节点内充当推理与工具引擎，模型无关且自带会话管理以防上下文溢出。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:117-127]

配套三条机制：只在需要 LLM 判断的节点放 Agent；节点内 Agent 拥有聚焦的上下文与工具历史，避免单体 Agent 忘记指令；MCP 集成、steering controls、护栏与评估能力随节点接入图中，同时保留 LangGraph 的强路由。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:188-202] 同思路的 serverless 变体见 [[entities/serverless-langgraph-multi-agent-aws|另一实现]]。

### AgentCore 原语如何对上合规要求

runtime 把本地 Agent 代码转成云原生部署：框架无关（LangGraph/Strands 开箱即用），提供长时调查的扩展运行时、交互式低延迟与按需伸缩，并接管容器编排、网络与会话。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:233-239]

memory 是合规价值最高的一环：`AgentCoreMemorySaver` 在每次节点完成后把 checkpoint 自动写入 AgentCore 记忆，无需自建 DynamoDB 或自定义序列化；`AgentCoreMemoryStore` 自动抽取洞察、摘要与偏好供检索。创建记忆可指定 `eventExpiryDuration`（示例 90 天），调用传 `thread_id` 与 `actor_id`——留存期加参与者标识即可回放、可追责。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:308-325]

身份与网关原语本文未展开（另见 [[entities/aws-bedrock-agentcore-identity-security|AgentCore 身份与安全]]、[[entities/amazon-bedrock-agentcore-gateway-mcp-extension|AgentCore Gateway MCP]]），但入口函数已从 payload 取出 `session_id`/`actor_id`，可追溯性从契约层就设计。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:249-258]

可观测性侧，CloudWatch 与 X-Ray 捕获执行轨迹、工具调用与指标并提供看板；叠加 LangGraph 的 OpenTelemetry 事件后，可见性可从工作流下钻到单次 LLM 调用，即"告警为何被触发"的证据链。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:359]

### 市场监控场景的硬性约束与对应设计

**时间性事件流**：示例问题问某日 11:00 的 AAPL 尖峰，security_monitor 即单日活动分析 Agent，分析价格、成交量与逐笔成交；取数走预定义报表（如 `TradeActivity`）加按列名等值过滤器（`{"symbol": "AAPL", "date": "2024-03-15"}`），行数上限 1..10000——用报表 + 过滤器替代自由查询，时间窗口才可复现。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:74-108]

**告警可解释**：共享状态为每位专家预留独立 insights 槽位（security/broker/risk/intel），最后由 synthesizer 汇总；节点返回结构化状态增补而非自由字符串，"谁贡献了什么证据"分层可读。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:134-145]

**误报成本与确定性**：`route_analysts` 按 orchestrator 写入的 `required_agents` 与 `current_agent_index` 顺序推进，走完全部专家才交 synthesizer 再到 END——"由谁看、按什么顺序看"完全确定，只有节点内部推理不确定。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:154-162]

**防注入**：工具把"发现数据"与"取数"分开——`get_report_list` / `get_report_schema` 只列报表与列定义，`run_report` 仅接受经 schema 白名单校验的过滤器并拼装带绑定参数的 SQL，LLM 从不写原始 SQL。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:24]

### 协作拓扑：不只是主管派活

图中是四类专家节点（security/broker/risk/intel）加 orchestrator 与 synthesizer。`route_analysts` 同时注册为 orchestrator 与每个专家的条件边，目标集合覆盖全部专家与 synthesizer——这是"主管派活 + 同级交接"的混合拓扑（参见 [[concepts/agent-orchestration-patterns|编排模式]]、[[concepts/agent-role-specialization|角色专精]]）：任一专家都能把流程交给下一位专家，不必先回主管。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:166-180]

每个专家节点运行时即时新建 Strands Agent（独立 name、model、system_prompt、tools），从共享状态取分派任务，跑完推理 + 工具循环后写回结果并把 `current_agent_index` 加一；上下文因此被 [[concepts/multi-agent-context-isolation|节点级隔离]]，跨节点记忆由 LangGraph 状态层持有。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:202-231]

### 权衡与失败模式

- **不确定性只是被围起来**：路由确定，但专家输出与 synthesizer 的叙述仍由 LLM 生成，仍需人工把关（另见 [[concepts/long-running-agent-architecture|长时运行 Agent 架构]]）。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:123]
- **延迟与成本随专家数增长**：示例按索引串行推进专家，各专家模型配 16000 max_tokens、8000 thinking budget 并开 prompt caching；长时调查靠扩展运行时兜底。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:32-40]
- **恢复依赖 checkpoint，且要容忍残缺输出**：每节点后快照，人工介入或失败后可从精确 checkpoint 恢复；调用侧需跳过解析失败的 SSE 分片而非崩溃。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:117-125]
- **就绪态本身是异步资源**：记忆创建需轮询到 ACTIVE 且带 10 分钟上限，部署流程必须处理这类依赖。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:299-336]

## 实践启示

1. 编排层确定、推理层自由：把"要跑哪些步骤"写进图与路由函数（如按 `required_agents` 顺序推进），只让 LLM 决定节点内部的判断。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:154-162]
2. 把 checkpoint 当审计底座：每节点结束即落快照，配合 `thread_id`/`actor_id` 与 `eventExpiryDuration` 设定留存窗口，"告警为什么出现"才有可回放的证据。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:308-325]
3. 数据访问走"发现 → 取数"两段式：schema 发现与执行分离，执行侧白名单校验加参数化查询，LLM 永不拼原始 SQL。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:24]
4. 一个节点一个专家：独立 system prompt、工具集与上下文窗口，用图状态而非共享上下文传信息，避免指令漂移。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:196-215]
5. 部署前先算串行成本：专家串行加大 thinking budget 意味着延迟与费用随专家数增长，优先 prompt caching 与按需裁剪 `required_agents`。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:32-40]
6. 显式设计人在环：把 checkpoint 恢复点当作人工复核与签署的插入位，synthesizer 的结论进入监管报告前必须过人。^[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen.md:123]

## 相关实体

- [[entities/langgraph-state-machine-under-the-hood|LangGraph 底层原理]]
- [[entities/strands-agents|Strands Agents]]
- [[entities/agentcore-harness|AgentCore Harness]]
- [[entities/building-web-search-enabled-agents-with-strands-and-exa|Building web search agents with Strands and Exa]]
- [[entities/deep-agents-bedrock-agentcore-subagent-orchestration-aws|Deep Agents 子 Agent 编排]]
- [[entities/evaluating-ai-agents-production-blueprint-strands-agentcore|Evaluating AI agents production blueprint]]
- [[entities/amazon-bedrock-agentcore-gateway-mcp-extension|AgentCore Gateway MCP]]
- Multi-Agent Orchestration
- [[concepts/multi-agent-collaboration-patterns|Multi-Agent Collaboration Patterns]]

→ [[raw/articles/market-surveillance-agent-with-langgraph-and-strands-on-agen|原文存档]]
