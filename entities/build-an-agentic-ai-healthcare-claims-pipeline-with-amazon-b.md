---
title: "Build an agentic AI healthcare claims pipeline with Amazon Bedrock and AWS HealthLake"
created: 2026-08-01
updated: 2026-09-07
type: entity
tags: ['raw', 'article']
sources: [raw/articles/build-an-agentic-ai-healthcare-claims-pipeline-with-amazon-b]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/build-an-agentic-ai-healthcare-claims-pipeline-with-amazon-b.md|原文存档]]

sha256: 661e4fc3ca640636f2e80281858344f1ad71587fdfc371fcdcf6001f250193cb ^[raw/articles/build-an-agentic-ai-healthcare-claims-pipeline-with-amazon-b.md]

## 摘要

AWS 机器学习博客演示如何用 Amazon Bedrock Data Automation（BDA）+ Amazon Bedrock AgentCore + AWS HealthLake 构建医疗理赔（CMS-1500 表单 PDF）自动化处理流水线。流程：提交者把 PDF 上传到 S3 → Lambda 触发 → BDA 用 OCR + ML + 生成式 AI 智能提取结构化数据（Blueprint 模板，输出带置信度分数和 bounding box 的 JSON）→ 运行在 AgentCore 上的 Strands Agent 校验数据——通过 create_fhir_claim 和 search_fhir_resources 两个工具在 HealthLake 中查找 Insured、Patient、Practitioner、Coverage 参照资源，校验通过后创建标准化 FHIR claim 资源 → Amazon SNS 发送技术摘要（给理赔处理员）和患者友好的状态说明。Lambda 作为 agent 工作流的确定性监督者，未处理的文档进入死信队列 ^[raw/articles/build-an-agentic-ai-healthcare-claims-pipeline-with-amazon-b.md]

文章用失败/成功两个场景验证：缺少 Coverage 资源时 agent 生成人类可读的失败说明；成功场景中 agent 处理了 claim ID 与数据库 ID 末位 o/0 字形差异（11-2234-10190 vs 11-2234-1019O），经姓名搜索识别被保险人并汇总 4 项 CPT 诊疗项目共 $660。最佳实践：Design-time AI 优于 runtime AI（编排逻辑确定就用代码显式编码而非靠 MCP 运行时推断；作者用 agentic IDE Kiro 生成 BDA 调用与 agent 工具代码，减少 Bedrock 调用降低成本）、对 agent 做确定性监督。成本示例：BDA $0.04/页（30 字段内）、Claude Sonnet 约 $0.32/文档（约 76K 输入 + 6K 输出 token）、AgentCore $0.0895/vCPU-小时、HealthLake 首 10GB $0.27/小时 ^[raw/articles/build-an-agentic-ai-healthcare-claims-pipeline-with-amazon-b.md]

## 关键要点

- 六步架构：S3 上传 → Lambda 触发 → BDA 提取 JSON → Lambda 调 AgentCore → Agent 查询 HealthLake 并创建 FHIR claim → Lambda 经 SNS 发送成功/错误通知。
- Agent 的健壮性设计：首次用直接方法调用 + 默认搜索参数找参照资源，未命中则用不同搜索参数再试两次，聚焦高置信度属性并报告匹配方式。
- Strands Agent 两个工具：create_fhir_claim（创建 FHIR 理赔资源）、search_fhir_resources（检索患者/保险资源）。
- 部署栈：AWS CDK（≥2.1025）+ AgentCore CLI，模型为 Anthropic Claude Sonnet 4.6，代码开源在 github.com/aws-samples/sample-agenticidptohealthlake。
- 典型差错处理：agent 能识别 claim ID 中字母 o 与数字 0 的差异并改用姓名搜索完成匹配——演示了数据不一致场景下的自动纠偏。

## 来源

- 原文: [[raw/articles/build-an-agentic-ai-healthcare-claims-pipeline-with-amazon-b.md|Build an agentic AI healthcare claims pipeline with Amazon Bedrock and AWS HealthLake]]
