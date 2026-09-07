---
created: 2026-06-10
title: "AWS Bedrock Serverless 异步推理：SQS + Lambda"
type: entity
tags: [aws, bedrock, serverless, async, inference, sqs, lambda]
summary: "SQS+Lambda异步管道：2000并发0%限流/三层timeout配置/mc=RPM/TPM公式/Partial Batch Failure"
sources: [raw/articles/aws-bedrock-serverless-async-inference-sqs-lambda]
review_value: 7
review_confidence: 9
updated: 2026-09-07
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.75: 异步管道第三份; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# AWS Bedrock Serverless 异步推理：SQS + Lambda

> 本页原内容在 2026-09-07 质量闭环中判定为 **dup-0.75**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/aws-bedrock-serverless-async-inference-sqs-lambda.md`，一手来源仍见下方 sources。

## 工程实践
- [[entities/building-multi-tenant-agents-with-amazon-bedrock-agentcore|Bedrock AgentCore 多租户 Agent 构建实践]] — 十组件多租户
- [[entities/using-amazon-bedrock-agentcore-openclaw-multi-2|基于 AWS 示例项目，展示如何将 OpenClaw 迁移为基于 Amazon Bedrock AgentCore 的多租户 Serverless 架构]] — 环境准备步骤篇
- [[entities/build-ai-agents-for-business-intelligence-with-amazon-bedrock-agentcore|Bedrock AgentCore 构建 BI 智能体]] — BI三agent案例
- [[entities/integrating-aws-api-mcp-server-with-amazon-quick-suite-using-amazon-bedrock-agen|AWS API MCP Server + Quick Suite + Bedrock AgentCore 集成]] — NL→AWS API范式+AgentCore认证模型
- [[entities/break-the-context-window-barrier-with-amazon-bedrock-agentcore|Bedrock AgentCore RLM：突破上下文窗口限制]] — RLM程序化环境
- [[entities/aws-bedrock-serverless-async-inference-multimodal|Amazon Bedrock模型推理的Serverless异步架构 – 处理在线多模态高负载案例]] — 异步推理rv9主版
- [[entities/kiro-job-scheduler-eventbridge-ecs-fargate|构建无服务器Kiro调度平台：用Kiro CLI + EventBridge + ECS Fargate实现定时AI任务]] — 无服务器Kiro调度平台三层架构
- [[entities/amazon-bedrock-agentcore-web-search-ga|Amazon Bedrock AgentCore Web Search: 托管式网页搜索能力 GA]] — 托管搜索GA
- [[entities/build-ai-powered-dashboard-automation-agents-with-nlp-on-amazon-bedrock-agentcor|Bedrock AgentCore NLP 仪表盘自动化 Agent]] — 仪表盘三代理
- [[entities/introducing-os-level-actions-in-amazon-bedrock-agentcore-browser|Introducing OS Level Actions in Amazon Bedrock AgentCore Browser]] — OS层动作补全浏览器自动化盲区
- [[entities/aws-一周综述amazon-bedrock-agentcore-付款适用于-aws-的-agent-工具套件等2026-年-5-月-11-日|AWS 一周综述：Amazon Bedrock AgentCore 付款、适用于 AWS 的 Agent 工具套件等（2026 年 5 月 11 日）]] — 周综述有功能详析
- [[entities/real-time-voice-agents-with-stream-vision-agents-and-amazon-nova-2-sonic|Real-time voice agents with Stream Vision Agents and Amazon Nova 2 Sonic]] — S2S单次往返+25插件+多端SDK16815字
- [[entities/build-custom-code-based-evaluators-in-amazon-bedrock-agentco|Bedrock AgentCore 自定义代码评估器]] — 代码评估器
- [[entities/aws-network-firewall-ai-conflict-detection-bedrock|AWS Network Firewall 规则冲突 AI 实时检测方案（部署小指南六）]] — 规则冲突检测分工
- [[entities/theburningmonk-com-2026-06-what-you-need-to-know-about-lambda-microvms|What You Need to Know About Lambda MicroVMs]] — MicroVM沙箱原语对比
- [[entities/asynchronous-agent-invocation-patterns-serverless-pipelines|异步调用模式：Serverless 流水线中调用 Agent（避免空闲计算成本）]] — 异步调用三模式
- [[entities/implementing-resilience-patterns-with-amazon-bedrock-and-llm|Amazon Bedrock + LLM Gateway 实现生产级推理弹性模式]] — 五种渐进弹性模式+LLM Gateway
- [[entities/extract-data-with-on-demand-and-batch-pipelines-dynamically|AWS Bedrock Dynamic Document Extraction Pipeline]] — Bedrock IDP混合路由+prompt版本registry
- [[entities/lambda-microvms-vs-lambda-functions全方位深度对比|'Lambda MicroVMs vs Lambda Functions：全方位深度对比']] — VM级隔离+挂起恢复经济学
- [[entities/ai-teammates-mondaycom-production-ai-agents-bedrock|AI Teammates: How monday.com Runs Production AI Agents on Amazon Bedrock]] — 生产agent架构
- [[entities/mattel163-marp-multi-agent-report-platform-aws-2026-08-17|Mattel163 MARP：多智能体报告自动生成平台（异步长任务 × 证据链 × Agent-as-Code × 项目级凭证）]] — 异步长任务+证据链+Agent-as-Code四模式
- [[entities/how-trends-automates-root-cause-analysis-with-amazon-bedrock|TReNDS 自动化根因分析（Strands Agents + Bedrock）]] — docstring驱动工具+事件驱动RCA
- [[entities/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study|Couchbase Capella iQ — 多模型 AI 推理架构的 Bedrock 实践]] — 多模型架构案例
- [[entities/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式|使用 Amazon Bedrock AgentCore 构建企业级 MCP 服务器：四种架构模式]] — 四种MCP架构渐进迁移

## 延伸导航
- [[moc/aws-cloud-ai-infrastructure|AWS 云 AI 基础设施]]
