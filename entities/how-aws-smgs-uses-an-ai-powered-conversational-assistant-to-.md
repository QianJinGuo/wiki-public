---

created: 2026-06-10
updated: 2026-10-03
title: "Business intelligence at scale: Key obstacles"
type: entity
tags: [rss, article, agent, ai, llm, bedrock, aws, observability]
source: [[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-]]
review_value: 7
review_confidence: 8
review_stars: 4
sources:
  - raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Business intelligence at scale: Key obstacles


## How AWS SMGS uses an AI-powered conversational assistant to transform business management with Amazon Bedrock AgentCore

 

AWS leaders manage complex data across multiple hierarchies while making time-sensitive decisions that impact global operations. Traditional business intelligence relies on static dashboards and manual reports, which creates delays and limits organizational agility. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

NarrateAI, our intelligent conversational solution, addresses this through conversational agentic AI powered by our data lake and [**Amazon Bedrock AgentCore**](https://aws.amazon.com/bedrock/agentcore/). Accessible through the **[Amazon Quick](https://aws.amazon.com/quick/)** conversational interface, NarrateAI delivers on-demand, context-rich business intelligence to leaders across AWS, from the Chief Executive Officer (CEO) to the field. By answering natural language questions about business performance, NarrateAI provides immediate, accurate, and actionable insights that remove barriers between leaders and their data. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

In this post, we share how we built NarrateAI using **Amazon Bedrock AgentCore** to deliver business intelligence at scale for the AWS SMGS (Sales, Marketing and Global Services) organization. You will learn about: ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

*   The two-layer architecture that separates batch processing from real-time interaction.
*   The specialized AI agents that power intelligent routing and validation.
*   Key engineering patterns for production deployment.
*   How to build similar solutions with AWS services.

## Business intelligence at scale: Key obstacles

AWS faced challenges that limited the effectiveness of traditional business intelligence approaches: ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

**Time-intensive preparation**: AWS leaders traditionally lost hours gathering data manually before business reviews. The preparation process involved navigating multiple dashboards, reconciling data across disparate sources, and manually synthesizing insights, leaving little time for strategic reasoning and decision-making. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

**Data fragmentation**: Business insights were scattered across multiple systems and dashboards, requiring leaders to piece together a coherent narrative from fragmented data sources. This fragmentation created inconsistencies in metrics and made it difficult to maintain a unified view of business performance across hierarchies and datasets. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

**Limited accessibility**: Complex dashboards required specialized knowledge to navigate effectively, creating dependencies on intermediary reporting teams. Leaders could not access insights on-demand and instead had to wait for curated reports, which delayed critical business decisions and limited organizational agility. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

## Solution overview

NarrateAI addresses the challenge of making complex business data conversational through a two-layer architecture: batch narrative generation and real-time interaction. This separation supports comprehensive data processing upfront while delivering instant, contextually accurate responses through natural conversation. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

Amazon Bedrock AgentCore removed the need to build custom orchestration infrastructure, providing serverless architecture, built-in authentication, memory management, and integration with foundation models. This accelerated our deployment from months to weeks while maintaining production-quality observability and security through native [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) integration and automated session management. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

### Automated narrative generation layer (batch processing)

NarrateAI batch-generates comprehensive persona-based narratives for each user through a three-stage pipeline: ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

1.  **Data extraction** — Configuration-driven Structured Query Language (SQL) templates (parameterized queries that adapt to each user’s role and permissions) extract structured data from [Amazon Redshift](https://aws.amazon.com/redshift/). These templates support multi-level breakdowns and time series analysis while enforcing user-specific access controls.
2.  **Data transformation** — [AWS Lambda](https://aws.amazon.com/lambda/) transforms the extracted data into structured JavaScript Object Notation (JSON) using section-type logic (objects, arrays, breakdowns, and containers) with field mappings and hierarchical organization.
3.  **Narrative rendering** — Jinja templates (a widely used Python templating engine) render human-readable narratives from the structured data. A hierarchical, business domain-aware chunking strategy handles large datasets efficiently. The system stores each user’s narrative as a text file in [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/), supporting row-level security through full data isolation.

### Conversational AI

## 深度分析

### The two-layer split is a latency/freshness tradeoff

NarrateAI's batch/real-time separation is pre-computation applied to BI: the expensive work (Redshift SQL extraction, Lambda transformation, Jinja rendering) runs offline in a three-stage pipeline, so the interactive layer only retrieves from pre-generated persona narrative files in S3. This mirrors the pre-generated knowledge pattern in [[concepts/retrieval-augmented-generation-rag|RAG]] — retrieval works over curated, structured knowledge, which is why a table-of-contents (TOC) extractor pulls only relevant narrative sections without scanning whole files. The cost is freshness: answers are only as current as the Data Refresh Scheduler's cadence. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:34-50] ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:93-100]

### Row-level security enforced at generation time, not query time

Permissions are applied during transformation, and each user's narrative file in S3 is fully isolated — access control is baked into data processing rather than guarded at query time. This "enforce access at the source" philosophy eliminates a class of RAG failure modes (retrieval-layer leaks, prompt-injection exfiltration, filter-bypass bugs) because unauthorized data never enters the artifact. The trade-off is refresh fan-out: 4,000+ users means one isolated narrative per user, regenerated across the permission matrix on each refresh. Context-awareness falls out of the same mechanism — the Persona Knowledge Identifier maps who is asking to which file. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:60-66] ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:130-132]

### Hallucination defense is layered and deterministic-first

Because output drives executive decisions, model output is treated as untrusted: LLM involvement in numeric calculation is deliberately limited — numbers come from deterministic SQL and templates — and every response passes an Online Evaluator that cross-references figures against source data before delivery. Bedrock Guardrails add content filtering, PII redaction, and tone controls. The stated lesson: the LLM handles language and synthesis; computation and validation stay deterministic. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:110-114] ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:128-130] ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:87]

