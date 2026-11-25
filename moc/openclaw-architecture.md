---
title: OpenClaw 架构与部署
created: 2026-05-15
updated: 2026-11-25
type: moc
tags: [openclaw, architecture, agent-runtime, multi-tenant, security]
description: OpenClaw 的运行时架构、工具总线、子 Agent、企业部署与多租户迁移路径。
---

# OpenClaw 架构与部署

> OpenClaw 是开源、自托管的 AI Agent 平台，核心特征是多通道接入、本地优先、工具与子 Agent 编排。AWS 官方博客补充的是企业级部署、多租户与迁移案例，不等于 OpenClaw 由 AWS 开发。

## 核心架构

- [[entities/openclaw-architecture-8-part-summary|OpenClaw 源码 8 篇拆解]] — 从入口到工具、消息循环与子 Agent 管理。
- [[concepts/openclaw-architecture|OpenClaw 架构概念页]] — MessageBus、会话与运行时结构。
- [[entities/openclaw-agent-loop-design-patterns|OpenClaw Agent Loop 设计模式]] — 从手动提示到循环运行。
- [[entities/openclaw-comprehensive-guide|OpenClaw 完全指南]] — 总体机制与工程实践。
- [[entities/openclaw-hermes-source-code-agent-architecture-review|OpenClaw × Hermes 源码架构对照]] — 两套运行时的边界与相似点。

## 工具、Skills 与子 Agent

- [[entities/openclaw-leveraging-nova-mme-s3-vector-implement-skill|Nova MME + S3 Vector Skill]] — 按需召回与工具实现。
- [[entities/openclaw-security-and-feature-enhancement-practices|OpenClaw 安全与功能增强实践]] — 自托管环境的权限、凭证、网络与注入风险。
- [[entities/hermes-agent-skills-source-code-analysis-shuge|Hermes Skills 源码拆解]] — 可作为 Skill 渐进加载与安全扫描的对照。
- [[moc/agent-engineering-guide|Agent 工程全景指南]] — 回到 harness、loop 与子 Agent 的通用框架。

## 企业部署与多租户

- [[entities/enterprise-openclaw-security-deploy-architecture-guide|企业级 OpenClaw 安全部署]] — 生产部署基线。
- [[entities/openclaw-from-personal-assistant-to-customer-service-a-trust-model-flip|从个人助手到客服：信任模型翻转]] — 会话隔离、渠道接入、Guardrails 与知识库。
- [[entities/openclaw-service-enterprise-share-system-design|企业共享记忆系统]] — 多客户服务下的上下文共享。
- [[entities/openclaw-multi-1|多租户迁移 #1]] — 单机到 Serverless 多租户的起点。
- [[entities/openclaw-multi-3|多租户迁移 #3]] — Replatform 与 AWS 托管服务替换。
- [[entities/openclaw-multi-7-ecs-fargate-graviton|多租户迁移 #7]] — ECS Fargate + Graviton 的轻量平台。

## 对比与选型

- [[entities/deerflow-hermes-openclaw-comparison|DeerFlow vs Hermes vs OpenClaw]] — 三方架构与适用边界。
- [[moc/claude-code-complete-guide|Claude Code 生态]] — 单 Agent 开发工具的代表。
- [[moc/multi-agent-coordination|多 Agent 协作]] — 分工、通信与冲突消解。
- [[moc/security-privacy-landscape|安全与隐私全景]] — 自托管 Agent 的风险面。

## 推荐阅读顺序

1. 先读 [[entities/openclaw-architecture-8-part-summary|源码 8 篇拆解]]，建立消息循环和子 Agent 模型。
2. 再读 [[concepts/openclaw-architecture|架构概念页]]，把术语与组件对齐。
3. 需要落地时读 [[entities/enterprise-openclaw-security-deploy-architecture-guide|企业部署]] 和 [[entities/openclaw-security-and-feature-enhancement-practices|自托管安全实践]]。
4. 要做多租户时按 `openclaw-multi-*` 系列顺序读，不要跳到最后一步。
5. 选型前读 [[entities/deerflow-hermes-openclaw-comparison|三方对比]]，并回到对应 entity 核对证据。

## 边界

- “开源可自托管”与“AWS 托管部署案例”是两件事，引用时不要混为同一主体。
- 多租户能力来自部署架构与产品配置，不等于协议层天然支持。
- 工具总线、共享记忆与渠道接入都扩大攻击面，生产使用必须先定义权限和审计边界。
