---
title: "AI Teammates: How monday.com Runs Production AI Agents on Amazon Bedrock"
created: 2026-07-24
updated: 2026-09-20
type: entity
tags: [aws, bedrock, monday-com, production-agents, ai-engineering, eks, sns, sqs, agent-architecture, morphex, sphera]
sources: [raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# AI Teammates: How monday.com Runs Production AI Agents on Amazon Bedrock

## Overview

AWS blog post (2026-07-22) by Claudio Mazzoni, Ofek Dayan, Netanel Abergel, Moran Zilberstein, and Erez Drutin sharing monday.com's production architecture for running AI agents on Amazon Bedrock at scale. Nine in ten Builders use AI coding tools monthly; per-engineer PR throughput is up by more than half. ^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md]

## Three Levels of AI Engineering

- **L1 (Assistant)** — Engineers use AI as pair programmers: Cursor for fast reflexive work, Claude Code for heavy lifts; adoption nearly doubled YoY
- **L2 (Skills & Sub-agents)** — Teams build reusable agents for repeated work; engineers remain in the driver's seat; per-developer PR throughput stepped up by more than half
- **L3 (Multi-agent)** — Fully agentic; agents own delivery end-to-end while engineers orchestrate; agents take tasks from boards, communicate in Slack and monday, ship code alongside humans ^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md]

## Architecture: Sphera Agent System

The system uses seven AWS services: SNS, SQS, EKS, RDS, ElastiCache, EFS, and S3. AWS Secrets Manager handles per-session secrets; Amazon Bedrock handles model calls; the `monday-agent-sdk` runs inside each agent runner pod.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md]


**Event path:** External triggers → SNS → per-team SQS queues → monday Builders CoWORK (SQS consumers on EKS) → resolves agent ownership → routes to agent runner pod. Pub/sub gives retries/DLQs, back-pressure, durable replay, and concurrent fan-out.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md]


**monday-agent-sdk:** A thin wrapper around Claude Agent SDK providing provider neutrality (routes to Bedrock), cold-start cost optimization (warm node_modules), and a custom harness for evaluation, plugin composition, Slack/monday/GitHub integration, and output review standards. ^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md]

## State, Memory & Sessions

Three kinds of state with distinct stores:^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md]

- **Live state** (ElastiCache): current task, run cursor, distributed lock, heartbeat, agent/human message log — sub-millisecond reads, self-expiring keys
- **Sessions and memory** (EFS): each session is a directory (`/sessions/<id>/` with `repos/`, `secrets.json`, `messages/`; `/agents/<id>/` with `MEMORY.md` and `diary/YYYY-MM-DD.md`)
- **Durable records** (S3): final transcripts, snapshots, artifacts, evals keyed by session ID

## Five Production Retrofits

1. **Evals before model upgrades** — deterministic metrics (PRs merged, revert rate) + LLM-scored evals across 5 dimensions
2. **Memory is a file, not a vector store** — per-agent `MEMORY.md` + `diary/YYYY-MM-DD.md`; plain markdown, no embeddings
3. **Remote sandbox before human review** — PR auto-deploys to sandbox; tests and replayed production traffic run before review
4. **PR Guardrails** — every monday engineering standard automated as a reviewer; tens of thousands of PRs evaluated per month; ~1 in 5 PRs fails a standard
5. **Builders CoWORK** — monday boards as the shared state layer; agent tasks, status, blockers live on same monday board as the team's

## Morphex: Fully Autonomous Engineering Agent

Morphex is monday's first fully autonomous engineering agent. 19 of every 20 Morphex PRs merge automatically using a confidence score combining: deterministic Guardrails outcome, per-agent eval trajectory, per-(agent × repo × change-class) historical revert rate, and sandbox outcome. Approximately 3 in 10 PRs merged, ~3/4 with zero human edits, revert rate in low single digits. Guardrails stopped ~1/4 of agent PRs before reaching a human. ^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md]

## Source

> [AWS Machine Learning Blog](https://aws.amazon.com/blogs/machine-learning/ai-teammates-how-monday-com-runs-production-ai-agents-on-amazon-bedrock)

## 深度分析

### 1. The event substrate is what makes agents retryable, not merely triggered

The SNS → per-team SQS → CoWORK consumers on EKS → agent runner pod path is not plumbing; it is what makes long-running, flaky sessions survivable. The message, not the process, is the durable unit, so a run that dies mid-task is redelivered rather than left half-applied to a repository. monday names the four guarantees it will not trade away — retries and dead-letter queues out of the box, back-pressure when Amazon Bedrock throttles, durable replay (a day of events re-run against a patched build before promoting) and concurrent fan-out — which together are the agent-era equivalent of a transaction boundary, held deliberately outside the agent because the runtime is replaceable and "the harness stays".^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:48-60]

### 2. Session identity, not agent identity, is the tenancy boundary

