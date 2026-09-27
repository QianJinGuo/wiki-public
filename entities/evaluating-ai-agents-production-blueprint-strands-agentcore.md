---
title: "Evaluating AI Agents: A Production Blueprint with Strands and AgentCore"
created: 2026-07-24
updated: 2026-09-27
type: entity
tags: [aws, bedrock, agent-evaluation, strands-agents, agentcore, ai-agents, llm-as-judge, production, motorway, eval-framework]
sources: [raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Evaluating AI Agents: A Production Blueprint with Strands and AgentCore

## Overview

AWS blog post (2026-07-23) by Amit Deol, Hin Yee Liu, and Ryan Cormack presenting a production evaluation blueprint for AI agents using the Strands Agents SDK (`strands-agents-evals`) for build-time testing and Amazon Bedrock AgentCore Evaluations for production monitoring. Uses Motorway's dealer stock search agent as a worked example. ^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]

## Three-Layer Build-Time Assessment

The evaluation framework operates across three distinct layers with pass/fail thresholds:^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]


**Layer 1: Tool Usage (>95% threshold)** — Validates correct tool selection and parameter passing. Uses deterministic code-based graders (`ToolSelectionGrader`, `TrajectoryOrderGrader`) to verify which tools were called and the call sequence. Example: "Diesel vehicles from £7,000 to £20,000" should use `search_vehicles` with typed filters. ^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]

**Layer 2: Reasoning (>85% threshold)** — Assesses logical decision-making using LLM-as-judge evaluators (`HelpfulnessEvaluator`, `TrajectoryEvaluator` from strands-agents-evals). Ensures the agent's reasoning holds together; agents arriving at the right response through illogical reasoning will fail unpredictably.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]


**Layer 3: Output Quality (>90% threshold)** — Measures response helpfulness, accuracy, and actionability using LLM-as-judge evaluation (`OutputEvaluator`, `GoalSuccessRateEvaluator`).^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]


All three layers must pass before deployment.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]


## strands-agents-evals Framework

The `strands-agents-evals` framework provides three primitives:^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]

- **Experiment**: A collection of test cases run against the agent
- **Case**: Input query, expected output, and expected tool trajectory
- **Evaluator**: Scoring logic (deterministic or LLM-based)

Three grader types:
- **Code-based deterministic** (Layer 1): Fast, cheap, reproducible — measures tool selection, parameter passing, trajectory ordering
- **LLM-as-judge (Claude Sonnet 4.6)** (Layers 2-3): Flexible but non-deterministic — measures reasoning quality, output helpfulness, goal success
- **Human review** (Calibration): Expensive, used to calibrate LLM judge prompts — handles edge cases and safety

## Handling Non-Determinism

The `run_all_layers()` function accepts a `num_trials` parameter. Two key metrics:^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]

- **pass@k**: likelihood of succeeding at least once in k attempts
- **pass^k** (pass to the kth): probability of succeeding in k consecutive trials — more important for customer-facing agents

Multi-turn conversation testing uses `ActorSimulator` to generate realistic multi-turn interactions and `InteractionsEvaluator` to score context retention. ^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]

## Production Monitoring with AgentCore Evaluations

Two complementary modes:^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]

- **On-demand evaluation**: analyzes specific agent interactions by selecting spans from CloudWatch logs — useful for debugging
- **Online evaluation**: automatically samples live traffic (1-5% sampling recommended) with up to 10 evaluators

Built-in evaluators: `Builtin.Helpfulness` (TRACE), `Builtin.GoalSuccessRate` (SESSION), `Builtin.ToolSelection` (TOOL_CALL), `Builtin.Correctness` (TRACE). Custom evaluators (e.g., `DataFreshnessEvaluator`, `SafetyGuardrailEvaluator`, `DealerDataScopingEvaluator`) handle domain-specific constraints.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]


## Deployment Pipeline with Evaluation Gates

Five-phase pipeline: Build-time evaluation → Staging validation (on-demand AgentCore) → Shadow mode (4h minimum, 2% deviation threshold) → A/B testing (5% traffic) → Production rollout (100% traffic with continuous online evaluation). Each phase has defined thresholds that block deployment on failure.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]


## Results

After implementing the pipeline: Tool selection accuracy 87%→98%, Task completion rate 82%→96%, Context retention 71%→94%, Production incidents 12→2 per month, Mean time to detect from hours to minutes. ^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md]

