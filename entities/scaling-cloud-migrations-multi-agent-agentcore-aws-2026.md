---
title: "规模化云迁移：Bedrock AgentCore 多 Agent 编排框架"
created: 2026-08-21
updated: 2026-09-27
type: entity
tags: [agentcore, aws, multi-agent, orchestration, migration, cloud-migration, aws-ml-blog, strands]
sources: [raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 规模化云迁移：Bedrock AgentCore 多 Agent 编排框架

> **Background**：AWS 官方 ML Blog（2026-08-20）关于用 Bedrock AgentCore 构建多 Agent 编排框架加速企业云迁移的架构案例。AWS Professional Services 构建一套 purpose-built AI agents（Intake / IaC / Governance / SRE），覆盖迁移生命周期从自动发现到主动运维。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

## 迁移程序的三大瓶颈

大型企业数据中心退出（data center exit）迁移程序持续出现三个核心瓶颈：^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

- **手动 intake 开销**：发现阶段（理解 on-prem 架构、清单、依赖、intake 问卷）消耗大量人工。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- **冗余基础设施开发**：工程师为每个应用手写 IaC，缺乏自动化导致重复劳动。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- **被动的迁移后运维**：依赖手动监控与响应式处理，缺乏主动智能检测性能退化与自动修复。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

## 多 Agent 编排框架架构

框架把重复工作移给 AI agents，人类保留决策权。两个 journey：迁移 journey（发现→部署）与运维 journey（迁移后监控）。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

- **Intake Agent（Phase 1）**：自动化应用发现与目标架构定义（含依赖映射）。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- **IaC Agent（Phase 2）**：生成符合安全最佳实践与标准的 IaC 代码。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- **Migration Intelligence and Governance Agent**：跨 Jira/Confluence/Webex 的自动化组合报告、well-architected 评估与迁移治理。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- **SRE Agent（Phase 3）**：迁移后主动监控与自动修复。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

AWS 托管服务互补：AWS DMS（生成式 AI 辅助 schema 转换 + 自动化 cutover）、AWS Transform（应用特定现代化）。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

## 组件如何连接

每个 agent 是 Strands agent（foundation model + system prompt + tool set），由 AgentCore runtime 以 serverless 环境托管，提供 session isolation 与 multi-agent orchestration。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

- 每个 agent 通过 AgentCore Gateway 调用 MCP tools（把 API/Lambda/现有服务转成 MCP-compatible tools）；AgentCore Identity 用 scoped IAM roles + IdP 认证每次调用。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- AgentCore memory 存储 agent session state 与共享 context——Intake Agent 完成发现后把目标架构与依赖映射写入 memory，IaC Agent 读共享 context 开始生成代码，无需手动交接（跨 300+ 应用跟踪迁移进度）。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

## 可迁移的工程要点

- **agent-in-code 模式**：用 Strands `Agent(model, system_prompt, tools=[gateway])` + `BedrockAgentCoreApp` 定义，entrypoint 返回产物，AgentCore 处理 session isolation 与 scaling。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- **MCP client_credentials grant**：url+auth 让 SDK 运行 client_credentials grant 并在过期时重铸 token，避免静态 bearer token 过期。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- **Guardrails 接入**：`guardrail_id/version/trace` 直接绑定 model；`guardrail_intervened` stop_reason 需显式处理（返回 blocked_by_guardrail 而非报错）。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]
- **共享 memory 作 agent 间交接媒介**：用 AgentCore memory 持久化 agent 产物与共享上下文，实现多 agent 流水线的自动 handoff（对应 2026-08-06 家族分裂判据中的可迁移编排架构模式）。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md]

## 深度分析

### 瓶颈→Agent 的一一映射是框架真正的组织原则

框架没有做一个“通用迁移 agent”，而是把迁移生命周期切成三个瓶颈（发现、IaC 开发、迁移后运维），每个瓶颈配一个 purpose-built agent。这种 1:1 映射让收益可以单独度量：IaC Agent 最早部署、效果最直接（单应用 IaC 开发从 3–4 周降到分钟级），Intake Agent 消化 intake 开销，SRE Agent 收尾主动运维。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md.md:29-37,161-163] 更深一层的判断是：单纯加工程师解决不了 300+ 应用在激进时间线上的迁移，必须改变工作被组织的方式——让 AI agent 承接重复性高量工作，人类只保留决策、审批与战略。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md.md:273-275]

