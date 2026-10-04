---

created: 2026-06-10
updated: 2026-10-03
title: "Workflow architecture"
type: entity
tags: [rss, article, ai, llm, bedrock, sagemaker, aws, observability]
source: [[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe]]
review_value: 7
review_confidence: 8
review_stars: 4
sources:
  - raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Workflow architecture


## Comprehensive observability for Amazon SageMaker AI LLM inference: From GPU utilization to LLM quality

Deploying large language models (LLMs) at scale on [Amazon SageMaker AI Inference](<https://aws.amazon.com/sagemaker/ai/deploy/>) makes observability a critical pillar of any production machine learning (ML) strategy. Unlike conventional software that returns deterministic outputs, LLMs generate variable, free-form responses that are difficult to validate with standard metrics. LLM output quality can change over time as input distributions shift, and quality monitoring helps detect these changes early. For generative AI workloads, observability also includes the model serving infrastructure, where unpredictable token consumption, GPU memory pressure, and latency spikes make capacity planning and cost control a moving target. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

A comprehensive observability approach for LLM inference must address two distinct but complementary dimensions: model serving infrastructure (quantity) and LLM quality. Quantity monitoring focuses on the operational health of inference infrastructure, tracking request throughput and resource utilization. These metrics help detect bottlenecks, right-size compute resources, and control costs. Quality monitoring focuses on the performance of the LLMs themselves, evaluating response accuracy, compliance, and consistency over time. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

Most teams build LLM observability in stages. The first stage establishes visibility into core operational metrics such as latency, errors, and resource utilization. These signals confirm the reliability of inference endpoints. The next stage adds LLM quality through sampling and evaluation, which surface issues such as model drift, degradation, or unexpected behavior in generated responses. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

With both dimensions in place, you can introduce thresholds and automated alerts that combine infrastructure and quality signals. Over time, the practice extends to comparative analysis across models and configurations so you can continuously tune cost, performance, and output quality. Quantity and quality metrics are interdependent: an endpoint can appear operationally healthy while producing poor or unsafe responses, or it can deliver high-quality outputs while running inefficiently on over-provisioned infrastructure. Production-grade LLM observability emerges when both dimensions are monitored, correlated, and optimized together. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

This post demonstrates a comprehensive observability solution using [Amazon Managed Grafana](<https://docs.aws.amazon.com/grafana/latest/userguide/what-is-Amazon-Managed-Service-Grafana.html>) dashboards that provides a holistic view of both quality and quantity for LLMs served on Amazon SageMaker AI endpoints with inference components. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

## Workflow architecture

For full visibility into LLMs across the two monitoring dimensions of quantity and quality, we built a solution using three core AWS services, each chosen for a specific role in LLM observability. The following high-level data flow diagram shows the three core components: Amazon SageMaker AI endpoints with inference components, Amazon CloudWatch, and Amazon Managed Grafana. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

[Amazon SageMaker AI Inference Components](<https://aws.amazon.com/sagemaker/ai/deploy/>) serve as the model hosting layer. A single SageMaker AI endpoint can host multiple inference components, each running a different LLM (for example, `gpt-oss-20b` and `Qwen2.5-7B-Instruct` as shown in the preceding architecture). Inference components let you deploy, scale, and manage multiple models on shared infrastructure while keeping per-model isolation for traffic routing, scaling policies, and metric attribution. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

[Amazon CloudWatch](<https://aws.amazon.com/cloudwatch/>) serves as the centralized metrics store. It receives two distinct streams of data from each inference component: enhanced metrics and custom quality metrics. Enhanced metrics are published automatically by SageMaker AI when you enable them on the endpoint configuration. The metrics include instance-level, container-level, and per-GPU dimensions, giving you granular visibility into invocation counts, latency, error rates, and GPU/CPU utilization per model. Enhanced metrics are logged to the `/aws/sagemaker/InferenceComponents/<model-name>` namespace (for example, `/aws/sagemaker/InferenceComponents/gpt-oss-20b`). For details, see the [Amazon SageMaker AI enhanced metrics documentation](<https://docs.aws.amazon.com/sagemaker/latest/dg/monitoring-cloudwatch-enhanced-metrics.html>) and the [enhanced metrics deep-dive blog post](<https://aws.amazon.com/blogs/machine-learning/enhanced-metrics-for-amazon-sagemaker-ai-endpoints-deeper-visibility-for-better-performance/>). ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

Custom quality metrics c^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]


## 深度分析

### Quantity and quality are not two dashboards — they are one diagnostic loop

The most interesting design decision in this architecture is not the tooling but the separation of metric namespaces: enhanced operational metrics flow to `/aws/sagemaker/InferenceComponents/<model-name>` while custom quality scores land in `/aws/sagemaker/inference-quality/<model-name>`. The split looks like hygiene, but it enables a diagnostic workflow neither signal supports alone: a latency spike without quality change points at infrastructure saturation (GPU memory pressure, noisy neighbors on a shared endpoint), while a quality drop with flat latency points at input drift or a bad model update. Correlating the two namespaces per inference component turns ambiguity ("something is wrong") into attribution ("what kind of wrong"). The article's framing that quantity and quality are "interdependent" is the practical core: an endpoint can be green on errors and p99 while producing unsafe completions, or serve excellent answers on hardware you are paying for and not using. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:17-19] ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:27-34]

### Multi-model endpoints change what observability must attribute

SageMaker inference components let one endpoint host several LLMs (the article's example runs `gpt-oss-20b` and `Qwen2.5-7B-Instruct` side by side) with per-model traffic routing, scaling policies, and metric attribution. This quietly raises the observability bar: aggregate endpoint metrics become nearly useless because they blend models with different token profiles, GPU appetites, and quality baselines. The dashboards in the article are consistently dimensioned per model — GPU compute % per model, cost/hour per model, quality scores compared across models — because the actionable questions are per-model: is one model starving the other on shared GPUs, and which model is driving the bill? This is effectively the observability analog of unit economics: cost and quality both need a per-tenant denominator, and the inference-component dimension is what makes that denominator available without custom instrumentation. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:25-25] ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:46-58]

### LLM-as-judge quality scoring inherits the evaluator's failure modes

Quality scores in the article are computed with an LLM-as-judge pattern (Claude Sonnet on Amazon Bedrock as the evaluator), and the article flags three governance constraints worth taking seriously: confirm the evaluator's terms permit judging other models' outputs, verify data-residency requirements, and pin the evaluator to a specific version so scores stay comparable over time. The version pin is the most consequential and easiest to skip — if the evaluator model silently updates, a composite score trending down may reflect judge drift rather than product degradation. This mirrors classical metric-instrument drift: the measurement apparatus is part of the system under test. A practical corollary the article only implies: judge latency is itself tracked (evaluation latency is a first-class quality metric), because quality sampling competes for the same budget as serving and can lag real-time by design. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:72-72]

### Silent degradation is a monitoring-architecture problem, not an alerting-configuration problem

The article notes that quality degradation "rarely triggers traditional alerts" — unlike a 5xx spike, a slow slide in relevance or factual accuracy has no natural error rate to page on. The architectural answer is to build quality signals into the same alerting fabric as infrastructure signals: threshold-based Grafana Alerting rules dimensioned per inference component, routed through Amazon SNS into existing SRE triage (Slack, PagerDuty, OpsGenie). The notable choice is reusing the incident pipeline rather than inventing a separate ML-governance channel — quality breaches become ordinary incidents with the same severity classification and correlation automation. The staging guidance matters too: teams that jump straight to quality scoring tend to build alerts they cannot triage, because they lack the infrastructure context to attribute the cause. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:64-66] ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:74-76] ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:15-15]

