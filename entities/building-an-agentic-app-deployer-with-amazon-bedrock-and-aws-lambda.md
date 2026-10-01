---
title: "Agentic App Deployer — 规划/供给双 Agent 分离模式（PDI Brew）"
created: 2026-08-07
updated: 2026-10-02
type: entity
tags: [agent, multi-agent, agentic-provisioning, manifest, lambda, serverless, amazon-bedrock, architecture, governance, security]
sources: [raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agentic App Deployer — 规划/供给双 Agent 分离模式（PDI Brew）

## 核心模式：规划 Agent 与供给 Agent 分离

PDI Technologies 为内部长尾工具（成本计算器、表单、仪表板）构建了 PDI Brew：非技术员工用自然语言描述工具，几秒内获得一个已供给完成的多租户 Web 应用（SSO 保护、运行在 AWS 上），无需 Git/终端/DevOps 知识。其架构核心是**"Agentic 不等于每个决策都经过 LLM"**——将 agent 定义（取目标、分解、选工具、行动）拆成两个信任画像完全不同的 Agent：^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

1. **规划 Agent（planning agent）**：捕获意图、访谈用户、生成前端、输出结构化部署清单（manifest JSON——应用名、类型、数据 schema、访问控制设置）。规划逻辑打包为 Vibe App Builder skill，运行在员工已用的 AI 助手里，保持 AWS 侧表面积小。
2. **供给 Agent（provisioning agent）**：AWS Lambda 函数，接收 manifest 后作为**确定性、可审计、会用工具的编排器**执行——校验请求、分类工作负载、选择供给路径、调用 AWS/Microsoft Graph API 作为工具、通过异步自调用处理长任务、返回实时 URL。供给逻辑放在 Lambda 而非聊天会话里是有意设计：供给是每个决策都必须可记录、可复现、无幻觉的工作负载。^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

## 可插拔规划层：同一 manifest 契约

`PLANNER_MODE` 环境变量选择规划路径，两条路径都产出**完全相同的 deploy manifest**，下游一切不变：^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

- **Path A**：Vibe Skill 运行在任意 AI 助手（Claude/ChatGPT/Claude Code）内——规划在 AWS 之外，用户体验丰富。
- **Path B**：Amazon Bedrock `InvokeModel` 作为规划器，运行在 AWS 信任边界内——每个决策落入 CloudTrail、绑定 model-invocation ID、意图数据不出 AWS 边界（满足严格数据驻留要求）。

关键设计点：两个规划器收敛到同一端点与契约，添加 Bedrock 路径是增量改动而非重写；`PLANNER_MODE` 可按组织/工作区/用户钉住。这为未来把 Bedrock invocation 替换为更丰富的托管 agent runtime 留下干净的前进路径。^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

## 供给流程（请求走查）

1. 员工用自然语言描述工具 → 规划器（Path A 或 B）产出 manifest JSON
2. manifest 经 HTTPS 到 `POST /deploy`（API Gateway），Entra ID bearer token（MSAL.js）认证
3. Deploy Lambda 校验 Entra JWT（tenant + expiry）、强制 access-control 模式存在、原子检查 slug 所有权
4. 分类工作负载：static（计算器/图表）vs full-stack（需持久化）
5. static：HTML 包 Entra 认证壳 → S3 → CloudFront 缓存失效 → DynamoDB 注册
6. full-stack：额外供给每应用 DynamoDB 表 + 每应用 Lambda（scoped IAM role）+ 每应用 API Gateway，注入 API URL 到前端
7. 长任务（如创建 Microsoft 365 组）用**异步自调用**——agent 调用第二个自身副本作后台任务，用户请求快速返回
8. 用户经 CloudFront 访问 `<slug>.domain`，每个应用都在 Entra SSO 之后^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

## 双轨计算模型与治理 AI

- **per-app 运行时采用双轨模型**：static 应用只付 S3/CloudFront/DynamoDB 成本；full-stack 应用才有专属 Lambda/API Gateway——scale-to-zero、空闲几乎零成本、无共享服务器可打补丁。
- **治理 AI 网关**：应用可 opt-in 受控 AI 能力（chat/summarize/classify），但只通过**最低权限网关**访问 Bedrock——永不嵌入自己的模型 key；Guardrails + 配额 + 完整审计轨迹。
- **安全与最小权限**：每个应用继承企业 SSO、scoped IAM、HTTPS、集中可观测性——"不存在不安全路径"；默认安全由平台强制而非应用作者选择。^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

## 深度分析

### manifest 契约是 Agent 架构的解耦原语

PDI Brew 最值得提炼的设计不是某个具体服务选择，而是把「用户意图」一次性建模为 manifest JSON 这个稳定契约。规划层因此变成可插拔件：Path A（助手内 Vibe Skill）与 Path B（AWS 信任边界内的 Bedrock InvokeModel）产出完全相同的 manifest，`PLANNER_MODE` 一行环境变量即可切换，且能按组织/工作区/用户粒度钉住。这意味着规划技术的演进（未来换成更丰富的托管 agent runtime）不会波及下游——供给 Agent 和 per-app 运行时零改动。对比常见做法（每个规划入口各写一套对接逻辑），契约先行把「加一条新规划路径」从重写降级为增量添加。这是接口设计在 agent 系统里的直接应用：稳定的不是实现，是数据形状。^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

### 为什么确定性供给层不能放进聊天会话

原文的关键论断值得展开：**供给是每个决策都必须可记录、可复现、无幻觉的工作负载**。聊天会话天然不满足这三条——上下文不可重放、温度带来非确定性、模型可能虚构资源名或跳过校验步骤。PDI Brew 的解法是把 LLM 的活动范围压缩到「意图→manifest」这一段，让 Lambda 函数从 manifest 出发以纯确定性代码编排所有资源创建。同样的分工逻辑也解释了异步自调用（`Event` 类型调用自身）处理 M365 组目录传播这类长任务：用 serverless 原语实现 agent 的「后台任务」模式，同时保持每一步都在 Lambda 执行信封的可审计范围内。这与 [[concepts/orchestrator-worker-architecture|Orchestrator-Worker 架构]] 的差异点在于：分离的依据不是功能分工，而是**信任画像**——对话层容忍幻觉，执行层零容忍。^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

### 双轨计算模型本质是治理粒度的设计

「共享 CRUD Lambda vs per-app Lambda 升级」表面是成本优化，实质是把**能力白名单变成治理开关**。大多数应用只是对自己 DynamoDB 表的 CRUD，走共享加权别名路径——一条温暖的、被审计的代码路径服务众多应用。只有当 manifest 声明了封闭白名单内的能力（发邮件、调外部 HTTPS 域名、读指定数据源），应用才升级到专属 Lambda + scoped IAM role，且升级需要管理员审批，审批依据是提交代码的静态分析 + 产出 IAM role 的 drift detection。这个结构的精妙处：默认路径便宜且安全，特权路径昂贵且被关卡——安全约束不是靠事后审计强迫合规，而是靠路径结构让「不安全」成为需要主动申请的例外。^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

### 12-counter 原子预算：把成本失控变成有界可观测事件

治理 AI 网关里最工程化的细节是 DynamoDB `TransactWriteItems` 实现的 12-counter 预算：4 个 scope（global / app / user / app-user）× 3 个窗口（日 / 周 / 月），任何 Bedrock 调用前单次事务预留额度，任一 counter 超限即 `429` 并返回结构化 body 指明触发的限额。其工程意义有三层：(1) 事务性保证高并发下不超卖；(2) 超限从「月底惊吓账单」变成「当下可观测、可归因的事件」（审计含 UPN、app、model、token 数、估算成本、guardrail 结果）；(3) 权限分层——owner 可被授予更高的 app 级预算，但 global 和 per-user 天花板 owner 无权放宽，加上两级终止开关（admin disable 永远压过 owner enable + 平台级一键停 AI），平台团队保留独立于任何应用的硬停止。^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

### Serverless 多租户让「第 100 个应用」几乎零成本

安全模型同样贯穿到底：供给 agent 持有宽权限创建资源，但它创建的 per-app Lambda 继承的 role 只能触达 `pdi-brew-{env}-app-*` 前缀的表——单租户代码 bug 无法读其他租户数据；所有路径无匿名入口（部署 API 校验 Entra JWT，生成的应用包 MSAL.js 壳并逐次访问校验 M365 组成员资格）。每个租户独立 subdomain、独立表、scoped role，无共享计算故无 noisy-neighbor，且各租户独立 scale-to-zero。叠加全链路可观测性（CloudWatch JSON 日志、`PDIBrew` 命名空间指标、CloudTrail 从意图到资源的端到端审计）与按用量计费（20 人团队约 10 个应用月成本仅 $5–15），结果印证了模式有效性：数百个应用上线，部署从数周缩到数分钟，非开发者成为应用作者。选择 Entra ID 而非 Cognito 是企业既有 SSO 的顺势整合，非技术约束。^[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda.md]

## 实践启示

- **企业内部工具长尾用 manifest 契约模式**：先定义「意图→结构化清单」的 JSON 契约，再让任意规划入口（AI 助手 skill、表单、IDE、IM bot）产出同一清单，下游供给层只认契约不认入口。新通道是增量，不是重写。
- **AI 能力收编为平台能力，而非各应用自带 key**：一个网关端点 + 强制 Guardrails + 多 scope 预算 + 审计，把「每应用一个模型 key」的凭证散落和失控支出问题在架构层面消灭。应用作者用 AI 但永远不碰模型端点。
- **IAM 最小权限从供给 agent 就开始设计**：agent 自己可以宽权限，但它产出的每个资源的 role 必须在创建时刻即 scoped（如表名前缀隔离），并配 drift detection 防篡改——不要指望事后收敛。
- **按信任画像切分 agent，而非按功能**：对话/规划层可容忍概率性行为，执行/供给层要求确定性、可记录、可复现。切分线画在「决策是否需要审计重放」上。
- **治理做成路径结构而非合规检查**：默认便宜安全路径 + 白名单能力触发特权升级 + 管理员审批 + 静态分析，让「不安全」成为需要主动申请的例外，而非需要事后发现的违规。相关成本治理框架见 [[concepts/ai-cost-optimization-framework|AI 成本优化框架]]。

## 与 Wiki 现有知识的关联

- 与 [[entities/how-lendingtree-built-a-multi-agent-mortgage-assistant-on-amazon-bedrock|LendingTree 多 Agent 架构]] 同属"生产级多 Agent 架构"家族——都强调编排层与执行层解耦，但本文的核心增量是**规划/供给分离 + manifest 契约**（确定性执行 vs 对话式规划）
- 多 Agent 编排 的信任画像分离实践——规划 Agent 可容忍 LLM 幻觉（对话），供给 Agent 必须确定性（记录/复现/无幻觉）
- [[concepts/orchestrator-worker-architecture|Orchestrator-Worker 架构]] 的变体——这里不是同层 worker 分工，而是跨信任边界的规划→执行流水线
- Agent 部署策略 与 serverless scale-to-zero 的工程实践
- → [[raw/articles/building-an-agentic-app-deployer-with-amazon-bedrock-and-aws-lambda|原文存档]]
