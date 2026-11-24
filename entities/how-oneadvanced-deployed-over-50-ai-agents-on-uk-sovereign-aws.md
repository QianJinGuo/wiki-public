---
title: "How OneAdvanced deployed over 50 AI agents on UK-sovereign AWS"
created: 2026-08-13
updated: 2026-11-22
type: entity
tags: [agent, multi-agent, sovereign-ai, aws, sagemaker, strands, llm, rag, security]
sources: [raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
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

## 深度分析

### 从 Bedrock 原型到自托管：主权约束下的选型路径

OneAdvanced 并非一开始就自托管：他们先在 Amazon Bedrock 上做了两周原型冲刺，完成聊天补全、英国成文法查询 agent、Snowflake 数据集成和图表生成，验证用例后才决定迁移。真正的决策驱动因素是托管服务的 region 缺位——目标模型（Llama 4 Maverick、Llama Guard 4）当时在 UK region 的托管服务中不可用，而医疗、法律等强监管行业客户要求数据必须留在英国法律与监管框架内。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:15-29] 硬件升级的逻辑同样值得注意：从 p4d.24xlarge 换到 p5.48xlarge 的直接动因是 120K–128K token 上下文窗口需求（大文档分析、多轮对话），而非单纯吞吐——先在与 AWS 的 advisory 合作中用 vLLM 负载测试验证，再切生产，并用预留实例折扣压低 GPU 成本。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:60-60] 这一"托管原型 → 主权自托管"的路径与 [[concepts/local-vs-cloud-agent-deployment-strategy]] 中的部署权衡框架一致。

### Agent 规模化的真正杠杆：三元模板 + 无代码构建器

OneAdvanced 在评估 LangChain、LangGraph 等框架后选择 Strands Agents SDK，核心理由是 model-first——没有僵化的工作流定义，支持轮替与 interview 式交互，从想法到部署的路径最短。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:68-70] 50+ agent 能在三周内落地（多数不到一天），关键在于把 agent 压缩成"system prompt + 工具集 + 可选输入表单"的三元模板，运行时配置统一放 DynamoDB，容器化跑在 ECS。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:72-72] 把边际成本真正打下来的是无代码构建器：产品经理、临床医生、业务分析师用拖拽表单设计输入字段，用 @ 语法在 system prompt 中注入表单值，不写代码就能生产 agent；共享工具库（图表、mermaid 图、Snowflake 查询、英国成文法检索、按组织与按查询双重 opt-in 的 web search）进一步复用能力。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:74-76] 另一个值得复制的模式是 interview-style agent：用"第 1/3 问"式的结构化提问逐步构建上下文，而非要求用户一次性给全信息——引导式聚焦交互的产出优于开放式 prompt。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:78-78] 这种多 agent 编排思路可对照 [[concepts/agent-orchestration-patterns]]。

### RAG 的务实取舍：自建检索层 + pgvector 起步

文档转换为 markdown 后切分 2,048-token 片段再嵌入 pgvector，embedding 选 intfloat/multilingual-e5-large-instruct，看重多语言支持与指令跟随能力；超长文档用递归摘要处理超出分块的内容。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:82-82] OneAdvanced 没有采用现成 RAG 框架，而是自建内部称为 "Llamadex" 的检索系统，换取对管道的完全控制；pgvector 的入选理由是简单与上线速度——但随着方案成熟，他们正在重新评估这一选型。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:84-84] 检索结果附带来源引用，用户能直接看到回答用了哪些文档，透明度成为信任的一部分。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:86-86] 分块策略细节可参照 [[entities/rag-chunking-optimization-2025]] 与 [[concepts/rag-retrieval-augmented-generation]]。

### 治理与质量闭环：从 ISO 42001 到 LLM-as-a-judge

安全设计贯穿各层：个人与组织文档空间完全隔离（连 OneAdvanced 员工都读不到用户数据），查询与响应不保留、不用于模型训练，GuardDuty 覆盖 S3 存储层的威胁检测；OneAdvanced 是英国/欧洲最早取得 ISO 42001 AI 治理认证的机构之一，也是 EU AI Pact 签署方。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:92-98] 2025 年 7 月上线后，质量闭环同样具体：用 LLM-as-a-judge 加情感分析做持续质量评估，roadmap 包括帮助用户打磨 prompt 的后端系统，以及模型版本切换期间并行运行新旧 LLM——用双模型并行保护终端用户稳定性。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:102-112] 这与 [[entities/llm-as-a-judge-agent-eval-offline-huolala-2026]] 的离线评估实践相互印证，也呼应 [[concepts/ai-ethics-responsible-ai]] 中的负责任 AI 主题。

## 实践启示

1. **先原型后迁移**：先在托管服务上用两周冲刺验证用例，再投入自托管 GPU 基础设施；托管服务的 region 缺位是迁移的触发器，而非起点。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:27-29]
2. **审核模型的误拒率是产品问题**：Llama Guard 3 的高误拒率直接推动了升级到 Llama Guard 4——安全过滤器的假阳性会实质性伤害用户体验，必须像评估主模型一样评估它。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:62-62]
3. **Agent 规模化靠压低边际成本**：把 agent 抽象为 system prompt + 工具 + 表单的三元模板，配合无代码构建器与共享工具库，让非工程师成为 agent 生产者，三周内从 1 到 50+。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:64-78]
4. **引导式交互优于开放式输入**：interview-style agent 用结构化多轮提问构建上下文，产出质量更高——该模式可直接移植到任何对话型 agent。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:78-78]
5. **基础设施可以"先简单后演进"**：pgvector 换上线速度，自建检索层保留替换自由——前提是团队成熟后真的回头评估选型，OneAdvanced 明确把这一点写进了演进计划。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:84-84]
6. **模型版本切换期并行运行新旧模型**：作为保护终端用户稳定性的 roadmap 策略，值得所有自托管部署预设。^[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws.md:112-112]

## 相关实体

- [[entities/strands-agents|Strands Agents]]
- [[entities/strands-agents-cloud-cost-optimizer|Strands Agents 成本优化]]
- [[entities/strands-agents-high-performance-genai-systems|Strands Agents 高性能 GenAI 系统]]
- [[entities/bedrock-agentcore-coding-agent-hosting|Bedrock AgentCore Coding Agent 托管]]
- [[concepts/retrieval-augmented-generation-rag|RAG]]
- [[entities/vllm|vLLM]]

→ [[raw/articles/how-oneadvanced-deployed-over-50-ai-agents-on-uk-sovereign-aws|原文存档]]