## 实践启示

1. **Start with quantity, then add quality — in that order.** Establish latency, error, and GPU utilization visibility before investing in LLM-as-judge scoring. Quality alerts are only triageable when you have infrastructure context to attribute the cause, and the staged approach keeps each alert actionable from day one. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:15-15]

2. **Separate quality metrics into their own CloudWatch namespace.** Publish custom quality scores to `/aws/sagemaker/inference-quality/<model-name>` rather than mixing them with enhanced metrics. The clean separation makes namespace-level IAM, retention, and dashboard scoping trivial, and prevents operational dashboards from breaking when quality metric schemas evolve. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:29-34]

3. **Dimension every alert and dashboard by inference component, never by endpoint.** On multi-model endpoints, aggregate metrics hide which model is saturated, degraded, or expensive. Per-component GPU %, cost/hour, and quality scores are what make the "which model is the problem?" question answerable in one glance. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:25-25] ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:74-74]

4. **Pin your LLM-as-judge evaluator to a fixed version.** Before trusting composite quality trends, lock the evaluator model version; otherwise evaluator drift masquerades as model degradation. Also confirm the evaluator's service terms cover judging other models' outputs and that data-residency requirements are met — both are governance blockers that are cheap to check upfront. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:72-72]

5. **Route quality breaches into your existing SRE incident pipeline.** Use threshold-based alerts on quality scores wired through SNS into the same triage tooling (PagerDuty/Slack/OpsGenie) as infrastructure alerts. A separate "ML governance" channel guarantees quality incidents get lower-priority handling; treating them as ordinary incidents gets them correlated and classified automatically. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:74-76]

6. **Diagnose by correlating the two namespaces, not by reading either alone.** Flat latency + falling quality suggests input or evaluator drift; spiking latency + stable quality suggests resource saturation. Build at least one Grafana panel that overlays a quality score against GPU memory % for the same component — that overlay is where the two-dimension strategy pays off. See also [[concepts/cloud-ai-infrastructure|cloud AI infrastructure]] for broader serving-context, and [[concepts/rag-retrieval-augmented-generation|RAG]] for the citation-quality dimension referenced in the quality taxonomy. ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:17-19] ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md:66-66]

## 相关实体
- [[entities/how-aws-smgs-uses-an-ai-powered-conversational-assistant-to-]]
- [[entities/automate-aml-alert-triage-with-amazon-quick-and-snowflake-co]]
- [[entities/滴滴国际化客服质检智能化之路基于-amazon-bedrock-的多语种多业务线质检实践]]
- [[entities/对抗-agent-遗忘kollab-基于amazon-bedrock-agentcore-的团队ai工作空间实践]]
- [[entities/process-financial-documents-using-amazon-bedrock-data-automa]]

→ [[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe|原文存档]] ^[raw/articles/comprehensive-observability-for-amazon-sagemaker-ai-llm-infe.md]

- [[moc/observability-monitoring|MOC]]
