---
title: "Higress 新发布 AI Gateway 能力增强 Gateway API  阿里云云原生"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-29-Higress-新发布-AI-Gateway-能力增强-Gateway-API--阿里云云原生]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-29-Higress-新发布-AI-Gateway-能力增强-Gateway-API--阿里云云原生.md|原文存档]]

sha256: ac361ce471506b78fe72c0aec415ba07197ea5da534cd474207624c954e66723 ^[raw/articles/2026-06-29-Higress-新发布-AI-Gateway-能力增强-Gateway-API--阿里云云原生.md]

## 摘要

Higress 发布 v2.2.3 版本（主仓库 48 项更新、Console 8 项更新），并官宣正式完成 CNCF Sandbox 入驻 ^[raw/articles/2026-06-29-Higress-新发布-AI-Gateway-能力增强-Gateway-API--阿里云云原生.md]。AI Gateway 方面的核心增强：AI Proxy 支持 vLLM 透传 Anthropic Messages 与新版 OpenAI endpoints（能原样透传的不做多余转换）、新增 ai-context-limit WASM 插件在网关层提前拦截超出模型上下文限制的请求、ai-security-guard 增加结构化拒绝响应并支持 Embedding API 内容检测、ai-load-balancer 新增一致性哈希 cluster_hash 策略 ^[raw/articles/2026-06-29-Higress-新发布-AI-Gateway-能力增强-Gateway-API--阿里云云原生.md]。Gateway API 侧支持可配置的 GatewayClass 隔离（多套 Higress 在同一集群各自管理资源）并默认分离稳定与实验性资源，推理扩展修复了 InferencePool 路由配置在 HTTPRoute 合并时丢失的问题；Ingress 迁移支持跳过 IngressClass 创建、正确保留 LoadBalancer hostname，尽量不动既有集群资源 ^[raw/articles/2026-06-29-Higress-新发布-AI-Gateway-能力增强-Gateway-API--阿里云云原生.md]。安全方面 jwt-auth 支持 remote JWKS、OIDC verifier 不可用时 fail closed、回滚了跳过 HTTPS 上游证书校验的行为；文章最后梳理了入驻 CNCF 的合规清单（商标移交 Linux Foundation、Apache 2.0、开放治理、DevStats 看板）并给出升级建议与 18 位贡献者名单 ^[raw/articles/2026-06-29-Higress-新发布-AI-Gateway-能力增强-Gateway-API--阿里云云原生.md]。

## 关键要点

- AI Gateway 增强与修复：ai-context-limit 插件在网关层提前判断请求是否超模型上下文（适用长文档问答、RAG、多轮对话、代码分析），省去打到模型服务才失败的浪费；同时修复 Vertex 场景 tool call ID 与 thoughtSignature、Claude API 名称识别改为后缀判断减少换模型出现 400 等协议兼容问题
- Gateway API Inference Extension 的方向意义：AI 推理流量调度要考虑不同模型、副本 GPU 负载、队列长度、缓存命中——网关不再只是入口，会逐步参与推理流量调度
- OIDC 的 fail closed 原则：认证组件异常时受保护路由应明确失败而不是悄悄放行；Key Auth 支持同一服务配置多个凭证
- CNCF Sandbox 入驻清单：签署贡献协议、商标与 Logo 移交 Linux Foundation、Apache 2.0 许可证与许可证扫描、迁入中立 GitHub 组织、启用 DCO、推进 OpenSSF 最佳实践徽章、接入 DevStats/CLOmonitor/LFX Insights
- 升级方式：helm upgrade higress higress.io/higress --version 2.2.3；使用 Gateway API、Ingress 迁移或 AI Gateway 插件的用户建议先 helm template 对比安装结果

## 来源

- 原文：[[raw/articles/2026-06-29-Higress-新发布-AI-Gateway-能力增强-Gateway-API--阿里云云原生.md|Higress 新发布 AI Gateway 能力增强 Gateway API  阿里云云原生]]
