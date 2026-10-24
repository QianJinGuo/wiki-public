---
title: "Couchbase Capella iQ — 多模型 AI 推理架构的 Bedrock 实践"
type: entity
tags: [aws, bedrock, inference, architecture, couchbase, case-study, llm]
created: 2026-07-24
updated: 2026-10-25
sources: [raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study]
confidence: 0.70
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Couchbase Capella iQ — 多模型 AI 推理架构的 Bedrock 实践

> **Background**：本文档基于 Couchbase 在 AWS Blog 上发布的 Capella iQ AI Assistant 架构案例，介绍了如何用 Amazon Bedrock 构建多模型、跨 Region 的生产级 AI 推理架构。^[raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study.md]

Couchbase 的 Capella iQ 是一个 AI-powered 开发者助手，能够生成 SQL++ 查询、推荐索引、支持多轮对话。随着企业采用增长，Couchbase 将其 AI 应用扩展为支持多个 Foundation Model（FM）提供商的**模型无关推理架构**，核心是使用 Amazon Bedrock。^[raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study.md]

## 架构设计

Capella iQ 的推理架构基于 Amazon EKS 集群部署在 AWS 双 Region（us-east-1, us-west-2）控制平面内：^[raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study.md]

- **cp-api pod**：主 API 服务，接收开发者请求、编排推理调用
- **cp-internal-api pod**：内部服务通信和模型路由逻辑
- **cp-ns pod**：命名空间级配置管理，包括模型供应商设置和租户级覆盖

请求通过 VPC Interface Endpoint 私有连接到 Bedrock Runtime，流量全程不经过公网。Cross-Region Inference (CRIS) 在 us-east-1/2 和 us-west-2 之间自动实现故障切换和负载分发。^[raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study.md]

## 模型评估框架

在模型选型前，Couchbase 建立了覆盖所有 Core Capella iQ 工作流的 benchmark 套件，评估维度包括功能正确性、确定性、延迟和格式一致性。Anthropic 的 Claude Sonnet 4.5 基于 BIRD 方法在内部评估中达到约 76% 准确率，被选定为首个生产模型。^[raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study.md]

## 关键技术决策

- **Provider abstraction**：早期投资于模型供应商抽象层，使 Bedrock 集成不干扰用户体验，长期保持模型切换灵活性
- **CRIS 简化运维**：跨 Region 推理无需预置容量或自定义故障切换逻辑，处理突发流量更高效
- **Multi-model readiness**：生产级多模型支持需要持续投入——benchmarking 基础设施、prompt 工程、可观测性应作为持续性工作流而非一次性建设^[raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study.md]

## 未来规划

Couchbase 正探索通过 Bedrock Custom Model Import 部署微调的小型模型来优化成本，为高容量、边界明确的工作负载（如索引推荐和查询解释）蒸馏任务专用模型。^[raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study.md]

## 深度分析

### 多模型路由：一个助手，几种不同的活

Capella iQ 的工作负载并不同质：SQL++ 生成是受限代码综合（对语法与格式正确性极敏感，对措辞不敏感），查询解释是自然语言转述（对可读性敏感，对精确 token 不敏感），索引推荐是结构化判断加取舍论证，iQ Insights 与多轮对话则要求长上下文与意图保持。这几类任务的最优模型尺度不同：代码生成需要更强的推理与指令遵循，解释类任务用更小模型即可达标，而未来的蒸馏路线（用 Custom Model Import 部署微调小模型）恰恰说明高容量、边界明确的任务应下沉到便宜模型。原文没有披露检索、embedding 或重排环节，也没有给出各任务的分模型单价与延迟数字，因此这里的路由逻辑只能是定性的：按任务挑模型，把最贵的推理能力留给最难的环节，用评估分数而不是模型规模决定谁能上岗。单一大模型同样能跑通演示，但它在每一类任务上都按最贵的那档付费，且任何一次模型退版都会同时影响所有工作流——这正是把路由拆开的组织理由。

### 为什么托管 Bedrock 胜过自建推理栈

对一个数据库厂商而言，模型推理是支撑功能而非核心产品。自建意味着要持有 GPU 容量规划、驱动与推理框架升级、批处理调度、故障切换一整套能力，而这些能力与 Couchbase 的差异化（分布式文档数据库）无关。Bedrock 把这套东西变成 API 调用：无需预置容量即可吸收突发流量（原文正是以 without pre-provisioned capacity 描述 CRIS 的价值），模型目录持续增长使新模型上线不必重构推理管道，单个 API 覆盖多个模型家族又正好支撑 provider-agnostic 的目标。更关键的是组织成本——对一个不以 [[concepts/inference-optimization|推理优化]] 为业的团队，运维模型栈的隐性开销（值班、容量、版本、区域）会持续侵蚀产品迭代速度。安全侧同样成立：VPC Interface Endpoint 让 prompt 与响应不经过公网，客户获得 AWS 的安全姿态、数据驻留控制与 SOC/HIPAA/ISO 合规覆盖。

