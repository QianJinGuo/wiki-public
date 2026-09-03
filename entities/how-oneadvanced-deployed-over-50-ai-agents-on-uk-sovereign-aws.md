---
title: "How OneAdvanced deployed over 50 AI agents on UK-sovereign AWS"
created: 2026-08-13
updated: 2026-08-13
type: entity
tags: [agent, multi-agent, sovereign-ai, aws, sagemaker, strands, llm, rag, security]
sources: [raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws]
confidence: 0.75
provenance_state: extracted
---

# How OneAdvanced deployed over 50 AI agents on UK-sovereign AWS

## 核心洞察：数据主权约束下的自托管开放权重模型

OneAdvanced（英国企业软件供应商，服务 10,000+ 客户）需要在数据不离开英国的前提下交付 AI 能力。当时目标模型 Llama 4 Maverick 和 Llama Guard 4 尚未在 UK region 的托管服务中可用，因此选择在完全自控的 AWS 基础设施上自托管开放权重 LLM。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md]

这个决策的本质是主权约束驱动的模型托管选择：当托管服务缺 region 时，自托管 vLLM 是唯一能满足数据驻留、安全与隐私标准（ISO 42001 AI 治理认证）的路径。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md]

## 架构组成

- **模型服务**：vLLM 在 SageMaker AI 上服务 Llama 4 Maverick (FP8) 和 Llama Guard 4，运行在 London (eu-west-2) 区域的 p5.48xlarge 实例。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md]
- **Agent 编排**：50+ Strands agents 运行在 Amazon ECS，每个 agent 有自己的 system prompt、工具配置、可选输入表单，agent 配置存储在 DynamoDB。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md]
- **RAG 管道**：S3 上传文档 → markdown 转换 → chunking → embedding 入 pgvector（Aurora PostgreSQL）→ 检索。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md]
- **内容审核**：Llama Guard 4 在请求到达主模型前检查用户输入的有害内容。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md]

## 请求流与治理

典型请求流：用户发送请求 → Llama Guard 4 审核 → 主模型（Llama 4 Maverick）处理 → 工具层/检索。整个方案支持 ISO 42001 AI 治理认证，保持对模型服务基础设施的完全控制。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md]

## 相关实体

- [[entities/strands-agents|Strands Agents]]
- [[entities/strands-agents-cloud-cost-optimizer|Strands Agents 成本优化]]
- [[entities/strands-agents-high-performance-genai-systems|Strands Agents 高性能 GenAI 系统]]
- [[entities/bedrock-agentcore-coding-agent-hosting|Bedrock AgentCore Coding Agent 托管]]
- [[concepts/retrieval-augmented-generation-rag|RAG]]
- [[entities/vllm|vLLM]]

→ [[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws|原文存档]]
