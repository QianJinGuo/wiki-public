---

title: "Reference your own AWS Secrets Manager secrets in Amazon Bedrock AgentCore Identity"
created: 2026-06-10
updated: 2026-09-05
tags: [agent, aws, code, data, open-source, prompt, rl, security, tool-use]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/bedrock-agentcore-secrets-manager-identity
---

# Reference your own AWS Secrets Manager secrets in Amazon Bedrock AgentCore Identity

→ [[raw/articles/bedrock-agentcore-secrets-manager-identity|原文存档]] ^[raw/articles/bedrock-agentcore-secrets-manager-identity.md]

## 深度分析

Reference your own AWS Secrets Manager secrets in Amazon Bedrock AgentCore Identity ^[raw/articles/bedrock-agentcore-secrets-manager-identity.md]
### 核心观点
1. sha256: 59ab9fcf9525ccb30d11b2162928a4cc0e1955d3db620fb2db6f9f07bc28ed70
# Reference your own AWS Secrets Manager secrets in Amazon Bedrock AgentCore Identity
AI agents are only as powerful as the tools they can access. ^[raw/articles/bedrock-agentcore-secrets-manager-identity.md]
2. Whether retrieving customer data from a CRM, posting updates to Slack, or querying a GitHub repository, agents need to call external APIs, and that means securely passing credentials at runtime.
3. Getting that right, without hardcoding secrets in code or exposing them in agent prompts, is one of the defining challenges of building production-ready agentic systems.
4. Amazon Bedrock AgentCore Identity meets this challenge through credential providers and a token vault that automatically create and manage a secret in AWS Secrets Manager in your account for each Outbound credential provider resource.
5. This secret contains either the API key or client secret along with the other metadata for the external identity provider.

### 关联实体

- [[entities/两万字详解claude-code源码核心机制]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/构建基于多智能体架构的深度思考交易系统-v2]]

## 相关实体

- [[moc/prompt-engineering-guide|MOC]]
