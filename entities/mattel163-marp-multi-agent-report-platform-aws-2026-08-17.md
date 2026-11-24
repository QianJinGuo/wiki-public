---
title: "Mattel163 MARP：多智能体报告自动生成平台（异步长任务 × 证据链 × Agent-as-Code × 项目级凭证）"
created: 2026-08-17
updated: 2026-11-14
type: entity
tags: [multi-agent, aws, bedrock, agentcore, strands, agent-as-code, production, harness, async, security, anti-hallucination]
sources: [raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Mattel163 MARP：多智能体报告自动生成平台

> 生产级多智能体系统的一个可迁移实例：把数周的玩家反馈人工分析压缩到分钟级自动报告。MARP（Multi-Agent Report Platform）由 Mattel163 与亚马逊云科技基于 Amazon Bedrock AgentCore + Strands Agents SDK 构建，其价值不在"多智能体"这个标签，而在四个可复用工程模式：异步长任务架构、证据链防幻觉标准、Agent-as-Code 声明式管理、项目级 IAM 凭证安全。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md]

## 背景与问题

Mattel163 运营 UNO!、Phase 10 等全球 IP 游戏，覆盖 200+ 国家和地区、14 种语言的数千万玩家。每天产生海量玩家反馈：应用商店评论、客服工单、社交媒体舆情、活动问卷、访谈录音。传统人工分析需数周、质量因人而异、多语种协调成本高、时效性差。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:49-66]

直接用 LLM 调用无法解决——报告生成是复杂多步骤工作流：数据不在提示词里（S3 Excel、数百 MB 音频）、上下文超单次容量（数千条多语种评论需分批）、输出有严格格式与证据要求、流程有分支决策（CSV 还是 Excel？多语种混杂？样本量是否足）。这些正是 Agentic AI 的用武之地。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:69-93]

## 四个可迁移工程模式

### 1. 异步长任务架构（SQS + 状态机 + WebSocket）

报告生成是分钟到小时级的长任务，远超 API Gateway 29 秒同步超时。MARP 用三层组合解决：

- **SQS 解耦提交与执行**：任务提交立即返回 task ID（DynamoDB 状态沿 `pending → running → completed/failed` 流转），DLQ 兜底失败（最大重试 3 次），任务不静默丢失。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:123-128]
- **WebSocket 实时流式推送**：Strands SDK 的 `stream_async()` 逐事件产出执行过程（正在调哪个工具、生成了什么），经 API Gateway WebSocket 推给前端，用户"看着"智能体工作——这种透明性显著提升对 AI 输出的信任。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:128-128]
- **双向调用链路**：对话式交互走直连链路（Cognito 认证后直接调 AgentCore Runtime HTTPS 端点，控制面与推理面分离）；长报告任务走异步链路。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:114-119]

### 2. 证据链防幻觉标准（可核验引用约束）

MARP 与 UX 团队共同定义的产出标准，核心是让每条洞察锚定在可核验的原文上：

- 每条结论下附玩家英文原话，原文不得修改、缩写、润色；引文带 role ID 和 VIP 等级（如 `"I love the vibrant colors." (role_id_xxx, VIP4)`）。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:229-237]
- 主题命名必须具体可操作（"回合节奏流畅，操作反馈及时" 合格，"体验好" 不合格）。
- 开放题必须结合前置选择题交叉分析；过滤 "fun"/"good" 等无上下文短答；主题结论须达最低样本量门槛，样本不足明确标注置信度限制。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:232-233]
- **核心思想**：对机器是结构性防幻觉（结论必须锚定可核验原文，模型无"自由发挥"空间）；对人让研究员可按 role ID 回溯任意引用的真实性，几分钟抽检即可建立对整个报告的信心。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:237-237]

这套标准不是工程团队设计，而是 UX 专家把多年分析方法论沉淀成提示词模板——领域专家不写代码，直接定义智能体的专业行为。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:239-239]

### 3. Agent-as-Code：声明式智能体定义与部署

MARP 构建 Agent Toolkit——"Terraform 式"的智能体定义部署工具：一个 YAML 文件声明智能体的全部要素（model、max_tokens、system_prompt、tools、mcps、memory 类型），工具链自动生成标准化代码并部署到 AgentCore Runtime。新增一个分析场景 = 一个 YAML + 一次 `python cli.py validate && deploy` 命令。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:350-398]

两个设计要点：

- **工具注册表（Tool Registry）**：工具集中注册（模块路径 + pip 依赖），部署时自动收集生成 requirements.txt；外部能力（如网络搜索）经 MCP 接入，与自研工具配置层面同等对待。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:392-392]
- **两层配置化体系**：YAML 定义智能体的"能力边界"（模型/工具/记忆/认证），分析模板定义"专业方法论"（分析框架/输出规范/证据规则）。业务团队与工程团队各自迭代自己的层，互不阻塞。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:398-398]

