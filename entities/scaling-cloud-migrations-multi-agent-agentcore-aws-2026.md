---
title: "规模化云迁移：Bedrock AgentCore 多 Agent 编排框架"
created: 2026-08-21
updated: 2026-09-07
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

## 相关实体

- [[entities/agentcore-harness|AgentCore Harness]]
- [[entities/deep-agents-bedrock-agentcore-subagent-orchestration-aws|Deep Agents 子 Agent 编排]]
- [[entities/building-multi-tenant-agents-with-amazon-bedrock-agentcore|多租户 Agent 构建]]
- [[entities/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools|MCP Bridge]]
- [[entities/market-surveillance-agent-langgraph-strands-agentcore|Market Surveillance Agent]]
- [[entities/evaluating-ai-agents-production-blueprint-strands-agentcore|Agent 生产评估蓝图]]

→ [[raw/articles/scaling-cloud-migrations-with-agentic-ai-on-amazon-bedrock-agentcore|原文存档]]
