---

title: "OpenClaw 多用户部署（四）：AgentCore Serverless 容器化"
type: entity
tags: [agent, aws]
created: 2026-05-21
updated: 2026-11-20
review_value: 6
review_confidence: 9
sources: [raw/articles/openclaw-multi-4]
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: thin
review_note: "judged thin-0.75: 容器化部署步骤文; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---
# OpenClaw 多用户部署（四）：AgentCore Serverless 容器化

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

## 深度分析

### 为什么 Phase 2 是 Replatform 策略的“分水岭”

原文把 Phase 2 定位为“整个迁移中最核心的一步”，这个判断的实质是：改造对象从“进程”上升为“运行时”。Phase 1 建基础设施时，OpenClaw 仍被理解为“一台服务器上的 Node.js 进程”，只是搬到了更好的机器上；到 Phase 2，代码被封装成 ARM64 容器镜像交给 AgentCore 托管，运行单位变成了按需启动、用完即毁的 microVM。这一步做完了，后面所有多租户能力（会话隔离、按需计费、自动扩缩）都是它的自然推论，而不是额外的功能点。

容器架构上有一个容易被忽略的强约束：Runtime 底层 microVM 跑在 Graviton（ARM）处理器上，镜像必须产出 ARM64 格式，否则连启动都做不到^[raw/articles/openclaw-multi-4.md:73-74]。原文给出的解法是用 CodeBuild 的原生 ARM64 构建机，而不是本地 Docker + QEMU 跨架构模拟——前者是托管服务里“顺手”解决的架构问题，后者则会在开发者本地环境埋下“我这能跑、CI 构不出来”的典型故障。对任何要迁到 Graviton 类托管运行时的 Agent 项目，镜像架构是应该在 Phase 1 就想清楚的前置条件。

### 会话隔离的真实语义：不是“分进程”，是“内存清零”

很多多租户方案止步于“每个用户一个进程/容器”，原文对 AgentCore 隔离的描述更深一层：Runtime 按会话 ID 分配独立 microVM，隔离 CPU、内存、文件系统三个层面，且“会话空闲到期或结束后 microVM 终止、内存清零”^[raw/articles/openclaw-multi-4.md:81-84]。这意味着内存态是天然的临时存储，任何需要跨调用存续的状态都必须显式外置。项目用 Session Storage 把 /mnt/workspace 挂载为会话持久化路径（同会话 ID 再调用时自动恢复），同时保留 S3 备份兜底——持久化是“挂载点 + 对象存储”的双层设计，而非单点。这一点与 [[entities/amazon-bedrock-agentcore-data-persistence-file-system-session-storage-efs-s3|AgentCore 数据持久化（EFS/S3 会话存储）]] 中总结的持久化模式一致。

### 业务层三组件的职责切分逻辑

Phase 3 部署的消息路由、定时任务、用量监控不是随便的三个服务，而是按“流量形状”切分的：消息路由是实时短请求（webhook 进来 → 调容器 → 回复出去），定时任务是低频长延后（EventBridge Scheduler 触发 → 查任务上下文 → 调容器），用量监控是纯异步旁路（订阅 Bedrock 日志 → 解析入库）。三者各自独立扩缩、独立监控、独立计费。原文还点明了一个部署顺序上的硬依赖：Router Lambda 和 Cron Lambda 必须知道 Runtime ID，所以 Phase 3 只能排在 Phase 2 之后——这不是流程惯例，而是资源引用图的拓扑序。

路由层的存储设计也值得拆解：openclaw-identity 用单张 DynamoDB 表，靠 PK/SK 前缀（CHANNEL#、USER#、SESSION、BIND#、ALLOW#、CRON#）区分六种记录类型，并配 KMS 加密和 PITR 时间点恢复^[raw/articles/openclaw-multi-4.md:131-134]。单表设计在这里成立的理由是：所有记录的生命周期都被 Router Lambda 同一套读写路径覆盖，访问模式高度内聚；如果六类记录各有独立的读写服务和容量画像，单表反而会成为热点和权限混杂的根源。

### 用量监控：从“事后看账单”到“实时旁路 + 异常检测”

Token 监控层的架构是迁移后新增的能力，本质是把“成本可观测”做成了独立的 Serverless 旁路：CloudWatch Subscription Filter 实时把 Bedrock 调用日志转给 Token Metrics Lambda，逐条解析 Token 数、估算成本、写 DynamoDB，同时推自定义指标。存储侧用单表 + 3 个 GSI 分别支撑按渠道、按模型、按日期的成本排名查询，配 90 天 TTL 自动清理。更有价值的是告警的三层设计：每小时 Token 总量与估算成本两个独立预算告警（阈值来自 cdk.json 的 budget 参数）、再加一个基于历史数据学正常范围的 Anomaly Detector，偏离 2 个标准差即触发——专门兜住“总量没超预算但模式异常”的情况（如半夜突然大量调用）^[raw/articles/openclaw-multi-4.md:220-232]。静态阈值拦得住超标，拦不住“没超标但不正常”，异常检测补的正是这个缺口。多租户场景下这笔账最终要按租户分摊，没有这条旁路，成本归因只能事后手工对账。更深层的成本纪律讨论见 [[entities/agent-memory-engineering-tax-aws-china-2026|Agent 记忆系统工程税]] 与 [[entities/amazon-bedrock-agentcore-runtime-deep-dive-and-scenario-analysis|AgentCore Runtime 深度解析]]。

## 实践启示

1. **迁移 Agent 到托管 Serverless 运行时，先确认镜像架构约束再写代码。** AgentCore Runtime 要求 ARM64（Graviton microVM），用托管构建服务（如 CodeBuild 原生 ARM 构建机）产出镜像，避免本地 QEMU 模拟在 CI 阶段炸掉；架构选型应在基础设施阶段锁死，而不是容器构建时才发现。
2. **把“会话隔离”理解为“内存不可信”。** microVM 用完即毁、内存清零，任何跨调用状态必须显式写入持久化挂载点或对象存储，并做双层兜底（挂载点恢复 + S3 备份）；评估任何多租户运行时都要问同一句话：会话结束后，什么东西没了？
3. **业务层按流量形状切组件，而不是按功能模块切。** 实时路由、延后定时、异步旁路三种流量形状对应三种扩缩与监控画像，拆成独立 Serverless 组件各自计费；同时先画清资源引用图（谁依赖谁的 ID），部署顺序跟着拓扑序走，不要按组件“重要性”排期。
4. **单表设计只有在访问模式高度内聚时才成立。** 用 PK/SK 前缀区分多种记录类型的前提是同一读写路径覆盖全部记录；当记录类型的容量画像、权限边界或访问服务开始分化时，及时拆表比维持单表优雅更重要。
5. **成本可观测要做实时旁路 + 双层告警，不能等账单。** 订阅模型调用日志逐条解析入库（GSI 支持按渠道/模型/日期归因），静态预算告警之上再加基于历史基线的异常检测（如 2 个标准差偏离），专门捕获“未超标但异常”的调用模式；多租户计费分摊完全依赖这条旁路的数据完整性。
6. **配置驱动的预算阈值是治理入口。** Token 与成本告警阈值来自 cdk.json 的 budget 参数而非硬编码，改预算=改配置=走部署流程，避免“调阈值”沦为绕过评审的手工操作。

→ [[raw/articles/openclaw-multi-4|原文存档]]
