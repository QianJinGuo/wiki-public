---

title: "Enrich your datasets with business context"
created: 2026-07-10
updated: 2026-09-07
type: entity
tags: [reasoning, aws]
sources: [raw/articles/enrich-your-datasets-with-business-context-migrating-from-le]
review_value: 7
review_confidence: 9
review_recommendation: worth-reading
review_stars: 3
confidence: medium
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Enrich your datasets with business context: Migrating from legacy Topics to semantic datasets in Amazon Quick

→ [[raw/articles/enrich-your-datasets-with-business-context-migrating-from-le|原文存档]] ^[raw/articles/enrich-your-datasets-with-business-context-migrating-from-le.md]

# Enrich your datasets with business context: Migrating from legacy Topics to semantic datasets in Amazon Quick

If you’ve been managing [Amazon Quick](<https://aws.amazon.com/quick/>) legacy Topics alongside your datasets, you know the challenge: two assets that must stay perfectly synchronized, each with its own permissions, lineage, and versioning. Column synonyms drift. Calculated fields diverge. A rename in the dataset breaks the Legacy Topic silently. You can now use Amazon Quick to embed that business context directly into the dataset itself through **Dataset Enrichment** in the new data prep experience. Column descriptions, synonyms, calculated fields, custom instructions, and business rules all live alongside the data. Dataset Enrichment bakes business context directly into the dataset. Everything (permissions, semantics, AI context) travels with the data and is automatically inherited by anything built on top of it. One asset, one source of truth, one place to govern. ^[raw/articles/enrich-your-datasets-with-business-context-migrating-from-le.md]

In this post, we walk through what Dataset Enrichment is, how it differs from legacy Topics, and provide three migration scenarios with step-by-step guidance so you can move your business context into the dataset layer with confidence. ^[raw/articles/enrich-your-datasets-with-business-context-migrating-from-le.md]

**Topic** is now the [multi-dataset](<https://aws.amazon.com/blogs/machine-learning/build-a-unified-semantic-layer-across-datasets-with-multi-dataset-topics-in-amazon-quick/>) semantic and reasoning layer, the construct where multiple datasets are composed, relationships are defined, business metrics are authored, and business terminology is mapped. Rather than introducing a net-new construct, we are re-purposing Topic to fulfill this role more completely. Moving dataset-intrinsic semantics down to where they belong, and elevating Topic to own the cross-dataset relationships, metrics, and business terminology that it was always meant to carry. This isn’t a cosmetic change. It establishes a clean, forward-looking architecture that supports both deterministic BI workflows and flexible AI-driven analytics from a shared semantic foundation. It also sets up the framework for catalog integration. ^[raw/articles/enrich-your-datasets-with-business-context-migrating-from-le.md]

### What is Topics (legacy)

Legacy Topics provided the initial approach to adding business context to datasets in Amazon Quick Sight. It stored column synonyms, calculated fields, named entities, filters, and custom instructions in a separate object that sat on top of the dataset, linked but independently managed. Going forward, we classify existing Topics as legacy. The new version of Topics is being elevated to a _multi-dataset semantic layer_**.** A single-entry point for cross-dataset Q&A that lets business users and AI workflows query across multiple enriched datasets in one conversation. Dataset Enrichment is the foundation that makes this possible: each dataset must carry its own semantic context before Topics can unify them at a higher level. ^[raw/articles/enrich-your-datasets-with-business-context-migrating-from-le.md]

### Key differences: Topics (legacy) vs. Dataset Enrichment (new data prep)

 

| **Legacy Topics** |

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

