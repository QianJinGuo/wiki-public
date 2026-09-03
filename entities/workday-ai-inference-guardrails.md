---

title: "Workday Inference Engine Built-in Guardrails - Enterprise AI Safety Infrastructure Path"
type: entity
created: 2026-06-30
updated: 2026-08-29
source: "[[raw/articles/workday-ai-inference-guardrails]]"
tags: [agent, guardrails, inference, enterprise-ai, MCP, safety, workday, infrastructure]
provenance_state: inferred
confidence: 0.80
provenance_state: extracted
review_value: 7
review_confidence: 8
review_recommendation: strong
review_stars: 4
sources:
  - raw/articles/workday-ai-inference-guardrails
---

# Workday Inference Engine Built-in Guardrails - Enterprise AI Safety Infrastructure Path

Workday CTO Gabe Monroy (former Google inference infrastructure lead) makes a core argument: **LLM Guardrails should be native components of the inference engine, not bolted-on safety layers**. This perspective comes from his experience building inference infrastructure for large AI labs at Google, and from practicing in Workday's zero-tolerance "people and money" scenarios. ^[raw/articles/workday-ai-inference-guardrails.md]

## Core Argument: Guardrails Belong in the Inference Engine

Monroy's key observation:

> "Inferencing involves prefill and decode, and a whole bunch of really technical machinery in place to stream tokens out to end users, but what is nowhere in that stack today is the concept of native LLM-level enforced guardrails - guardrails that are part of the core inference."

This means current enterprise AI safety solutions (external filtering, post-processing checks, API gateway interception) are all **patch-style**, not architectural. Workday's direction is embedding safety checks into the inference flow itself. ^[raw/articles/workday-ai-inference-guardrails.md]

## Product Architecture

### Agent-Ready Tools
- MCP (Model Context Protocol) based connectors
- Enable agents to act across the Workday platform
- Agent capability boundaries defined at the tool layer, not the agent layer

### Developer Agent
- Build applications and agents on Workday using natural language
- Lowers agent development barrier, while enforcing safety constraints at the inference layer

### Agent Passport
- **Pre-production testing and verification**: Agents must pass verification before going live
- **Continuous monitoring**: Ongoing evaluation of agent behavior post-deployment
- Cisco as the first attestation partner
- Similar to "Agent safety certificate" - verified agents can access sensitive data

## Why "99% Correct" Is Not Enough

Workday's scenario specificity:
- 99% correct payroll = 1% of employees don't get paid
- HR data breach = compliance disaster (GDPR, CCPA etc.)
- Financial data errors = audit failure ^[raw/articles/workday-ai-inference-guardrails.md]

This is fundamentally different from general AI applications (chatbots, content generation) in terms of tolerance. Monroy argues that only in these zero-tolerance scenarios do inference engine built-in guardrails become necessary. ^[raw/articles/workday-ai-inference-guardrails.md]

## Differentiation from Other Approaches

| Dimension | Traditional Approach | Workday Approach |
|-----------|---------------------|------------------|
| Guardrails location | API gateway / post-processing | Inside inference engine |
| Agent verification | Runtime monitoring | Agent Passport (pre-production + continuous) |
| Data boundary | Agent accesses external API | "Bring it to our shop" (data doesn't leave domain) |
| Identity management | Service accounts | Agent as first-class identity (Okta model) |

## Technical Implications

1. **Inference infrastructure-ization**: LLM inference is transforming from "AI lab's proprietary capability" to "enterprise infrastructure's standard layer"
2. **MCP as Agent interface standard**: Workday chose MCP over custom APIs, indicating accelerating MCP adoption in enterprise agent ecosystems
3. **Agent Passport pattern**: Pre-production verification + continuous monitoring dual-phase governance may become standard for enterprise agent deployment ^[raw/articles/workday-ai-inference-guardrails.md]

-> [[raw/articles/workday-ai-inference-guardrails|original archive]] ^[raw/articles/workday-ai-inference-guardrails.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

