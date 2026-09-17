---
title: "阿里云 Agent Native Cloud — Infra-Platform-Desktop Three-Layer Architecture"
created: 2026-07-22
updated: 2026-09-17
type: entity
tags: [Alibaba-Cloud, agent-native, agent-platform, agent-infrastructure, enterprise-agents, AIOps]
sources: [raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力]
confidence: 0.6
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 阿里云 Agent Native Cloud

阿里云 Agent Native Cloud is [[alibaba-agentic-cloud|Alibaba Cloud]]'s three-layer architecture for making agents a native enterprise capability. Presented by Zhou Qi (简志), head of Alibaba Cloud's cloud-native application platform, at the World AI Conference, it defines how enterprises can evolve from building individual agent demos to operating agents as **controllable, reusable, collaborative, and evolving organizational assets**. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md]

## Three-Layer Architecture: Infra - Platform - Desktop

### Agent Infra
The runtime foundation providing secure, elastic, low-cost execution environments. Core component is **Agent Sandbox**, with four capabilities: ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md]
- **Ready-to-use templates** — Code Interpreter, Browser, AIO, OSWorld from open-source community images
- **Strong isolation** — MicroVM/VM-level compute isolation + network, storage, and session isolation
- **Low-cost long sessions** — Deep sleep, light sleep, and on-demand wake (scale to zero)
- **High-concurrency elasticity** — Sub-second startup for large-scale scheduling
- Covers both Function Compute (FC) and Container Compute (ACS) scenarios

### Agent Platform
A unified enterprise-grade Agent PaaS control plane with 7 modules: ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md]
1. **Identity** — Unique AgentID with authentication and permission control
2. **Gateway** — Credential management, zero-touch secrets, security guardrails
3. **Policy** — Business rule validation and intervention on dangerous operations
4. **Asset Registry** — Unified management of Agents, MCPs, Tools, and Skills
5. **Observability** — Call chain, tool chain, decision chain traces
6. **Evaluation & Optimization** — Drive continuous evolution
7. **Version Management** — Canary release, rollback, change tracking

### Agent Desktop
Powered by [[阿里云发布-agentteams-与-agentloop破解企业智能体规模化落地两大难题|Wuying Agentic Computer]], providing a 7×24 complete desktop environment covering 80% of enterprise white-collar work scenarios. Supports MCP, CLI, SDK, and OpenAPI for flexible integration. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md]

## Key Platform Products

- **[[aliyun-agentrun|AgentRun]]** — High-code Agentic AI infrastructure platform managing runtime, sandbox, models, memory, knowledge bases, credentials, gateway, observability, and evaluation
- **[[alibaba-cloud-agentteams-enterprise-multi-agent|AgentTeams]]** — Multi-agent governance and collaboration supporting Human-to-Agent and Agent-to-Agent workflows, with Leader/Worker agent organization
- **[[aliyun-agentloop-enterprise-agent-self-evolution-flywheel|AgentLoop]]** — Full-stack observation, audit, evaluation, experimentation, and continuous optimization

## The Five Dimensions of Agent Native

Being "Agent Native" is more than connecting an agent — it's a new production relationship across five dimensions: ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md]
- **Business native** — Agents enter critical production workflows and deliver measurable results
- **Organization native** — Clear human-agent collaboration mechanisms
- **Engineering native** — Full lifecycle management of agent construction, release, and reuse
- **Operations native** — Agent observation, evaluation, and continuous optimization
- **Infrastructure native** — Runtime, data, identity, and reliability guarantees

## 深度分析

### Agent ≠ Agent Native：成熟度的单位不是 Agent 数量

A demo proves **model** capability; production proves the enterprise's **platform** capability to convert model capability into deterministic business results. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:32] Connecting an agent is therefore not the same as being agent native — a difference of kind, not degree. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:98] If agent count were the unit of maturity, every team running a few coding agents would already be native and the metric would rise as a pure function of adoption. The article's unit is instead 可控、可复用、可协作、会进化 applied to agents as organizational assets; the five nativities operationalize it by testing whether agents entered the enterprise's production relations and not whether more of them exist. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:34]

