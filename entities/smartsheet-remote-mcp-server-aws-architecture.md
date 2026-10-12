---

title: "How Smartsheet built a remote MCP server on AWS"
created: 2026-07-24
updated: 2026-10-12
type: entity
tags: [mcp, aws, agent, server-architecture, enterprise, smartsheet]
sources:
  - raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# How Smartsheet built a remote MCP server on AWS

> → [[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws|原文存档]]

## Overview

Smartsheet, an enterprise work management platform, built a remote [[mcp-agent-external-ecosystem-integration-guide-2026|Model Context Protocol (MCP)]] server on AWS to give AI agents structured access to enterprise Smartsheet data. The server connects AI assistants like Amazon Quick and Claude Desktop to Smartsheet's capabilities through natural language, enabling analysis, task updates, sheet creation, workspace management, and autonomous agent workflows. Since launch, Smartsheet has saved over 3 billion tokens through AI-specific optimizations. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## Architecture

One MCP layer serves both internal and external agents — Smartsheet's own Smart Assist and externally connected AI clients run on the same infrastructure with the same tools, optimizations, and intelligence stack. The key AWS services in the data path are: ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

- **AWS Fargate for Amazon ECS** — stateless server containers for the MCP server
- **Amazon Kinesis Data Streams + Amazon Managed Service for Apache Flink** — change-event ingestion into Amazon S3
- **Amazon Bedrock + Amazon Neptune** — LLM inference and knowledge graph for cross-project insights

The architecture flow is: AI clients → API gateway layer (AWS WAF, AWS Shield, ALB, OAuth validation) → MCP Server on AWS Fargate → Domain Services (transactional APIs) → Intelligence Layer (Amazon Neptune, Databricks) with change events streaming through Kinesis and Apache Flink into S3 following the medallion architecture. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## Deployment and Scaling

AI traffic differs from conventional request patterns — agents autonomously orchestrate bursts of tool calls followed by quiet reasoning periods. Smartsheet uses **AWS Fargate for ECS** with target-tracking Auto Scaling policies combining traffic volume and compute utilization. Compute-aware scaling is critical because each request involves server-side LLM-optimized serialization. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

Deployments use **Amazon ECR** with CI/CD pipeline safety nets. ECS deployment circuit breakers detect failing containers and auto-revert. Rollouts follow the AWS Well-Architected principle of reducing impact radius (smallest region first), with automated end-to-end tests and canary tests every 15 minutes. The pattern follows the [[aws-health-analytics-ai-agents-mcp|AWS Guidance for Deploying MCP Servers]]. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## Security & Governance

**Governance** is built into the tool framework itself — access control, error handling, and audit trails ship with every tool by default. Access is tiered per organization (global AI access, non-destructive only, or full write/destructive). Tools carry MCP annotations like `readOnlyHint` and `destructiveHint` for automatic confirmation flows. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

**Security** follows defense-in-depth: AWS WAF and AWS Shield at the edge, private subnets in a VPC, mutual TLS (mTLS) for service-to-service calls, and an OAuth2 proxy rejecting unauthenticated requests. Layered rate limiting through AWS WAF provides blanket protection, per-user metering via identity headers, and path-specific controls for expensive operations — following the three most important AWS WAF rate-based rules pattern. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## Observability

The server emits OpenTelemetry signals (logs, traces, metrics) across the full request lifecycle, capturing user, organization, tool name, and outcome per invocation. Logs stream through Amazon Kinesis into Amazon OpenSearch Service, with infrastructure metrics in Amazon CloudWatch. Datadog provides per-tool APM visibility, and PagerDuty handles incident routing. Every invocation emits structured analytics through Amazon SQS into the Intelligence Layer for continuous optimization feedback. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## Testing Non-Deterministic AI Workflows

MCP tool responses pass through an LLM before reaching the user, introducing a non-determinism layer that changes what "correct" means for testing. Smartsheet invests heavily in end-to-end workflow tests that include the LLM in the loop, simulating realistic business scenarios (creating workspaces, writing data, querying results). These run in the CI/CD pipeline (GitLab CI on AWS) and continuously as canary tests against each production AWS Region. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## AI Optimizations

Smartsheet optimizes at three levels for AI consumption, saving 3+ billion tokens: ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

1. **Progressive disclosure** — Each tool response targets a token budget. The server dynamically calculates how many rows fit based on column count and data density. Metadata fields (`is_sampled`, `rows_in_sheet`, `rows_actual`, `filters_applied`) tell the model whether it has the full picture or needs to narrow with filters. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

2. **Schema-driven tool contracts** — Each tool publishes strict JSON Schema through MCP's tool discovery, generated from Pydantic models. Parameters are constrained to valid enums, column names validated against actual sheets before execution, and mismatches return structured errors with valid options. This prevents hallucinated parameters at the boundary. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

3. **Token-efficient serialization** — A proprietary serialization format that reduces token count by 35–47% on data-heavy responses. Key names appear once instead of repeating per row, and structural syntax is replaced by delimiters that tokenize more efficiently. On a 33-item filtered query: ~3,900 tokens vs ~6,000 for equivalent JSON. At 1,000 rows the gap widens further. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## Future Directions