### Managed agent infrastructure compressed the build from months to weeks

Amazon Bedrock AgentCore replaced custom orchestration with serverless runtime, built-in auth, and native memory — the team migrated conversation history off a hand-rolled DynamoDB session store, deleting custom session code. OpenTelemetry-based observability cut troubleshooting from hours to minutes, and model flexibility (upgrading Claude versions without architectural change) pays dividends past initial deployment. What was NOT outsourced: domain logic — institutional knowledge encoded with domain experts through standardized templates. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:116-124] ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:136]

## 实践启示

1. **Precompute what is asked often.** If most queries hit a known, role-shaped slice of data, batch-generating narrative artifacts offline beats answering live against the warehouse — latency collapses and answers become consistent. Manage the freshness lag with scheduled refresh. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:34-50]
2. **Enforce permissions during data preparation, not at query time.** Baking row-level access into per-user artifacts is structurally safer than filtering retrieved results, and persona-based personalization falls out of the same mechanism. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:60-66]
3. **Keep the LLM away from arithmetic.** Deterministic pipelines compute the numbers; the model only synthesizes language — then every response is validated before it reaches a decision-maker. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:128-130]
4. **Route by complexity.** Simple questions take a fast path; only multi-part questions get decomposed into parallel sub-tasks. This routing plus pre-analyzing document structure at ingestion fixed early latency problems and low adoption. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:82] ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:134]
5. **Buy orchestration, build domain logic.** Managed agent infrastructure moved this project from months to weeks, but encoding institutional knowledge with domain experts remains the differentiating, manual work. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:136]
6. **Watch the pre-computation tax.** Per-user artifact generation scales storage and refresh cost linearly with user count; confirm your permission matrix and refresh cadence don't make regeneration the new bottleneck. The roadmap points the other way — event-driven, proactive delivery triggered by data changes. ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md:150-152]

## 相关实体
- [[entities/滴滴国际化客服质检智能化之路基于-amazon-bedrock-的多语种多业务线质检实践]]
- [[entities/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe]]
- [[entities/automate-aml-alert-triage-with-amazon-quick-and-snowflake-co]]
- [[entities/对抗-agent-遗忘kollab-基于amazon-bedrock-agentcore-的团队ai工作空间实践]]
- [[entities/process-financial-documents-using-amazon-bedrock-data-automa]]

→ [[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-|原文存档]] ^[raw/articles/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-.md]