### 评估框架才是可复用资产

这个案例里最容易被忽略、也最可迁移的部分不是架构图，而是那套 benchmark：覆盖全部核心工作流（SQL++ 生成、索引推荐、查询解释、iQ Insights、多轮对话）的标准化 prompt/response 测试集，四个打分维度（功能正确性、确定性、延迟、格式一致性），以及一个以 BIRD 方法为参照的 76% 内部准确率基线。它的真正作用不是选出 Claude Sonnet 4.5，而是把"换模型"从一次冒险变成一次可回归的发布：新模型先在既有测试集上跑分，无关键回归才进入候选。原文还点出跨模型比较的实际困难——tokenization、上下文窗口处理、响应格式差异必须归一化后才谈得上公平打分，于是团队建了自动化评估管道保证可复现。这条经验对任何做多模型路由的系统都成立：[[concepts/evaluation-harness-design|评估框架]] 是资产，模型是租来的。

### 工程细节：延迟预算与"错比慢更贵"

多模型流水线把一次用户请求拆成若干次模型调用，延迟因此是叠加量而非单点指标。原文虽未给出逐跳延迟，但指出跨 Region 故障切换是最大的验证难点：部分端点降级与区域限流难以在开发环境复现，需要自建测试夹具，并与 AWS 共同调优 timeout 与 retry。这暗示延迟预算必须按跳分配，且 retry 策略要计入最坏路径——否则一次限流重试就能吃掉整个 SLO。另一条隐含约束是生成式 SQL 的安全属性：一条错误的 SQL++ 语句代价远高于一句慢回答。于是 [[concepts/prompt-engineering-fundamentals|prompt 工程]]、租户级配置（cp-ns）与响应归一化这三层实际承担了护栏职责：模板限定输出形态，命名空间配置限定可用模型与租户覆盖，归一化在返回前做格式校验。多模型可观测性也在此变得必要——没有按模型、按任务、按 hop 的追踪，就无法判断一次质量回退来自模型、prompt 版本还是路由。

### 这个架构该叫什么

命名清楚才谈得上复用：数据库 copilot 是**一个控制平面背后的窄模型组合**（a composition of narrow models behind one control plane）。控制平面负责鉴权、会话、模板选择、租户配置、路由与归一化；窄模型各自只做一件事，可独立替换、独立评估、独立计费。它与"单模型 chat-on-your-data"演示的差别有三点：质量由每个任务的评估分数保证，而非单模型的综合能力；模型替换是命名空间层的配置变更（原文：no code changes, no downtime），不是代码改动；成本可按任务粒度优化。换句话说，前者把模型当成可替换的执行器，后者把模型当成产品本身。

## 实践启示

1. **按任务写路由表，而不是按模型写功能。** 先把工作负载拆成代码生成、解释、推荐、洞察、多轮对话几类，为每类标注质量下限与延迟上限，再决定谁能承接；路由表应是配置，而不是散落在代码里的 if-else。
2. **选模型之前先建评估集。** 测试集要覆盖全部工作流，并同时打分正确性、确定性、延迟与格式一致性；没有基线分数，任何模型替换都是凭感觉的赌博。
3. **给每一次模型调用分配延迟预算。** 把端到端 SLO 拆到每一跳，把重试与故障切换计入最坏路径，否则跨区域容错会在高峰期悄悄吃掉余量。
4. **把生成的 SQL 当成不可信输入。** 模板限定输出形态、返回前做语法与格式校验、对高危语句（DDL 与写操作）加确认或拦截——错误的语句比慢的答案代价高一个量级。
5. **模型调用藏在接口后面。** 业务代码只依赖"生成 / 解释 / 推荐"这类能力接口，供应商与模型名只出现在配置层，这样换模型才可能做到零代码改动、零停机。
6. **按功能追踪 token 成本。** 多模型架构的成本优势只在被测量时才存在：按任务、按租户记录 token 与调用次数，才能判断哪些任务值得蒸馏下沉。
7. **把 benchmark 与可观测性当持续工作流。** 原文的教训是"不是一次性建设"：模型目录在变、prompt 在变，评估集与追踪管道必须随之维护。

## 相关实体

- [[entities/amazon-bedrock-model-inference-serverless-architecture-case-study|Amazon Bedrock 模型推理 Serverless 架构案例]] — 不同的推理架构（SQS 异步 vs EKS 同步）
- [[entities/amazon-bedrock-cross-region-inference-cris-eu-gdpr|Amazon Bedrock CRIS 跨 Region 推理]] — CRIS 在 GDPR 场景的应用
- AWS Bedrock Multi-Agent Collaboration

→ [[raw/articles/couchbase-capella-iq-multi-model-ai-architecture-bedrock-case-study|原文存档]]