In the first four weeks after GA, Smartsheet saw over 87% week-over-week user growth. Future plans include resources that shape themselves to users, autonomous agents on workflows, and a routing layer for specialist reasoning handoffs. The team continues to adopt AWS capabilities like [[amazon-bedrock-agentcore-harness-ga|Amazon Bedrock AgentCore]] as the protocol evolves toward elicitations, MCP Apps, and Tasks. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## 深度分析

### One layer, every client: the build-once economics of a shared MCP surface

The most consequential decision in Smartsheet's design is easy to overlook: internal and external agents run on identical infrastructure. Smart Assist and a Claude Desktop user hit the same tools, the same serialization, the same governance stack. That parity converts the MCP server from a side project into a distribution layer — every optimization (progressive disclosure, schema validation, token-efficient serialization) is amortized across all agentic clients the moment it lands. The [[mcp-agent-external-ecosystem-integration-guide-2026|Model Context Protocol (MCP)]] framing as "distribution layer, with intelligence at the connection point" also explains the roadmap: resources that adapt per customer and specialist-to-specialist reasoning handoffs require no re-deployment because the surface is already shared. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

### Token cost treated as a first-class engineering constraint

The 3+ billion tokens saved is not a caching win; it is a redesign of what a tool response *is*. Progressive disclosure moves the budgeting decision server-side — the server computes how many rows fit a token budget from column count and density, while metadata (`is_sampled`, `rows_in_sheet`, `rows_actual`, `filters_applied`) forms a contract that lets the client decide when to narrow. The deeper insight is the split of responsibilities: the server guarantees bounded responses, and the model orchestrates follow-ups rather than receiving everything at once. Because most agent loops today call tools one at a time with reasoning in between, each response must be self-contained — that is why serialization (35–47% reduction, ~3,900 vs ~6,000 tokens on a 33-item query) and grounding-by-schema were built as complements rather than alternatives. See [[concepts/harness-tool-design-evolution|harness tool design evolution]] for the same tool-contract pattern in agent frameworks. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

### Agent traffic breaks conventional operational assumptions

The operational sections reveal a consistent theme: assumptions baked into standard practice fail under agent traffic. Burst-and-pause load patterns (multiple tool calls per second, then silence while the model reasons) make target-tracking scaling on compute + traffic mandatory — the server does LLM-optimized serialization per request, so it is compute-bound, not a proxy. IP-based rate limiting collapses because a single question fans out into many tool calls and enterprise users share proxies, forcing identity-header-based per-user metering layered with edge and path-specific controls. Observability inherits the same problem: one user request becomes a chain of tool calls, so Smartsheet is moving to agent-first identity and tracing that correlates context across chains, on top of OpenTelemetry signals and per-tool APM. This aligns with the [[concepts/llm-observability-4-layer-model|LLM observability layered model]]. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

### Non-determinism moves "correct" upstream and into the boundary

MCP responses pass through an LLM before the user sees them, which redefines testing: a perfectly valid tool response can still produce a wrong interpretation. Smartsheet's answer is two-fold — schema-driven contracts catch hallucinated parameters at the boundary (enums, column names validated against real sheets, structured errors listing valid options), and LLM-in-the-loop end-to-end workflow tests evaluate whether the model's *interpretation* is sensible, run both in CI/CD and as 15-minute canaries per production region. Governance embeds into the same boundary: tiered per-org access, `readOnlyHint`/`destructiveHint` annotations driving automatic confirmation flows, and the invariant that UI permissions equal MCP permissions. The pattern matches [[concepts/agent-security-architecture|agent security architecture]]: enforcement at the tool boundary, not in prompts. ^[raw/articles/how-smartsheet-built-a-remote-mcp-server-on-aws.md]

## 实践启示

1. **Serve internal and external agents on one MCP deployment.** Build once so every client inherits tools, optimizations, and governance immediately — don't fork an internal-only path.
2. **Make token budgets a server-side responsibility.** Cap each tool response, decide sampling in the server, and return metadata (`is_sampled`, `rows_actual`, …) so the client can orchestrate narrowed follow-ups itself.
3. **Validate tool parameters at the boundary with strict schemas.** Constrain enums, check column names against the actual sheet, and return structured errors with valid options instead of failing silently — this kills hallucinated calls cheaply.
4. **Meter agent traffic by identity, not IP.** Layer rate limiting: edge-wide (WAF), per-user via identity headers, and path-specific caps for expensive operations.
5. **Scale for burst-and-pause patterns and compute-bound requests.** Target-tracking auto scaling on both traffic and utilization; assume each request does real server-side work, not just proxying.
6. **Test the interpretation, not just the response.** Include the LLM in end-to-end workflow tests in CI, plus high-frequency per-region canaries; and emit structured analytics per invocation so production usage prioritizes tool work.

## Key Takeaways

- One MCP infrastructure serves both internal (Smart Assist) and external (Amazon Quick, Claude Desktop) AI agents
- Enterprise governance (tiered access, audit trails, readOnlyHint/destructiveHint) is built into the tool framework
- 3+ billion tokens saved through progressive disclosure, schema validation, and optimized serialization (35-47% reduction)
- AI traffic requires burst-aware scaling (target-tracking on compute + traffic) and layered rate limiting (edge + per-user + path-specific)
- Testing must include the LLM in the loop due to non-deterministic interpretation layers
