---
title: "AgentCore vs Claude Code：托管 vs 自建 Harness 决策"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, agentcore, claude-code, harness, managed, self-built]
sources: [entities/agentcore-harness, entities/claude-code-harness-deep-understanding, entities/agent-harness-architecture]
---

## 对照表

| 维度 | AgentCore (托管) | Claude Code (自建) |
|------|---|---|
| 所有权 | AWS 托管 | 本地 CLI，用户完全控制 |
| context 管理 | AWS 自动管理硬件/scale/context | 用户通过 working set 规则管理 |
| 定制深度 | 受限于 AWS API 接口 | 无限（可改任意工具/prompt/loop） |
| 运维成本 | 几乎为零（托管） | 自担（实际就是跑 CLI） |
| 安全合规 | AWS IAM / KMS / SOC2 即得 | 依赖本地 git/repo 安全实践 |
| 扩展性 | AWS 弹性扩展 | 本地资源上限（单机） |
| 适合团队 | 企业/团队/有合规需求 | 个人/小团队/希望深度定制 |
| 生态集成 | AWS Bedrock / SageMaker 全家桶 | Hermes/cronjob/MCP skills |

## 判断

决策矩阵：< 5 工程师 → 选托管（AgentCore）省运维；> 20 工程师 + 有定制需求 → 自建（Claude Code/Hermes）；混合策略 → 核心 loop 自建 + observability/queue/storage 用 AWS。注意：托管的最大隐性成本是「锁定 + 调试难」，自建的最大隐性成本是「人月成本被低估」。

## 对比方来源

- [[concepts/100-line-vs-managed-harness-tradeoff|100 行 vs 托管 harness 权衡]]
- [[concepts/harness-as-product-surface|harness 作为产品界面]]
- [[concepts/managed-agents-architecture|managed agents 架构]]
- [[entities/agentcore-harness|AgentCore harness 详解]]

## 进一步阅读

- [[entities/agentcore-harness]]
- [[entities/claude-code-harness-deep-understanding]]
- [[entities/agent-harness-architecture]]
