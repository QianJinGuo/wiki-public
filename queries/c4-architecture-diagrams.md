---
title: "C4 架构图索引"
type: query
tags: [architecture, c4, diagram, wiki-meta]
created: 2026-06-27
updated: 2026-06-27
---

# C4 架构图索引

从 2062 个 entity 中严格筛选出 20 个适合 C4 架构图的 entity。筛选标准：标题含架构/系统关键词 + 内容有 3+ 个明确组件 + 描述组件间关系。

## Tier 1: 高置信度（8 个）

这些 entity 清晰描述了具有多个组件的软件系统。

| # | Entity | 核心架构 |
|---|--------|----------|
| 1 | [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计]] | Context + Tool + Memory + Safety 四支柱 |
| 2 | [[entities/anthropic-claude-managed-agents-platform-2026|Claude Managed Agents 平台]] | Agent + Environment + Session + Webhook |
| 3 | [[entities/claude-code-architecture|Claude Code 架构]] | 启动链路 + REPL + Query Loop + Tool Runtime |
| 4 | [[entities/hermes-9-module-architecture-winty|Hermes Agent 9 大模块]] | Loop + Prompt + Tool + Memory + Session + Nudge + Skill + Cron + Gateway |
| 5 | [[entities/litellm-aws-ecs-eks-ai-gateway-architecture|LiteLLM 生产级部署]] | Control Plane + Data Plane 分离 |
| 6 | [[entities/agentscope-java-2.0-enterprise-distributed-harness|AgentScope Java 2.0]] | 分布式 Harness：Engine + Provider + Queue + Store |
| 7 | [[entities/openclaw-architecture-800lines|OpenClaw 800行架构]] | Tool + 消息总线 + 子Agent 管理 |
| 8 | [[entities/netflix-real-time-service-topology|Netflix 实时服务拓扑]] | 服务发现 + 依赖追踪 + 实时监控 |

## Tier 2: 中置信度（12 个）

这些 entity 有系统描述但可能需要人工 review。

| # | Entity | 核心架构 |
|---|--------|----------|
| 9 | [[entities/aliyun-kafka-iceberg-zero-etl-architecture-subtraction-2026-06-18|阿里云 Kafka × Iceberg 零 ETL]] | Kafka → Iceberg 无缝入湖 |
| 10 | [[entities/deepseek-cost-migration-system-layer-kv-cache-harness|DeepSeek 成本迁移]] | KV Cache + MoE + Harness 协同 |
| 11 | [[entities/hermes-agent-memory-system-openclaw-comparison|Hermes 记忆系统]] | 短期 + 长期 + 技能三层架构 |
| 12 | [[entities/gateway-architecture-openclaw-claude-hermes-comparison|Gateway 三框架对比]] | OpenClaw/Claude Code/Hermes 统一接入 |
| 13 | [[entities/promptqueue-async-task-queue-opengorilla-integration|PromptQueue 异步任务]] | Queue + Worker + Provider + Verifier |
| 14 | [[entities/headroom-context-compression-cache-stabilization|Headroom 上下文压缩]] | Live Zone + CCR + Byte-level Patch |
| 15 | [[entities/openclaw-service-enterprise-share-system-design|OpenClaw 企业级共享记忆]] | 多租户隔离 + 共享 + 同步 |
| 16 | [[entities/enterprise-openclaw-security-deploy-architecture-guide|OpenClaw 安全部署]] | 网络 + 权限 + 监控 + 审计 |
| 17 | [[entities/building-enterprise-agentic-ai-with-kiro-on-aws|Kiro 企业级 Agentic AI]] | AWS 全栈集成 |
| 18 | [[entities/kiro-job-scheduler-eventbridge-ecs-fargate|Kiro 无服务器调度]] | EventBridge + ECS Fargate |
| 19 | [[entities/aws-bedrock-agentcore-doris-mcp-server|Doris MCP on AgentCore]] | VPC 原生 MCP 部署 |
| 20 | [[entities/fast-fashion-ecommerce-agent-design-8-websocket-voice-system|快时尚电商语音系统]] | WebSocket + Nova Sonic + AgentCore |

## 生成方式

使用 `[canonical wiki 路径已隐藏]` 生成，支持 `--theme light`（默认，wiki 友好）和 `--theme dark`（展示用）。

```bash
# 单个生成
python3 [canonical wiki 路径已隐藏] --json data.json --theme light

# 批量生成
python3 [canonical wiki 路径已隐藏] --batch --indir [canonical wiki 路径已隐藏] --outdir [canonical wiki 路径已隐藏] --theme light
```

## 文件位置

所有 C4 图文件位于 `assets/c4/` 目录，格式为自包含 HTML，可直接在浏览器打开。
