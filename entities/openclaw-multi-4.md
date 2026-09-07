---

title: "OpenClaw 多用户部署（四）：AgentCore Serverless 容器化"
type: entity
tags: [agent, aws]
created: 2026-05-21
updated: 2026-09-07
review_value: 6
review_confidence: 9
sources: [raw/articles/openclaw-multi-4]
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: thin
review_note: "judged thin-0.75: 容器化部署步骤文; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# OpenClaw 多用户部署（四）：AgentCore Serverless 容器化

> 本页原内容在 2026-09-07 质量闭环中判定为 **thin-0.75**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/openclaw-multi-4.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-harness-engineering-survey-2026|Agent Harness Engineering: A Survey]] — harness工程survey
- [[entities/agentic-overlays-rest-to-a2a-enterprise|Agentic Overlays -- Retrofit Legacy REST Services into A2A Agents]] — REST转A2A模式

## 工程实践
- [[entities/building-multi-tenant-agents-with-amazon-bedrock-agentcore|Bedrock AgentCore 多租户 Agent 构建实践]] — 十组件多租户
- [[entities/build-ai-agents-for-business-intelligence-with-amazon-bedrock-agentcore|Bedrock AgentCore 构建 BI 智能体]] — BI三agent案例
- [[entities/yidian-tianxia-context-engineering-agentic-ai-qcon|一点天下：Context Engineering 与 Agentic AI (QCon)]] — 7114字最全六层上下文版
- [[entities/integrating-aws-api-mcp-server-with-amazon-quick-suite-using-amazon-bedrock-agen|AWS API MCP Server + Quick Suite + Bedrock AgentCore 集成]] — NL→AWS API范式+AgentCore认证模型
- [[entities/break-the-context-window-barrier-with-amazon-bedrock-agentcore|Bedrock AgentCore RLM：突破上下文窗口限制]] — RLM程序化环境
- [[entities/aws-bedrock-serverless-async-inference-multimodal|Amazon Bedrock模型推理的Serverless异步架构 – 处理在线多模态高负载案例]] — 异步推理rv9主版
- [[entities/agentic-ai-data-mesh-aws-s3-vectors-mcp|Building Agentic AI Applications with Data Mesh on AWS]] — data mesh治理架构
- [[entities/finops-devops-dual-agent-cost-optimization|FinOps + DevOps 双Agent 协作：AI驱动的云成本优化实战]] — 双Agent结构化交接+边界意识，$47629隐性成本案例
- [[entities/amazon-bedrock-agentcore-web-search-ga|Amazon Bedrock AgentCore Web Search: 托管式网页搜索能力 GA]] — 托管搜索GA
- [[entities/build-a-healthcare-appointment-agent-with-amazon-nova-2-soni|Build a Healthcare Appointment Agent with Amazon Nova 2 Sonic]] — 语音agent方案
- [[entities/build-ai-powered-dashboard-automation-agents-with-nlp-on-amazon-bedrock-agentcor|Bedrock AgentCore NLP 仪表盘自动化 Agent]] — 仪表盘三代理
- [[entities/introducing-os-level-actions-in-amazon-bedrock-agentcore-browser|Introducing OS Level Actions in Amazon Bedrock AgentCore Browser]] — OS层动作补全浏览器自动化盲区
- [[entities/深度拆解-hermes-agent-记忆系统它修正了-openclaw-的哪层误区|深度拆解 Hermes Agent 记忆系统：它修正了 OpenClaw 的哪层误区？]] — 记忆四问frozen snapshot
- [[entities/amazon-bedrock-agentcore-gateway-mcp-extension|Extending MCP support for Amazon Bedrock AgentCore Gateway]] — MCP网关27k深度
- [[entities/automate-aml-alert-triage-with-amazon-quick-and-snowflake-co|Solution overview]] — AML triage方案
- [[entities/aws-sagemaker-sft-dpo-tool-calling|SFT+DPO 双阶段微调：Qwen3-1.7B Tool Calling 精度提升方案]] — 双阶段微调数据
- [[entities/滴滴国际化客服质检智能化之路基于-amazon-bedrock-的多语种多业务线质检实践|滴滴国际化客服质检智能化之路：基于 Amazon Bedrock 的多语种多业务线质检实践]] — 三条管线38%到86%
- [[entities/deep-agents-bedrock-agentcore-subagent-orchestration-aws|Deep Agents + Bedrock AgentCore：多 Agent 编排 + 隔离基础设施的端到端研究 Agent 实战]] — 两层编排参考实现
- [[entities/agentic-incident-triage-assistant-amazon-quick-new-relic-asana|Agentic Incident Triage Assistant with Amazon Quick, New Relic MCP Server, and Asana]] — incident编排实战
- [[entities/agent-memory-engineering-tax-aws-china-2026|Agent 记忆系统工程税：写入纪律·Prompt Cache 冲突·跨模型容量·Embedding 迁移·自产 Skill 治理]] — 记忆工程税框架
- [[entities/amazon-bedrock-agentic-payments-guardrails|Enable safe agentic payments with built-in guardrails using Amazon Bedrock]] — agent支付护栏15k
- [[entities/data-for-ai明其所耗知其所因让每一分-token-消耗都可量化的全栈实践|Data for AI：明其所耗，知其所因！让每一分 Token 消耗都可量化的全栈实践]] — token可观测四方案
