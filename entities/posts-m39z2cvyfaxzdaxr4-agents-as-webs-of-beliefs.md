---

title: "Agents as Webs of Beliefs"
created: 2026-06-29
updated: 2026-09-05
type: entity
tags: [agent, mlops, research]
provenance_state: inferred
source: "[[raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs]]"
sources:
  - raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs
review_value: 7
review_confidence: 6
review_stars: 4
review_recommendation: worth-reading
confidence: 0.6
---

# Agents as Webs of Beliefs

> **Source**: [www.lesswrong.com](https://www.lesswrong.com/posts/M39Z2CvyfaxZdaxR4/agents-as-webs-of-beliefs)

Synthesizes ideas from active inference, agent foundations, and ML into a 'belief webs' framework. Original synthesis but lacks empirical data or implementation details. Good conceptual depth for agent foundations, but confidence is moderate due to lack of verifiable benchmarks. ^[raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs.md]

## Content Summary



In this post I’ll sketch out an informal model of intelligent agents as webs of beliefs (or belief webs for short). The belief webs framework pulls together ideas from active inference, agent foundations and machine learning. In doing so it aims to unify beliefs, goals and actions as three facets of a single phenomenon. Few of these ideas are original to me, but I haven't seen anyone tie them together in a single place before. I've flagged the frameworks I'm drawing from throughout the post. ^[raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs.md]

### Beliefs are held together by local consistency constraints

The core premise of belief webs is that an agent’s beliefs are typically _locally_ consistent with nearby beliefs but not necessarily _globally_ consistent with all its other beliefs (except, perhaps, in the limit of ideal rationality). This poses a problem for frameworks which describe agents in terms of a single probability distribution (as causal graphs, Solomonoff induction, and active inference do). ^[raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs.md]

Two frameworks which are capable of handling global inconsistency are [Richardson’s probabilistic dependency graphs](https://arxiv.org/abs/2202.11862) (PDGs) and [Garrabrant induction](https://www.lesswrong.com/posts/y5GftLezdozEHdXkL/an-intuitive-guide-to-garrabrant-induction). (They focus on empirical inconsistency and logical inconsistency respectively, but I’ll abstract away from that difference for now.) We can roughly analogize the nodes in PDGs to the propositions in Garrabrant inductors; I’ll call them “base-level beliefs”. The central type of base-level belief I think about is beliefs about sensory inputs.[1](http://www.lesswrong.com/posts/M39Z2CvyfaxZdaxR4/agents-as-webs-of-beliefs#fn38cdnvw1q1g) ^[raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs.md]

There’s then a second layer of structure in both PDGs (namely hyperedges) and Garrabrant induction (namely traders) which imposes local constraints on base-level beliefs. I think of hyperedges/traders as steps towards formalizing the concept of “concepts”.[2](http://www.lesswrong.com/posts/M39Z2CvyfaxZdaxR4/agents-as-webs-of-beliefs#fnl4lxbcv75t) For example, if you see the front half of a cat starting to emerge from around the corner, a “cat” hyperedge/trader might make predictions about what you’ll see next, which shape your base-level beliefs. ^[raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs.md]

However, having exactly two layers of structure seems rather artificial. In active inference/predictive processing, minds are viewed as hierarchical generative models, with each layer of the hierarchy forming new concepts with reference to lower-level concepts.[3](http://www.lesswrong.com/posts/M39Z2CvyfaxZdaxR4/agents-as-webs-of-beliefs#fnq6v6sftxiih) The success of deep learning suggests that there’s something fundamentally important about this kind of hierarchical concept formation. Whereas you can’t have hyperedges connecting other hyperedges, or traders trading on other traders. ^[raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs.md]

So you can think of the term “belief webs” as a (still vague) pointer towards a framework which is ^[raw/articles/posts-m39z2cvyfaxzdaxr4-agents-as-webs-of-beliefs.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

