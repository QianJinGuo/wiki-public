---
title: "HarnessTax: How Much Does the Harness Matter for Coding Agents?"
created: 2026-09-22
updated: 2026-11-09
type: entity
tags: [harness, agent, evaluation, cost, coding-agent]
sources: [raw/articles/arena-ai-harness-tax-coding-agents]
confidence: 0.7
---

# HarnessTax: How Much Does the Harness Matter for Coding Agents?

## 摘要

Arena.ai 系统研究 21 个 model-harness 组合（Claude Code/Codex CLI/Pi × 2 benchmarks）：harness 选择对任务成功率影响有限但显著影响成本——harness 是成本变量而非能力变量^[raw/articles/arena-ai-harness-tax-coding-agents.md]

## 核心要点

- v*c=64（value=8, confidence=8, stars=4），newsletter ingest 2026-09-22
- 详细分析见原文存档

相关：[[concepts/coding-harness-engineering]]
相关：[[concepts/agent-harness-engineering-paradigm]]

## 深度分析

### What was measured, and how

Arena.ai profiled 21 model–harness pairs spanning 7 models and 3 harnesses (Claude Code, Codex CLI, Pi) on 30 randomly sampled tasks from each of SWE-bench Lite and Terminal-Bench 2.0, running every pair three times per task with each attempt capped at 100 agent turns. Success is scored by each benchmark's official evaluator; 95% confidence intervals come from 10,000 bootstrap resamples over task averages, and token costs use a single fixed direct-API price list (September 1, 2026) applied to every harness, so model pricing cannot contaminate cross-harness comparisons. For SWE-bench Lite the containers were network-isolated, default web tools were disabled in Claude Code and Codex, and hosted tool declarations were rejected at the API request level; Kimi K3 was served via Fireworks AI in its single native thinking mode. Profiling traces are slated for public release.

### Harness moves cost, not correctness

The average harness effect on success rate stays within ±2% on SWE-bench Lite and within about ±5% on Terminal-Bench 2.0 — but cost spans up to 5x for the same model at similar success. Geometric means of cost ratios across shared models put Claude Code at ~2.0x Pi and ~1.6x Codex on SWE-bench Lite, and ~1.5x Pi on Terminal-Bench 2.0. The sharpest anchor: Claude Fable 5 solves 97.8% of SWE-bench Lite attempts in Claude Code vs 96.7% in Codex and 96.7% in Pi — a +1.1-point success delta for exactly 2x the spend ($1.33 vs $0.67 per task). Elsewhere on the frontier, GPT-5.6 Luna offers the lowest cost on both benchmarks, Claude Fable 5 is the top SWE-bench Lite scorer, and open-weight Kimi K3 sits near the Pareto frontier next to GPT-5.6 Sol on SWE-bench Lite and just below it on Terminal-Bench 2.0. The authors name the gap the "harness tax": money paid for essentially identical quality when accepting an agent's default harness without comparing alternatives.

### The tax accrues per token, not per turn

Turn counts do not explain the gap: for Fable 5 on SWE-bench Lite, Pi and Claude Code average 15.4 and 15.3 turns per attempt — the same amount of work — yet Claude Code bills about twice as much for a 1.1% higher success rate. Spending diverges starting at the first model call: across all seven models, Claude Code's mean initial context is over 10x Pi's, driven by longer system instructions and much larger tool schemas. That initial weight then flows into every subsequent turn, subject to caching and generated-token volume, which makes the first call the cheapest place to audit the tax ([[concepts/context-window-economics]]).

### Minimal tools are Pareto-competitive; native pairings are not optimal

Pi reaches the Pareto frontier on both benchmarks while providing only four tools — read, write, edit, bash ([[entities/pi-agent-framework]]) — evidence that a minimal tool surface can beat feature-rich harnesses on cost per solved task, and that open-source harness research is viable with existing models without proprietary access or co-training. Meanwhile, provider-native pairing is no guarantee of the best result: across six Anthropic and OpenAI models on both benchmarks, an alternative harness achieved the highest observed success rate in 9 of 12 comparisons. Sonnet 4.6 scores 68.9% in Codex vs 66.7% in Claude Code on SWE-bench Lite at similar cost; GPT-5.6 Sol reaches 83.3% in Pi vs 78.9% in Codex on Terminal-Bench 2.0 at about half the cost ($0.42 vs $0.76). Model capabilities are portable and generalize across harnesses ([[entities/model-harness-fit-agent-harness]]), so an official pairing is a default hypothesis to test, not a conclusion — richer harness features may still pay off for other models, workloads, or interaction settings.

## 实践启示

1. Treat harness selection as a first-class procurement decision: the same model can cost up to 5x more for near-identical success rates, so profile at least one alternative harness ([[entities/swe-bench-agent-evaluation]] is a natural first venue) before standardizing on a default agent.
2. Report cost per solved task, not raw token price or leaderboard accuracy — a ~2x harness multiplier dwarfs most model-pricing deltas and is invisible in success-rate-only evaluations.
3. Audit initial context size first: a >10x first-call gap between harnesses is the cheapest lever to inspect, and trimming system prompts and tool schemas attacks the tax before any capability-level work.
4. Start from a minimal tool surface (read/write/edit/bash) and add harness features only when an ablation shows they pay for themselves — harness complexity should be treated as an empirical trade-off, not a default good.
5. Do not assume provider-native harnesses are optimal: 9 of 12 provider-model comparisons were won by an alternative harness on success rate, so run cross-harness pairings when selecting or renewing a model.
6. Build cost metrics into evaluation pipelines ([[concepts/evaluation-harness-design]]) and cost-optimization reviews ([[concepts/ai-cost-optimization-framework]]): with success confined to a ±2–5% band while cost spans up to 5x, any eval measuring only accuracy is structurally blind to the largest variable.

→ [[raw/articles/arena-ai-harness-tax-coding-agents|原文存档]]
