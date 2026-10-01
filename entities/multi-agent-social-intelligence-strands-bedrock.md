---
title: "Multi-agent social intelligence with Strands Agents and Amazon Bedrock AgentCore"
created: 2026-07-24
updated: 2026-10-01
type: entity
tags: [strands-agents, bedrock, agentcore, multi-agent, swarm, graph, orchestration, social-intelligence, thradai, aws]
sources: [raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz]
confidence: 0.75
vxc: 49
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Multi-agent social intelligence with Strands Agents and Amazon Bedrock AgentCore

Thrad.ai built a multi-agent social intelligence system using Strands Agents framework on Amazon Bedrock AgentCore. The system discovers trending launches and buying-intent signals, enriches prospect profiles, scores prospect-trend pairs, and generates personalized outreach emails. ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

## Agent Architecture

The system uses four specialized agents: ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

| Agent | Responsibility | Data Sources |
|-------|---------------|--------------|
| **Trend Research** | Discovers trending launches and buying-intent signals | Hacker News, YouTube, dev.to, ProductHunt, Reddit, Stack Overflow |
| **Search Specialist** | Enriches prospect profiles with context | Wikipedia, GitHub, Lobste.rs, Stack Overflow |
| **Analysis** | Scores prospect-trend pairs (0-100) | Scoring engine, ICP matcher, Claude Sonnet 4.6 on Bedrock |
| **Email Generation** | Drafts personalized outreach | Brand knowledge retrieval, lead storage |

Scoring relies on **signal triangulation**: a prospect needs correlated evidence from at least two independent sources. The Analysis Agent uses five weighted criteria: topical alignment (25%), timing relevance (20%), engagement potential (20%), intent signals (20%), and data quality (15%). ICP matching adds up to 10 bonus points for developer tools with open source presence and B2B focus. Temporal decay: signals under 24 hours old get 1.5x weight, signals over 7 days get 0.5x. ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

## Swarm vs Graph Orchestration

Strands Agents provides two orchestration patterns. Thrad.ai built and benchmarked both against 50 prospects: ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

| Metric | Swarm | Graph |
|--------|-------|-------|
| Avg latency per prospect | 45s | 32s |
| P95 latency | 78s | 38s |
| Avg tokens per prospect | ~12,000 | ~8,500 |
| Email relevance (human-rated 1-10) | 8.2 | 7.6 |
| Cost per prospect (est.) | ~$0.08 | ~$0.06 |

**Key findings**: Swarm produced higher-quality emails (8.2 vs 7.6) because agents looped back for more context when data was sparse. Graph cost 25% less per prospect with tighter latency bounds. For a 1,000-prospect batch, Graph saves ~3.6 hours and $20 in token costs. ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

### Swarm Pattern
Agents pass control dynamically using a `handoff_to_agent` tool with shared working memory. Configurable safety bounds include `max_handoffs`, `execution_timeout`, and `repetitive_handoff_detection_window` to prevent agent ping-pong. Best when prospect complexity varies and agents benefit from re-engaging earlier stages. ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

### Graph Pattern
Agents follow a fixed directed workflow with parallel entry points, all-dependencies-complete gating, and conditional edges. Trend Research and Search Specialist run in parallel; Analysis waits for both to finish; Email runs only if score >= 60. Best for repeatable workflows where auditability matters. ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

## Bedrock AgentCore Deployment

Production deployment uses four Amazon Bedrock AgentCore managed services: ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

- **Runtime**: Hosts agents in isolated microVMs with IAM authentication and lifecycle controls (15-min idle timeout, 8-hour max lifetime)
- **Gateway**: Single MCP endpoint for nine tools; agents discover tools dynamically via Strands `MCPClient`
- **Memory**: Short-term context within sessions, long-term semantic data across sessions; agents degrade gracefully without it
- **Observability**: Distributed traces via OpenTelemetry with span-level latency and token counts; integrates with CloudWatch

A key finding: YouTube API calls accounted for 40% of total latency, leading the team to add `get_with_retry` with exponential backoff to HTTP calls. ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

## Governance & Safety Controls

Three-level guardrail system: ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

1. **Policy gates via conditional edges**: Analysis-to-Email edge checks relevance score; prospects below 60 are logged but skipped
2. **Scoped tool access**: Each agent receives only the tools it needs; agents cannot invoke tools outside their scope
3. **Swarm safety bounds**: Repetitive handoff detection stops loops; `max_handoffs` and `execution_timeout` cap autonomous behavior

## Practical Guidance

1. **Intent signals beat passive trends**: Adding Reddit intent detection increased prospects scoring above 80 by 22%. A prospect asking "What tool should I use for X?" converts at higher rates than one trending passively.
2. **Temporal decay prevents stale outreach**: Signals under 24 hours old get 1.5x weight; signals over 7 days get 0.5x.
3. **Pick pattern based on the job**: Swarm wins on quality when data is sparse; Graph wins on cost and predictability for batch work. Run both in the same code base switched by a configuration flag.
4. **Build retry logic for external APIs**: YouTube API calls were 40% of total latency — use exponential backoff.

## 深度分析

### 信号三角化不只是质量控制，更是成本控制