### 为什么单一产品解不了这个问题

The cross-role divergence is partly adversarial: developers want faster build and deploy, security wants permission management and process audit, the business wants quantifiable delivered value, the platform team wants stability, performance and cost at scale. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:42] Velocity argues for broad permissions while audit argues for narrow ones; shipping a scenario argues for speed while cost governance argues for retiring it. Overlaying that, the optimization target shifts with lifecycle stage — build-and-ship trades on efficiency, run-time on effect, maturity on cost and ROI. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:44-46] One artifact, many owners, changing objectives: the Infra–Platform–Desktop split is not taxonomy but the place where those conflicts are reconciled in code, which is why no single framework absorbs them.

### Infra 与 Desktop：把 Agent 放进真实执行环境

Agent Infra rests on Sandbox, Database, FS and Network, with [[concepts/agent-sandbox|Agent 沙箱]] as the core across four layers: out-of-the-box templates (community images plus built-in Code Interpreter, Browser, AIO, OSWorld), MicroVM/VM compute isolation with network, storage and session isolation, deep/light sleep with on-demand wake so instances scale to zero, and extreme elastic startup for high concurrency. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:62-67] Hibernation is the least glamorous and most consequential layer: the dominant cost of a real agent workload is rarely compute but wall-clock idleness — an agent blocked on human approval or a multi-day process holds resources while doing nothing — and scale-to-zero turns that waiting from a metered cost into a scheduling problem, which is the precondition for long-horizon agents being economically defensible at all. Session isolation is layered for the same reason: an agent's context, filesystem and credentials form one unit, and leaking it across tenants is a different failure mode from a noisy-neighbour CPU problem. FC-and-ACS coverage with an FC-compatible SDK, K8s protocol support and E2B compatibility reads as a bet on protocol-level adoption over proprietary lock-in. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:69]

The desktop layer's argument against an in-browser agent is surface area, not performance: a real Windows/Linux desktop runs existing enterprise software unmodified, whereas a browser-shaped agent must re-implement every interaction surface of every line-of-business application — and legacy software is precisely what cannot be re-platformed. Durability completes it, since laptop sleep and network loss interrupt tasks and destroy the value of long-horizon autonomy; a datacenter SLA takes the client machine off the critical path, six identity sources and a seven-layer control loop over seven risk classes supply a compliance evidence chain, and integration runs through MCP, CLI, SDK and OpenAPI. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:85-90]

### Platform：Agent 身份与零明文凭据托管才是安全模型的切换点

Of the seven control-plane modules, two change the security model rather than adding a feature. Identity issues a unique AgentID so every operation is attributable to a specific person or agent; Gateway holds credentials and enforces guardrails so the agent never touches plaintext secrets at any point. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:75-76] Compare the common anti-pattern of injecting an API key into the agent's environment: a prompt injection's blast radius becomes the key's entire scope, revocation requires redeploying the agent, and the audit trail records "the agent did it" rather than which human caused it. Identity plus credential custody inverts the direction of trust — the agent presents an identity, the gateway exchanges it for a scoped short-lived credential, and the call chain becomes attributable end-to-end. Policy, the asset registry, observability across call/tool/decision chains, evaluation, and canary release with rollback only become meaningful on top of that: without attribution and scope-limited secrets there is nothing coherent to register, observe, evaluate or roll back. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:77-81]

### 复利：企业拥有的是产能装置，而不是一批 Agent

