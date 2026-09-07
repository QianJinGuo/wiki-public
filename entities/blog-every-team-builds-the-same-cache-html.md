---

title: "Every Team is Building the Same Cache — TierFS"
created: 2026-06-26
updated: 2026-09-07
type: entity
tags: [article]
source: "[[raw/articles/blog-every-team-builds-the-same-cache-html]]"
sources:
  - raw/articles/blog-every-team-builds-the-same-cache-html
review_value: 7
review_confidence: 8
review_stars: 4
review_recommendation: worth-reading
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Every Team is Building the Same Cache — TierFS

> **来源**: [Every Team is Building the Same Cache — TierFS](https://tierfs.com/blog/every-team-builds-the-same-cache.html)



Why infrastructure teams keep solving the same storage problem from scratch, and what would happen if they didn't have to. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

If you work at a company that runs serious compute workloads in the cloud, there's a good chance one of your engineers has spent the last quarter building a cache. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

Not on purpose, exactly. Nobody set out to build a cache. The project was supposed to be about something else — getting an inference cluster online, scaling training across more GPUs, making the agent platform feel snappier. But somewhere in the middle of it, somebody noticed that pulling 70 GB of model weights from S3 every time a replica came up was taking ten minutes, and that wasn't going to work, and so they wrote a thing. A script that pre-warms a local NVMe drive. A sidecar container that pulls from S3 once and shares to peers. A Redis-backed lookup that hands out cached file paths. The thing has different names at different companies. It's always there. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

We've talked to engineers at several different neocloud and AI infrastructure teams in the last few months. Every single one has built some version of this. None of them are particularly proud of it. It's not the work they wanted to be doing; it's the work that turned out to be in the way of the work they wanted to be doing. And when you ask them about it, they all say roughly the same thing: _"yeah, we should probably open-source it, but it's pretty specific to our setup."_ ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

That's the part that keeps catching our attention. Many teams, many versions of the same thing, none of them confident their version is good enough to share. Meanwhile every new team starting up does the same thing again, because they have to. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

## The shape of the problem

The actual problem is simple enough to state in a sentence: compute is fast, networks are slow, and the same data gets read many times. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

This is, of course, the reason caches exist. It's also a problem that's getting worse rather than better as workloads change. The compute side keeps getting more ephemeral — sandboxes that spin up for a task and disappear, inference replicas that come and go with traffic, training jobs that claim GPUs for an afternoon. The data side keeps getting larger — model weights that used to be 1 GB are now 70 GB or 700 GB, datasets that used to fit on a laptop now span buckets. And the same data gets touched, over and over, by different instances of compute that don't share state. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

If you run a single big stable cluster, you can paper over this. Pre-stage the data, keep it warm, never let the working set go cold. If you run an ephemeral fleet — which everyone is moving toward, because it's cheaper and more flexible — you can't. Every new compute instance starts cold, and the cost of starting cold dominates the cost of doing the actual work. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

So you build a cache. Or, more often, you build several caches, in succession, because the first version handles the easy case and then breaks when the workload gets weirder. The pattern goes something like: ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

First, somebody adds a local NVMe cache on each compute node. Works great. Hit rates are high, latency drops, everyone's happy. Then the fleet grows, and you notice that every new node is pulling the same data from S3, and your S3 bill is going up, and you're hitting rate limits. So somebody builds a peer-to-peer layer — nodes ask each other for cached data before going to S3. Works great, for a while. Then the fleet becomes truly ephemeral — sandboxes spin up and disappear — and the local caches never get warm enough to matter, and the peer fabric is too thin because half the peers don't exist by the time you check. So somebody builds a dedicated cache cluster, separate from the compute, that stays warm and gets hit by everybody. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

This is the arc. We've seen it told to us by different people who didn't realize they were describing the same arc. The endpoint is roughly: local cache, peer cache, dedicated cache tier, falling back to the actual data store as a last resort. Three layers. Same shape every time. Different code every time. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

## Why does it keep happening

You'd think, given that everyone ends up at the same place, somebody would have built the thing and shared it by now. And to be fair, some pieces exist. The cloud providers ship their own managed-cache products in front of object storage, but each one is tied to its provider. Several open-source projects have built filesystem-shaped abstractions over object stores, each with its own trade-offs in data format, deployment model, or feature gating. There are options. None of them are quite the thing the agentic stack needs. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

So everyone keeps building. And the builds are private, because they're _"specific to our setup,"_ even though when you look at them they're 80% the same code with different names. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

There's an interesting question buried in here, which is: why are we still doing this in 2026? The obvious answer is that the infrastructure layer underneath compute hasn't kept up with the compute layer itself. Kubernetes solved how to schedule containers. Terraform solved how to provision infrastructure. The agent frameworks solved how to orchestrate language models. The piece in the middle — _how does the right data get to the right compute fast enough to matter_ — is still being solved one company at a time, badly. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

The deeper answer is that this kind of infrastructure has historically been hard to do well in the open. Storage systems are operationally tricky. Caching is full of subtle correctness bugs. Distributed protocols have a long tail of failure modes. The companies that have the expertise to build it well also have a business reason to keep it private. The companies that would benefit from a shared solution don't have the expertise to build one. There's a gap. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

We think the gap can close. That's mostly what we want to talk about. ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

## What we're working on

We're building TierFS, an open-source filesystem 

→ [[raw/articles/blog-every-team-builds-the-same-cache-html|原文存档]] ^[raw/articles/blog-every-team-builds-the-same-cache-html.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