### 4. 项目级 IAM 凭证安全（Prompt Injection 纵深防御）

多项目共用平台的场景：A 项目的智能体手握 S3/Shell 强工具，如何防止读到 B 项目数据？MARP 把权限收敛从提示词层下沉到 IAM 层：

- 用户经 Cognito 认证后，凭证服务从 DynamoDB 查其项目列表，调用 STS AssumeRole 并附加动态生成的 Session Policy，把 S3 权限精确限制在该用户项目前缀下（"权限跟着会话走，而不是跟着角色走"）。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:412-439]
- 系统提示词要求"必须用封装了项目凭证的专用 S3 工具，不要直接调 boto3/AWS CLI"，但提示词约束非硬边界——真正的安全底线由 IAM 保证：即使提示词被注入、即使模型"想"读别的项目文件，凭证本身无越权能力。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:437-437]
- 配合 AgentCore Runtime 会话级隔离，形成"IAM 硬边界 + 工具层封装 + 提示词软约束 + 运行时隔离"纵深防御。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:437-437]

## 与既有实体关系

- 架构立足 [[entities/agentcore-managed-harness|AgentCore Managed Harness]] 与 Strands SDK——MARP 是 AgentCore 托管底座上的一个生产案例。
- 报告生成本质是多智能体编排，对照 [[entities/agent-orchestration-multi-agent-systems|Agent Orchestration 多智能体系统]] 的模式分类。
- 异步长任务 + 可观测流式输出属于生产级 Harness 工程范畴，参见 [[entities/harness-engineering-2026-why-it-matters|Harness Engineering]]。
- 项目级凭证安全与 [[entities/agent-memory-main-contradiction-context-scheduling|Agent 记忆与上下文调度]] 的无害权设计互补（权限/记忆都须按会话最小化）。

## 深度分析

### 1. 异步长任务三件套：队列解耦、状态机恢复、流式透明

MARP 以 SQS 解耦 + DynamoDB 状态机 + WebSocket 推送应对分钟级到小时级长任务：提交即返回任务 ID，状态沿 pending → running → completed/failed 流转，隔天登录仍能恢复视图；DLQ（重试 3 次）兜底限流与数据异常，任务不静默丢失。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:126-135] 流式推送是信任机制而非体验优化——让用户"看着"智能体工作以建立信任。

### 2. 证据链约束：把防幻觉做成结构性规则

产出标准全落在可核验结构上：结论附未经修改的玩家英文原话、引用携带 role ID 与 VIP 等级、主题命名具体可操作、开放题与选择题交叉归纳、无上下文简短回复不得作证据且须达样本量门槛。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:230-236] 对机器是结构性防幻觉；对人，凭 role ID 几分钟抽检即可建立信心，VIP 等级让建议按付费深度加权。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:240-240]

### 3. 两层配置化：能力边界与方法论解耦

Agent-as-Code（YAML 声明模型/工具/记忆/认证）与分析模板（UX 方法论注入）构成两层体系：工程团队迭代能力边界，业务团队迭代专业方法论，新增场景 = 一个 YAML + 一次部署。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:353-401] 对照评审背书成效：主题覆盖与人工高度一致，长尾主题捕捉更全——模板质量而非模型能力成为产出上限。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:261-263]

### 4. 权限跟着会话走：安全底线从提示词下沉到 IAM

Cognito 认证后，凭证服务以动态 Session Policy 调 STS AssumeRole，S3 权限限于该用户项目前缀且 1 小时过期。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:415-442] 提示词"必须用封装凭证的专用 S3 工具"只是软边界，硬边界是即使被注入、凭证也无越权能力；叠加会话级隔离（[[entities/agentcore-managed-harness|AgentCore Managed Harness]]）形成四层纵深防御。

## 实践启示

1. 长任务从第一天按"队列 + 状态机 + 流式推送"设计，同步超时调参无解。
2. 定信任标准优先设计可核验结构（原文引用 + 唯一 ID + 样本量门槛），而非提示词禁令。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:228-242]
3. 领域专家不写代码也能定义智能体行为：方法论沉淀为模板即可新增报告类型。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:217-227]
4. 场景扩张前先建 Agent-as-Code 声明层，避免维护成本线性增长。
5. 开放文件/数据库访问前先做项目级凭证收敛（动态 Session Policy + 自动过期），提示词只当软防线。^[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17.md:440-442]
6. 对照评审是验证产出的最短路径：与人工报告同题对比，重点核查长尾主题覆盖。

## 相关概念

- 多智能体编排 → [[concepts/agent-orchestration-patterns|Agent Orchestration Patterns]]
- Agent 运行安全 → [[concepts/agent-security-architecture|Agent Security Architecture]]
- 记忆架构（权限最小化对照）→ [[concepts/agent-memory-architecture|Agent Memory Architecture]]

→ [[raw/articles/mattel163-marp-multi-agent-report-platform-aws-2026-08-17|原文存档]]