The article names Platform's most important value as 复利 — compounding. A Skill, Tool, policy or evaluation set sunk by one team is reusable by the next scenario, a risk found in one scenario becomes a global rule, one optimization propagates across the whole agent population, and the end state is not a set of agents but a workflow that keeps producing them. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:118] This is the one genuinely competitive claim, and it is deliberately not model-side: models get stronger and easier to obtain, while an enterprise's own high-quality task trajectories, evaluation standards, failure cases and correction mechanisms cannot easily be copied. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:126] The surfaces are [[entities/aliyun-agentrun|AgentRun]] as standardized high-code infrastructure, [[entities/alibaba-cloud-agentteams-enterprise-multi-agent|AgentTeams]] as governance across Human-to-Agent, Agent-to-Agent and multi-person/multi-agent collaboration with Leader/Worker topology, and [[entities/aliyun-agentloop-enterprise-agent-self-evolution-flywheel|AgentLoop]] as the trajectory → evaluation → next-round-optimization loop. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:122-128] Leader/Worker matters because that is how an agent organization mirrors a human one: a single desktop agent improves one person's throughput, whereas a supervised agent team changes who performs the work in a process. [[entities/starops-rum-intelligent-inspection|STAROps]] is the production showcase — an AIOps agent in assistant, long-task and digital-employee forms that converts expert operational experience into reusable Skills. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:130]

### 证据的边界：可迁移的模式与厂商叙事

This remains a vendor's platform story. The dogfooding numbers — 15 agents at 7×24, 85% of Q&A volume, 90% less operational support time, one-day release cycles — are self-reported, with no independent benchmark, no baseline and no denominator, on a sample of one enterprise sold to by the same vendor. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md:146] What survives the discount is architecture-level: agent identity plus credential custody as a governance primitive rather than an add-on, an asset registry that makes Skills and Tools versionable (see [[concepts/model-context-protocol-mcp|MCP]] for the tool-plane analogue), a trajectory-driven evaluation loop as the mechanism by which an agent improves after launch, hibernation and scale-to-zero as the economic enabler for long-horizon agents, and a real-desktop surface for software that cannot be re-platformed. What does not transfer is the product-specific layer — the exact seven-module decomposition, the 80% coverage figure, the one-day onboarding claim. The framing itself is the durable contribution: the scarce good is not model access but the ability to convert model capability into deterministic, attributable, continuously improving business results.

## 实践启示

1. **Score maturity with the five nativities, never with agent count.** Require an artifact behind each dimension — a measurable result, a named human-agent responsibility split, a lifecycle process, an evaluation loop, a reliability guarantee. A dimension backed only by a demo means adoption stage, not native stage.

2. **Build identity and credential custody before choosing a framework.** Issue a distinct identity per agent and route every credential through a custody layer so no agent holds a plaintext key. Cheapest early, most expensive to retrofit: without attribution and scoped short-lived secrets, observability, evaluation and rollback operate on an unattributable subject.

3. **Treat the sandbox as first-class infrastructure, and budget for idleness rather than concurrency.** Verify templates for the environments you need, compute plus session isolation, hibernation with wake-on-demand so approval- or pipeline-blocked sessions scale to zero, and startup fast enough for your burst pattern. Measure cost per completed long-horizon task, not per vCPU-hour.

4. **Stand up an asset registry and a trajectory-driven eval loop as the compounding mechanism.** Version Agents, MCPs, Tools and Skills so the next scenario inherits the last one's work, and turn production failure cases into evaluation sets and global rules. The eval set, failure catalogue and correction mechanism are what fail to commoditize when models do.

5. **Reach for a real desktop only when the software cannot be re-platformed.** For thick-client or legacy line-of-business dependencies, or multi-day continuity a laptop cannot hold, a datacenter-hosted desktop with an enterprise identity source and an audit evidence chain beats re-implementing each application surface inside a browser agent. Pair it with per-team token budget visibility.

6. **Separate transferable architecture from vendor claims before committing.** Validate the structural patterns against your own pilot and baseline, and treat self-reported multipliers as unverified until reproduced on your workloads — define the denominator before accepting anyone's percentage.

## Dogfooding Results

Alibaba Cloud runs 15 agents 7×24 handling development, customer support, and operations — processing 85% of Q&A, reducing operational support time by 90%, and compressing release cycles to 1 day. ^[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力.md]

→ [[raw/articles/阿里云-agent-native-cloud让智能体成为企业原生的能力|原文存档]]
