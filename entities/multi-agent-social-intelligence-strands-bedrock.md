---
title: "Multi-agent social intelligence with Strands Agents and Amazon Bedrock AgentCore"
created: 2026-07-24
updated: 2026-07-24
type: entity
tags: [strands-agents, bedrock, agentcore, multi-agent, swarm, graph, orchestration, social-intelligence, thradai, aws]
sources: [raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz]
confidence: 0.75
vxc: 49
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

## Related Entities

- [[entities/strands-agents-high-performance-genai-systems]] — Strands Agents + NVIDIA NIM + Bedrock AgentCore
- [[entities/hands-free-first-notice-of-loss-using-strands-agents-and-ama]] — Strands Agents insurance claims intake
- [[entities/building-enterprise-level-with-bedrock-agentcore-and-strands]] — Enterprise search with Strands

→ [[raw/articles/multi-agent-social-intelligence-with-strands-agents-and-amaz|原文存档]]