Isolation is scoped to the session: per-session secrets in Secrets Manager, a per-session EFS directory, and one EKS pod per active session with per-session crash isolation.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:44-46] That is tighter than agent identity — two runs of one agent on different tickets share no repo state, no secrets and no blast radius — and it buys resumability, since a run picked up on another pod mounts the same EFS path.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:64-78] The bill lands in warm capacity: warm plugin caches keep the first model call under a second, KEDA scales on average active sessions, and repo caches are reused.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:108-110] Per-session tenancy is cheap to reason about and expensive to keep hot; compare the same tension in [[entities/shared-infrastructure-isolated-tenants-pool-model-multi-tenancy-with-amazon-bedrock-agentcore|pooled multi-tenant agent infrastructure]] and [[entities/bedrock-agentcore-secrets-manager-identity|Secrets Manager 与身份]].

### 3. L1/L2/L3 is a measurement ladder, not a maturity ladder

The L3 claim is not that agents are good enough, but that monday can quantify a human reviewer adding nothing on a class of change: Morphex's nineteen-in-twenty auto-merge is a threshold over four signals available the moment a PR opens — Guardrails outcome, per-agent eval trajectory for *this version*, per-(agent × repo × change-class) revert history, and sandbox outcome.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:158-165] Each input is the downstream artifact of an earlier retrofit, which is why L3 is gated on L1/L2 measurement maturity rather than on model capability.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:116-126] "Engineers orchestrate" also hides a capacity constraint rather than dissolving one: by late Q1 2026 the wall was human review, not code generation.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:142-144]

### 4. Retrofit cost sits in observability and state, not model choice

None of the five retrofits replaces the model, and monday's own counterfactual is the evidence: "we changed no model, no prompts, no human nudges, only the evals, and scores moved across every dimension".^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:116-118] The expensive work was turning every engineering standard into an automated reviewer with internal knowledge wired in through MCP servers, then running it across tens of thousands of PRs a month.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:128-132] Memory repeats the pattern — vector retrieval over past transcripts "worked badly" while a plain `MEMORY.md` plus a dated diary won, and monday says it over-invested in vector stores first. Model choice stays a call site the thin SDK wrapper keeps swappable; observability and state are the architecture.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:120-122]

### 5. Agent-owned delivery forces an org answer, not a tooling answer

Accountability is the part tooling cannot supply. monday reuses human infrastructure: agents hold real Slack, GitHub and monday identities under the same RBAC, so they can be tagged, code-reviewed or deactivated like teammates, and every PR — agent- or human-authored — passes the same Guardrails, so an agent's PR fails for the same reasons an engineer's does.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:30-34] Its most expensive failure mode was organizational — agents in silos opened PRs no team owned — and the fix was putting agent tasks, status and handoffs on the team's own board, so accountability stopped being a system problem.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:134-136] Audit is equally concrete: one Bedrock audit trail for every model call, and transcripts, snapshots, artifacts and evals keyed by session ID in S3.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:95-102]

### 6. Separate the portable pattern from the monday-shaped surface

Portable: queue-decoupled retryable sessions, the three-state split (live state in a cache, working memory on a shared POSIX filesystem, durable records in object storage), memory-as-file, evals before model upgrades, sandbox before human review, standards-as-automated-reviewer, and confidence-scored auto-merge.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:112-136] monday-specific: boards as the shared state surface, monday MCP servers as the substrate standards plug into, monday's auth and identity, the deploy pipeline its agents ship through — and the harness itself, which monday calls precisely too monday-specific to open-source.^[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md:177-186] Porting the surface without the evals, guardrails and identity plumbing reproduces the demo, not the discipline.

## 实践启示

**Put evals in on day one.** monday's first listed regret is that evals arrived in month nine, and the eval layer is also the input that later gates auto-merge — autonomy built before evals is unmeasurable. See [[concepts/evaluation-harness-design|Evaluation Harness Design]].

**Design the queue before the agent.** Retries, dead-letter queues, back-pressure against provider throttling and event replay are what let you promote a change by replaying yesterday's traffic instead of hoping; added after the agent, each becomes a retrofit.

**Make memory a file, not a retrieval system.** Start with a per-agent `MEMORY.md` plus a dated diary read at session start, on a filesystem the runtime already understands; reach for embeddings only when a written file demonstrably fails. See [[concepts/agent-memory-system-design|Agent Memory System Design]] and [[entities/claude-agent-sdk-skills-reusable-knowledge|Claude Agent SDK Skills]].

**Automate review standards before adding reviewers.** Standards-as-code catches roughly a quarter of agent PRs before a human ever looks, and converts approval into a machine-readable signal a confidence score can consume.

**Gate autonomy on the pre-filtered population.** A low-single-digit revert rate measured after Guardrails has already rejected about a quarter of agent PRs is the meaningful signal; a headline merge rate without that filter is not. Expect the next constraint to be attribution, not throughput. Related: [[entities/agent-autonomy-levels-l0-l5-addy-osmani-2026|Agent Autonomy Levels L0–L5]].

**Do not copy the surface.** The board/Slack/MCP surface and the closed `monday-agent-sdk` harness are what monday itself calls non-transferable, and per-session isolation plus the warm pool it requires have to be budgeted deliberately rather than inherited by default.

---
## 关联
→ [[raw/articles/ai-teammates-how-mondaycom-runs-production-ai-agents-on-amaz.md|原文存档]]
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