### 共享 memory 是 agent 间流水线的“接口协议”

组件之间不靠 prompt 交接、也不靠人工传文件，而是通过 AgentCore memory 持久化产物与共享 context：Intake Agent 完成发现后把目标架构与依赖映射写入 memory，IaC Agent 直接读取开始生成代码，无需人工交接，跨 300+ 应用的迁移进度同样靠它跟踪。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md.md:73-77] 这让 memory 从“对话记忆”升级为流水线交接介质——上游 agent 的输出 schema 就是下游 agent 的输入契约。这是可迁移到任何多 agent 流水线的编排模式：agent 之间松耦合，只通过 memory 中的产物 schema 相连。

### 安全不是外挂层，而是结构性约束

每个 agent 动作要穿过多层控制：AgentCore Identity 用 scoped IAM role 认证每次调用，Gateway 的自定义 MCP tools 在边界校验输入 schema 并拒绝畸形输入，Policy in AgentCore 用 Cedar 规则评估每次 tool call（计算潜在变更范围、检查与并发 wave 的依赖冲突、确认合规窗口有效），Bedrock Guardrails 在推理层对每个 prompt 和每个模型响应做内容过滤与 contextual grounding 检查。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md.md:179-183,229-236] 凭证与敏感值完全不进 agent context——Identity 在运行时从集中式 credential provider 解析 secrets；所有 agent 动作经 Observability 与 CloudTrail 写入不可变审计链。安全标准本身也被编码进 Confluence 上的企业规范并直接作用于生成的 IaC，安全更新经 pattern 基线在下次部署周期自动传播。

### 人工审批门是设计原则，不是合规补丁

Governance Agent 的自动动作（更新 Confluence 状态、创建 Jira task、生成 ServiceNow 升级单）与 SRE Agent 的修复建议（数据库 right-sizing、性能调优、存储分层、扩缩容）执行前都需要显式人工批准；“no agent acts autonomously on production systems” 贯穿整个 agent 套件。^[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore.md.md:205,213-215,233] approval-gated 架构的深层逻辑：agent 完成约 99% 的重复劳动并生成可选项，人类只保留决策点——由此自动化的速度收益与治理风险被解耦，两者不再互相牵制。

## 实践启示

1. **从最痛、最可度量的环节切入**：IaC Agent 是套件中第一个部署的，因为手写 IaC（每应用 3–4 周）是量化最清晰的瓶颈；先摘低垂果实建立组织信任，再向 Intake / SRE 扩展。
2. **把组织标准编码成 pattern 构造体**：网络配置、安全组规则、IAM roles、CloudWatch 告警、强制 tagging 固化为可复用 IaC patterns，agent 只按 pattern 生成——“no wave can deviate from the approved IaC patterns baseline”，一致性由此免费获得，且免去逐应用手写。
3. **用共享 memory 做自动 handoff，别靠人工传产物**：定义好上游输出契约（目标架构 + 依赖映射），下游 agent 直接从 memory 读取启动，消除人工交接队列。
4. **token 用 client_credentials grant 自动重铸**：MCP client 配 url+auth 让 SDK 在过期时重新铸发 token，静态 bearer token 会失效——长时运行 agent 的基本功。
5. **显式处理 guardrail stop_reason**：`guardrail_intervened` 应返回 `blocked_by_guardrail` 状态而非抛异常，让调用方能区分“被拦截”与“故障”，并保留 guardrail trace 进审计。
6. **每个落地生产的自动化动作前加 approval gate**：人批的是“执行与否”，而不是审核全部生成物——保持 agent 支持决策而非替代决策。

## 相关实体

- [[entities/agentcore-harness|AgentCore Harness]]
- [[entities/deep-agents-bedrock-agentcore-subagent-orchestration-aws|Deep Agents 子 Agent 编排]]
- [[entities/building-multi-tenant-agents-with-amazon-bedrock-agentcore|多租户 Agent 构建]]
- [[entities/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools|MCP Bridge]]
- [[entities/market-surveillance-agent-langgraph-strands-agentcore|Market Surveillance Agent]]
- [[entities/evaluating-ai-agents-production-blueprint-strands-agentcore|Agent 生产评估蓝图]]

→ [[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore|原文存档]]
