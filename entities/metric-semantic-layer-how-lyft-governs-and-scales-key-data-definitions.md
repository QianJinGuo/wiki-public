---

title: "Metric Semantic Layer: How Lyft Governs and Scales Key Data Definitions"
description: "Strong technical depth on building a metric semantic layer with YAML/Jinja templates, governance, and change management. Unique insight into Lyft's internal solution."
created: 2026-06-22
updated: 2026-08-29
type: entity
tags: [agent, llm, data-engineering, analytics, architecture]
provenance_state: inferred
source: [[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions]]
sources:
  - raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions
review_value: 7
review_confidence: 8
review_recommendation: worth-reading
review_stars: 4
---

# Metric Semantic Layer: How Lyft Governs and Scales Key Data Definitions

Markdown Content:
[![Image 1: Iraklikhorguani](https://miro.medium.com/v2/da:true/resize:fill:64:64/0*m85T5tMk8enr2P2-)](https://medium.com/@iraklikhorguani?source=post_page---byline--56bee3643c29---------------------------------------) ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

![Image 2](https://miro.medium.com/v2/resize:fit:700/1*9r1bGT0StHEZaLhFgNhI7A.png) ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

_Written by_[_Rohit Channe_](https://www.linkedin.com/in/rohit-channe-5368b469/)_and_[_Simran Mirchandani_](https://www.linkedin.com/in/simranmirchandani/)_at Lyft._ ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

At Lyft, data isn’t just a resource — it’s woven into everything we do. Metrics drive key forecasts, steer operational decisions, and put our boldest hypotheses to the test. But as Lyft scaled, products launched and evolved, and team members came and went, we found ourselves at risk of different teams using different definitions for a given metric. What did “Metric ABC” actually mean? The answer often depended on the context and application of the team you asked. ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

The consequences were predictable. Without centralized version control or a shared standard, outdated metric definitions crept into decision-making. ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

Our solution was to build an internal **Metric Semantic Layer (MSL)**: a centralized repository that serves as a single, authoritative home for every metric’s definition — providing both a clear, plain-English description and the definitive SQL code. No more hunting across codebases or tribal knowledge — just one place to store and access a standardized, agreed-upon definition. With MSL, we have **a single source of truth** — consistent terminology and assumptions across every team, so everyone is genuinely speaking the same language. We achieve this through three key principles: ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

1.   **Simplified onboarding and change management** — update a metric definition once, and the change automatically and frictionlessly flows through every downstream application that depends on it
2.   **Intentional governance**— clarified ownership, defined scope, clear accountability for data quality, and a structure resilient enough to survive org changes, team rotations, and attrition
3.   **Transparency and accessibility** — definitions are easy for both technical and non-technical users (and downstream applications) to find and integrate into day-to-day workflows ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

Taking the above principles into account, we **implemented the Metrics Semantic Layer as a Python package**: ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

→ [[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions|原文存档]] ^[raw/articles/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