## Source

> [AWS Machine Learning Blog](https://aws.amazon.com/blogs/machine-learning/evaluating-ai-agents-a-production-blueprint-with-strands-and-agentcore)

---
## 深度分析

### Layered grading maps cost to failure mode, not just accuracy

The three-layer design is really an economic stratification: deterministic code graders run first because they are fast, cheap, and reproducible, and only then do expensive LLM-as-judge layers (Claude Sonnet 4.6) score reasoning and output quality. This ordering means the 95%/85%/90% thresholds are not arbitrary quality bars — they encode which failure modes are cheap to catch mechanically (wrong tool, wrong parameters, wrong call order) versus which genuinely require semantic judgment (illogical reasoning that still lands on a right answer). The blueprint's counterintuitive note that grading *outcomes* catches more issues than grading *trajectory* reinforces that trajectory checks are a cheap safety net, not the main quality signal.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md:44-52] Human review appears only as a calibration instrument for the LLM judge itself — a third-order use of human labor that keeps the expensive resource out of the per-run path entirely.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md:50-50]

### pass^k is the correct reliability lens for customer-facing agents

The framework imports pass@k and pass^k from the code-generation research community and deliberately inverts their priority for production agents: pass@k ("succeeds at least once in k tries") fits exploration tasks, while pass^k ("succeeds in k consecutive trials") matches user expectations of consistency. The arithmetic makes the stakes concrete — a 75% per-trial success rate yields only ~42% reliability across three consecutive runs (0.75³), meaning the majority of users would hit a failure in a short session.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md:76-85] Gating deployment on pass^k with `num_trials=5` converts non-determinism from an anecdotal annoyance into a measurable, threshold-enforced release criterion.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md:175-175]

### The evaluation suite is designed to be grown, not authored

Motorway's suite went from 50 seed cases to 150 in three months, each originating from a real production interaction flagged by monitoring. This inverts the usual "write a big test suite up front" pattern: the initial 20–50 cases are deliberately thin scaffolding, and the production monitoring layer (online evaluation sampling 1–5% of live traffic) acts as the case factory. Negative cases get equal billing — an eval suite that only checks what the agent *should* do invites one-sided optimization, so profile queries must assert the search tool is *not* called.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md:91-99] Multi-turn testing via `ActorSimulator` extends this to conversational coherence, catching context drift and filter-accumulation errors invisible to single-turn suites.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md:103-111]

### Shadow mode as the bridge between synthetic and real distribution

The five-phase pipeline's shadow phase (≥4 hours, 2% deviation threshold, zero user impact) exists because synthetic staging traffic systematically misses three failure classes: timeout behavior under concurrent load, domain vocabulary absent from test cases, and tool-call-ordering latency spikes under real traffic patterns.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md:177-185] The two-mode production monitoring design (on-demand span inspection for debugging, online sampling for continuous health) then closes the loop: every detected production failure becomes a regression test case, which is the mechanism that compounds evaluation quality over time rather than letting it decay.^[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md:189-197]

## 实践启示

1. **Match grader type to failure mode.** Use deterministic code graders for tool selection, parameters, and call order (cheap, reproducible); reserve LLM-as-judge for reasoning and output quality. Don't pay LLM-judge cost to check what a unit test can.
2. **Gate on pass^k, never single trials.** Set `num_trials` ≥3 and require consecutive-trial success. Remember 75% per-trial ≈ 42% over three runs — single-trial pass rates flatter non-deterministic agents.
3. **Start with 20–50 test cases and let production grow them.** Include happy path, edge cases, AND refusal cases; convert every detected production failure into a new regression case so the suite compounds.
4. **Add negative trajectory assertions.** Explicitly test that certain queries must NOT call certain tools (e.g., profile query should not hit search). One-sided evals produce one-sided optimization.
5. **Run shadow mode ≥4 hours at a 2% deviation threshold before any live traffic.** It catches concurrency timeouts, missing domain terminology, and real-traffic latency patterns that staging validation cannot.
6. **Start online evaluation at 1% sampling and scale up.** Cap built-in evaluators at 10, add custom `Evaluator` subclasses (freshness, safety, scoping) for domain constraints built-ins don't cover, and watch evaluator cost before widening sampling.

---
## 关联
→ [[raw/articles/evaluating-ai-agents-a-production-blueprint-with-strands-and.md|原文存档]]
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

