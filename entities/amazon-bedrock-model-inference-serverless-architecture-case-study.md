---

title: "Amazon Bedrock 模型推理 Serverless 架构案例"
type: entity
tags: [agent, api, architecture, aws, inference, model]
created: 2026-05-21
updated: 2026-09-07
review_value: 7
review_confidence: 9
sources: [raw/articles/amazon-bedrock-model-inference-serverless-architecture-case-study]
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.8: dup-correction: 同主题保留更全版本; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# Amazon Bedrock 模型推理 Serverless 架构案例

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.8**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/amazon-bedrock-model-inference-serverless-architecture-case-study.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agent-harness-engineering-survey-2026|Agent Harness Engineering: A Survey]] — harness工程survey
- [[entities/vercel-com-blog-protecting-against-token-theft|Protecting against token theft]] — 推理盗用经济学与防御
- [[entities/amazon-bedrock-mantle-litellm-gateway-2026|Amazon Bedrock Mantle 推理引擎 + LiteLLM 网关统一收敛]] — Mantle引擎收敛

## 工程实践
- [[entities/yidian-tianxia-context-engineering-agentic-ai-qcon|一点天下：Context Engineering 与 Agentic AI (QCon)]] — 7114字最全六层上下文版
- [[entities/hermes-agent-kanban-deep-test-by-wjjagi-2026|Hermes-Agent 官方 Kanban 深度实测：让商业 CLI 工具当 Orchestrator]] — Kanban深度实测+七条bug 7101字rv9全版
- [[entities/www.blocksandfiles.com-5241795|Redis agentic AI flowers with Iris]] — 上下文范式翻转四支柱
- [[entities/shared-infrastructure-isolated-tenants-pool-model-multi-tenancy-with-amazon-bedrock-agentcore|Bedrock AgentCore Pool Model Multi-Tenancy]] — 池模型多租户三级隔离
- [[entities/how-loka-built-a-natural-low-latency-voice-agent-with-amazon|How Loka Built a Natural, Low-Latency Voice Agent with Amazon Nova 2 Sonic]] — S2S vs三步流水线+prompt迭代2.7→3.8
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进|存之有序，治之有矩——Agent 记忆系统的工程实践与演进]] — 写入纪律prompt cache冲突
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr|AgentOps: Operationalize agentic AI at scale with Amazon Bedrock AgentCore]] — 四支柱解析版
- [[entities/aws-reinforcement-fine-tuning-llm-as-judge|AWS 强化微调：LLM-as-Judge 训练范式]] — RFT judge范式
- [[entities/aws-bedrock-agentcore-quality-optimization-flywheel|AWS Bedrock Agentcore Quality Optimization Flywheel]] — 质量飞轮
- [[entities/aws-sagemaker-capacity-aware-inference-fallback|AWS Sagemaker Capacity Aware Inference Fallback]] — 容量仲裁
- [[entities/using-amazon-bedrock-agentcore-openclaw-multi-5|基于 AWS 示例项目，展示如何将 OpenClaw 迁移为基于 Amazon Bedrock AgentCore 的多租户 Serverless 架构]] — 消息渠道验证篇
- [[entities/extending-mcp-support-for-amazon-bedrock-agentcore-gateway|Extending MCP support for Amazon Bedrock AgentCore Gateway]] — MCP三原语统一+OAuth委托网关机制
- [[entities/让-amazon-quick-操作飞书构建远程-mcp-服务的设计实践|让 Amazon Quick 操作飞书：构建远程 MCP 服务的设计实践]] — MetaTool分层注册设计
- [[entities/构建无服务器kiro调度平台用kiro-cli-eventbridge-ecs-fargate实现定时ai任务|构建无服务器Kiro调度平台：用Kiro CLI + EventBridge + ECS Fargate实现定时AI任务]] — 定时AI任务7x24
- [[entities/secure-ai-agents-with-policy-and-lambda-interceptors-in-amaz|Secure AI agents with Policy and Lambda interceptors in Amazon Bedrock AgentCore gateway]] — Cedar策略+Lambda拦截器双模式
- [[entities/amazon-quick-mcp-kdbx-time-series|Amazon Quick integration with time-series databases for market intelligence using MCP]] — 集成短条
- [[entities/构建基于多智能体架构的深度思考交易系统|构建基于多智能体架构的深度思考交易系统]] — 对抗辩论TypedDict状态
- [[entities/aws-sagemaker-ai-agent-guided-workflows-finetuning|AWS SageMaker AI Agent 引导式工作流微调]] — 引导式微调
- [[entities/building-serverless-a2a-gateway-agent-discovery-routing-access-control|构建 Serverless A2A 网关：Agent 发现、路由与访问控制]] — A2A网关三层
- [[entities/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study|Couchbase Capella iQ — 多模型 AI 推理架构的 Bedrock 实践]] — 多模型架构案例
- [[entities/readonly-code-qa-agent-game-business-team-aws-2026|为游戏业务团队构建只读代码问答 Agent：架构、性能与安全实践]] — 只读代码QA：执行环境与数据面分离

## 延伸导航
- [[moc/agent-memory-architecture-decision-points|Agent Memory 架构选择的关键决策点是什么？]]
- [[moc/llm-research-frontiers|LLM 研究前沿]]
- [[moc/agent-engineering-guide|Agent 工程全景指南]]