Thrad.ai 的信号三角化（signal triangulation）设计把「要求至少两个独立来源的关联证据」前置为过滤闸门：`check_existing_leads` 先跳过管道内已有线索，一条只有 Hacker News 热度、没有 Reddit 讨论、没有 Stack Overflow 活动和 GitHub star 的帖子大概率是推广冲量，系统会在花 token 分析之前就把它滤掉。Reddit 工具扫描五个 subreddit，用关键词模式把帖子分为四类意图（求推荐、竞品抱怨、产品发布、购买意图），交叉信号让「HN 发布 + Reddit 求工具帖」的候选得分更高。每个 agent 拥有单一职责、单一工具集和一个 Pydantic 校验的输出契约——错误形状的数据在进入下一个 agent 之前就被拦截。这套设计的本质是把 LLM 调用预算集中花在多源交叉验证过的信号上。 ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

### 双模式编排的真实取舍：质量 vs 成本 vs 可预测性

实测数据揭示了 [[concepts/agent-orchestration-patterns]] 中两种模式很少被量化的一面：Swarm 换手推理开销换来了 8.2 vs 7.6 的邮件相关性评分，代价是 token 翻倍（~12,000 vs ~8,500）和 P95 尾延迟近两倍（78s vs 38s）；Graph 固定 DAG 省下 25% 成本，但无法动态回环——agent 缺上下文时必须在图定义里显式加 feedback edge。Thrad.ai 的答案不是二选一，而是按任务分流：夜间批量跑 Graph，高价值线索的每周深挖跑 Swarm，同一代码库用配置开关切换。这与 [[concepts/multi-agent-collaboration-patterns]] 的「模式服务于负载形态」结论一致，但给出了少见的同负载 head-to-head 数字。 ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

### 可观测性驱动的性能调优闭环

这次部署最能复用的是「trace → 定位 → 修复」闭环：OpenTelemetry span 级延迟数据显示 YouTube API 占总延迟的 40%，团队据此给 HTTP 调用加上指数退避重试（`get_with_retry`）。真实 run 的 walk-through 同样有信息量：Prospect A 得 88 分靠的是 HN + Reddit + dev.to 三源信号、GitHub star 命中 ICP、信号全在 48 小时内吃到 1.5x 时间权重；Prospect C 低于 60 分阈值被条件边直接跳过，省下约 3,000 token——Graph 模式 30 分钟内跑完 50 个候选。AgentCore Memory 是可选项，agent 在没有记忆时优雅降级，这与 [[concepts/agent-memory-architecture]] 中「记忆是增强而非依赖」的分层观点相合。 ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

### 从销售智能到通用多智能体模式

文章结尾把同一架构外推到三个场景：竞争情报（把发现工具换成竞品监控，同一多信号融合检测发布与行业变化）、候选人寻源（GitHub 贡献、Stack Overflow 活跃度、dev.to 文章都是强候选信号）、内容策展（意图信号识别受众当下关心什么）。三个外推共享同一个骨架：多源信号采集 → 交叉验证打分 → 条件门控的产出动作。这个骨架与 [[concepts/llm-observability-4-layer-model]] 的分层观测和 [[concepts/agent-security-architecture]] 的最小权限工具授予组合起来，就是一套可迁移的生产级多智能体参考架构。 ^[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz.md]

## 实践启示

1. **先过滤后分析**：`check_existing_leads` + 信号三角化在花 token 之前过滤推广噪音；两条腿缺一不可——去重省重复分析，交叉验证省假信号分析。
2. **在自己负载上跑双模式基准**：不要照搬通用结论。同 50 个候选上 Swarm/Graph 的质量-成本差（8.2 vs 7.6、$0.08 vs $0.06）只有自己 benchmark 才知道落在哪个方向；用配置开关保留切换能力。
3. **Swarm 安全参数是必选项不是可选项**：`repetitive_handoff_detection_window=8` + `min_unique_agents=3` 强制前向进展，否则两个 agent 可以无限 ping-pong 烧 token；`max_handoffs` 和 `execution_timeout` 封顶自治行为。
4. **把人类审批建模为图节点**：条件边挡住低分候选只是第一层；在同一位置插入 review 节点即可变成 human-in-the-loop 审批门，策略演进而不用改代码结构。
5. **用 span 级 trace 定位外部 API 尾延迟**：40% 延迟来自单一数据源（YouTube）说明外部依赖才是主要瓶颈，不是模型调用；先上 OpenTelemetry 再谈优化。
6. **时间衰减应成为意图类信号的标准配置**：24 小时内 1.5x、7 天以上 0.5x，让「昨天的 Stack Overflow 热度开启对话、上个月的是噪音」成为打分函数的一部分而非人工判断。

## Related Entities

- [[entities/strands-agents-high-performance-genai-systems]] — Strands Agents + NVIDIA NIM + Bedrock AgentCore
- [[entities/hands-free-first-notice-of-loss-using-strands-agents-and-ama]] — Strands Agents insurance claims intake
- [[entities/building-enterprise-level-with-bedrock-agentcore-and-strands]] — Enterprise search with Strands

→ [[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz|原文存档]]
