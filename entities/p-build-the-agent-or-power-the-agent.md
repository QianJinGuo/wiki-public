---

title: "Build the Agent or Power the Agent?"
created: 2026-06-26
updated: 2026-10-15
type: entity
tags: [article]
provenance_state: inferred
source: "[[raw/articles/p-build-the-agent-or-power-the-agent]]"
sources:
  - raw/articles/p-build-the-agent-or-power-the-agent
review_value: 8
review_confidence: 8
review_stars: 4
review_recommendation: worth-reading
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Build the Agent or Power the Agent?

> **来源**: [Build the Agent or Power the Agent?](https://tanayj.com/p/build-the-agent-or-power-the-agent)



_I’m Tanay Jaipuria, a partner at [Wing](https://www.wing.vc/) and this is a weekly newsletter about the business of the technology industry. To receive Tanay’s Newsletter in your inbox, subscribe here for free:_ ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

Hi friends,

A question I keep coming back to with founders is some version of this: should you build an agent, or should you power the agent your customer is already using? ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

For a growing number of people, the default place they do knowledge work is now a horizontal agent: Claude Code/Cowork, Codex, Copilot. That’s where the day starts and where a lot of the tasks already happen. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

So if you’re building a product, you face a fork. Either you try to become the agent for some set of users, the thing they open instead of the horizontal one. Or you accept that they live in the horizontal agent and show up there, powering it for the slice you’re good at. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

In this piece, I’ll cover:

*   What “build the agent” and “power the agent” actually mean

*   How to figure out which way you lean

*   What doing both entails realistically

[![Image 1: Build the agent versus power the agent](https://substackcdn.com/image/fetch/$s_!Vy-T!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc2a179c8-1a6b-43c1-b21f-8c5afa9b9d99_2400x1440.png)](https://substackcdn.com/image/fetch/$s_!Vy-T!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fc2a179c8-1a6b-43c1-b21f-8c5afa9b9d99_2400x1440.png) ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

Building the agent means that for some set of users, you want to be the core agent. Not as a connector inside Claude / ChatGPT, but the core interface they open instead of it. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

The appeal is obvious. You own the interface and you capture the usage and are the go to across the jobs to be done the user has. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

The catch is that you increase friction for adoption of your product and are now going heads up against ChatGPT / Claude Cowork / Codex, etc. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

It can make sense in two cases. The first is when you’re the system of record (and or action/engagement already), so your users already live in your product all day and a native agent inside it beats sending them elsewhere. The second is when the domain reasoning is the product, where the work is specialized enough that a horizontal agent can’t do it well enough. Both essentially mean that your product is verticalized enough that for some subset of users it can be the core agent they use on a daily basis. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

Harvey is one example. It’s betting that for lawyers, it can be the core agent rather than a feature inside a horizontal one. The same logic holds for other vertical players going deep on a single profession. The bet is that the work is specific enough to justify a dedicated agent. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

As general harnesses improve however, there is a risk that some competitors in these verticals take a “power the agent” approach and meet customers where they are - in another horizontal agent. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

If a general agent already does most of someone’s job, convincing them to switch to yours for a slice of it is a hard sell. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

The other path is to accept that your customer lives in a horizontal agent, and make that agent better at your domain instead of trying to replace it. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

The usual form is an MCP server but a clean CLI or a well designed API built with agents in mind also works as well. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

You are essentially building a version of your product meant to be used and consumed by an agent, most often a horizontal agent. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

So what value does your product provide in this case? ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

1.   **Data and Context**. Some products are valuable because of the data and actions they sit on, not because of any interface, and the move is to expose that data to wherever the customer already is. Some good examples include Granola which launched an MCP to allows its context to be consumed by agents. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

2.   **Capabilities and Actions**. The other value lever is to broaden or improve the set of capabilities and actions that the horizontal agent can take by giving it access to your product. One example of this is Higgsfield, an image and video generation platform, which ships an MCP server so that horizontal agents can use their reasoning and ability on orchestration and leverage Higgsfield’s products expertise in rendering and generating images and videos. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

In practice, most taking this approach will want to find ways of adding value in both dimensions - data/context and capabilities/actions. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

Its interesting to see that the incumbents are moving in this direction as well. Salesforce announced “Headless 360” in April 2026, exposing the whole platform as APIs, MCP tools, and CLI commands, with the framing that the API is the UI. HubSpot followed with a remote MCP server giving any compatible agent read and write access to CRM data. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

[![Image 2: Four questions to decide whether to build or power the agent](https://substackcdn.com/image/fetch/$s_!w_f8!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff231b681-7471-485e-9d0d-9f41791ac009_2240x1320.png)](https://substackcdn.com/image/fetch/$s_!w_f8!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff231b681-7471-485e-9d0d-9f41791ac009_2240x1320.png) ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

I think there are a few considerations for founders/builders to decide whether they should build an agent or power the agent. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

**Where does your customer already work?** Inside a tool you control or inside a horizontal agent all day? The further toward the general agent, the more you should power it, because pulling them somewhere new is going to be challenging. ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

**Do you own the whole dataset/workflow, or a slice?** If you own the full picture for a workflow or set of workflows, you can credibly be the agent for it. If you own a slice that only becomes useful once it’s combined with other data, the orchestr ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

→ [[raw/articles/p-build-the-agent-or-power-the-agent|原文存档]] ^[raw/articles/p-build-the-agent-or-power-the-agent.md]

---
## 深度分析

### The fork is a distribution decision disguised as a product decision

"Build or power" reads like an architecture question, but the article frames it as a question about where the customer's day starts. For a growing number of people that place is now a horizontal agent — Claude Code/Cowork, Codex, Copilot — and the founder's fork follows from that fact: either become the agent some users open instead of the horizontal one, or accept where they already live and show up inside it.^[raw/articles/p-build-the-agent-or-power-the-agent.md:20-24] Building means being the core interface, not a connector inside Claude or ChatGPT, which buys you the usage across all of a user's jobs-to-be-done — at the price of higher adoption friction and a head-to-head fight with the best-funded platforms in software.^[raw/articles/p-build-the-agent-or-power-the-agent.md:36-40] The asymmetry is strategic: building bets you can change user behavior, powering bets on riding behavior that already exists.

### What makes "build" defensible — and how long it stays defensible

The article names two cases where building the agent makes sense: when you are already the system of record (and/or the action/engagement layer), so a native agent inside your product beats sending users elsewhere; and when the domain reasoning itself is the product, specialized enough that a horizontal agent cannot do the work well.^[raw/articles/p-build-the-agent-or-power-the-agent.md:42] Harvey is the archetype — betting that for lawyers the work is specific enough to justify a dedicated core agent.^[raw/articles/p-build-the-agent-or-power-the-agent.md:44] But the moat has a half-life: as general harnesses improve, a competitor in the same vertical can take the power path and meet your customers inside a horizontal agent instead.^[raw/articles/p-build-the-agent-or-power-the-agent.md:46] If a general agent already does most of someone's job, convincing them to switch to yours for one slice of it is a hard sell.^[raw/articles/p-build-the-agent-or-power-the-agent.md:48] A build decision therefore needs an answer not just to "why us" but to "why still us in two years".

### Powering is building an agent-native product, not porting the old one

Powering does not mean shrinking your product into a plugin; it means building a version of it meant to be consumed by an agent — usually via an [[concepts/model-context-protocol-mcp|MCP server]], sometimes a clean CLI or an [[entities/agent-ready-api-design-patterns-2026|agent-ready API]].^[raw/articles/p-build-the-agent-or-power-the-agent.md:50-54] The article splits the value you can add into two levers: **data and context** (expose what you sit on wherever the customer already is — Granola's MCP lets agents consume its meeting context) and **capabilities and actions** (broaden what the horizontal agent can do — Higgsfield ships an MCP so agents can lean on its rendering expertise).^[raw/articles/p-build-the-agent-or-power-the-agent.md:58-62] Most players will need both levers. The strongest validation is that incumbents are moving first: Salesforce's Headless 360 exposes the whole platform as APIs, MCP tools and CLI commands under "the API is the UI", and HubSpot followed with a remote MCP server for CRM read/write^[raw/articles/p-build-the-agent-or-power-the-agent.md:64] — the interface stops being the moat exactly when powering stops being a niche play. Getting the tool surface right is its own discipline; [[entities/mcp-tool-design-tradeoffs-anthropic-2026|MCP tool design tradeoffs]] matter more than raw API coverage.

### Four questions, and the reality that most companies do both

The article's decision matrix asks: where does your customer already work, do you own the whole dataset/workflow or a slice, where does the value live (domain reasoning vs data and actions), and how much of the work stays inside one app?^[raw/articles/p-build-the-agent-or-power-the-agent.md:70-76] The predictive pattern: slices that only become useful when combined with other data get orchestrated up in the agent, so you almost have to power it; high-control, regulated, multi-step judgment leans build.^[raw/articles/p-build-the-agent-or-power-the-agent.md:72-76] The realistic end state for most companies is both, on two surfaces aimed at two audiences: an embedded agent for power users who live in your product, and a headless MCP layer for everyone on the edges who calls you from whatever agent they already run.^[raw/articles/p-build-the-agent-or-power-the-agent.md:80-90] Powering also quietly solves distribution: users who would never pay the learning cost of your UI can now call your product directly, expanding the audience to people your interface was previously inaccessible to.^[raw/articles/p-build-the-agent-or-power-the-agent.md:92] This two-surface playbook is now being written up as production doctrine elsewhere in the wiki ([[entities/vertical-domain-production-agent-playbook-alibaba-2026|vertical agent playbook]]), and the incumbent-side dynamics line up with broader [[concepts/enterprise-ai-adoption|enterprise AI adoption]] patterns.

## 实践启示

1. **Default to powering when you own a slice.** If your data only becomes useful combined with other systems, orchestration happens up in the agent — build the agent-native surface instead of fighting for the core interface.

2. **Treat the MCP/CLI/API surface as a product, not a port.** Design tools, descriptions and actions for how an agent reasons and selects; an agent-consumable interface is a distinct artifact from your GUI, and its quality is the whole product for powered users.

3. **If you do both, segment the audiences deliberately.** The embedded agent serves power users who live in your product; the headless layer serves everyone who only needs you for a slice of their job. Same backend, different surfaces, different metrics.

4. **Re-justify build-side moats on a harness-improvement clock.** Every quarter the horizontal agents get more capable, so "the domain reasoning is the product" needs fresh evidence — a competitor powering the agent can meet your customers without you.

5. **Use powering as audience expansion, not just retention.** Calling your product from a horizontal agent removes the learning cost entirely, reaching users your interface never would have — measure that as new reachable market, not cannibalization.

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

