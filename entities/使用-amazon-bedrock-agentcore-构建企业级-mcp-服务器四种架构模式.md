---
title: "使用 Amazon Bedrock AgentCore 构建企业级 MCP 服务器：四种架构模式"
created: 2026-07-29
updated: 2026-09-25
type: entity
tags: [aws, bedrock, agentcore, mcp, architecture, enterprise, serverless]
sources: [raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 使用 Amazon Bedrock AgentCore 构建企业级 MCP 服务器：四种架构模式的实践指南

AWS China Blog 2026-07-29 发布的深度技术文章，系统地介绍了使用 Amazon Bedrock AgentCore 构建企业级 MCP 服务器的四种架构模式及其渐进式迁移路径。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

## 四种架构模式

### Method 1：直接 Lambda 调用（演示和测试专用）
绕过所有中间层，MCP 客户端直接通过 AWS SDK 调用 Lambda 函数。最快上手，但不支持标准 MCP 客户端、多用户场景和公网暴露。仅用于 PoC 和开发调试。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

### Method 2：用户认证 + AgentCore Gateway
利用 Amazon Bedrock AgentCore 的原生 MCP 协议支持。客户端通过 Amazon Cognito 用户池认证，获取 JWT 令牌后调用 AgentCore Gateway。支持用户级认证，完全兼容 MCP 协议，适合企业内部多团队使用。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

### Method 3：OAuth2 + AgentCore Gateway
代表最高级别的企业安全实现。客户端通过 Cognito OAuth2 客户端凭证流获取令牌，使用 Bearer Token 调用 AgentCore Gateway。提供最高安全性和完整 MCP 协议兼容性。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

### Method 4：API Key + API Gateway
通过 Amazon API Gateway + Lambda Proxy 架构向外部客户暴露 MCP 服务。Lambda Proxy 作为 API Gateway 和 AgentCore 之间的协议转换层，将 API Gateway 的标准 HTTP 请求转换为 AgentCore 可以理解的 MCP 协议。适合公网暴露的外部客户接入场景。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

## 核心架构设计

### AgentCore Gateway 模式
通过引入 AgentCore Gateway 实现了从单点连接到中心化网关的架构演进，本质上是在探索 MCP-as-a-Service 的概念。网关层统一管理多个 MCP 服务端、实现协议版本透明升级、提供统一监控和日志、支持动态工具发现和路由。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

### 多身份验证路径
项目区分 API Key、OAuth2 和直接调用三种认证方式，解决了"如何在公网安全地暴露 MCP 服务"的关键痛点。分层的认证设计让不同安全等级的应用都能找到合适的接入方式。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

### 云原生无状态化
利用 AWS Lambda 处理 Tool 逻辑，将 MCP 这种基于长连接的有状态协议适配到无状态 Serverless 环境。实现自动扩缩容、按需付费、零运维成本。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

## 渐进式迁移路径

| 阶段 | 架构 | 适用场景 |
|------|------|---------|
| 第一阶段：快速验证 | Method 1（直接 Lambda） | PoC 验证 MCP 工具可行性 |
| 第二阶段：内部部署 | Method 2（用户认证 + AgentCore） | 企业内部多团队使用 |
| 第三阶段：企业级安全 | Method 3（OAuth2 + AgentCore） | 敏感数据/严格安全要求 |
| 第四阶段：公网暴露 | Method 4（API Gateway + API Key） | 外部客户服务 |

关键优势：整个迁移过程中，MCP 工具本身不需要修改，只需更改客户端配置。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md]

## 深度分析

### 四种模式的本质：一条「信任边界外推」的频谱
把 Method 1–4 并排看，四者并非平行的可选项，而是安全边界逐级外推的连续频谱：Method 1 完全依赖 IAM 凭证，客户端必须持有 AWS SDK 与密钥，信任边界就是 AWS 账户本身；Method 2 把边界推到 Cognito 用户池，业务用户无需 AWS 凭证即可使用；Method 3 引入 OAuth2 client credentials 流，让「应用身份」与「用户身份」解耦，适合机器对机器的大规模调用；Method 4 则把边界推到公网，用 API Key 对外、OAuth2 对内的双层结构完成信任转换。极简、内部、标准、外部四个定位，本质是同一工具平台面向不同信任等级的四张门禁。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md:125-128]

### 复杂度收敛在服务端：Method 4 为何成为客户默认
原文给出一个有分量的观察：客户实践中最常见的选择恰恰是 Method 4。Lambda Proxy 把最难的复杂度全部封装在服务端——OAuth2 握手、client_secret 保管、Token 自动获取都在代理层完成，客户端只需一个 API Key 和标准 HTTP 接口，任何编程语言都能接入。这种「安全敏感的留在服务端、简单易用的暴露给客户端」的不对称设计，与 [[entities/anthropic-12-mcp-production-patterns|Anthropic 12 MCP Production Patterns]] 把鉴权下沉到基础设施层的主张相互印证。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md:325-379]

### MCP-as-a-Service：网关层与「有状态协议的无状态化」
AgentCore Gateway 标志着 MCP 从「单点集成」走向「平台能力」：统一管理多个 MCP 服务端、协议版本透明升级、统一监控日志与动态工具发现路由，正是企业 Agent 平台刚起步探索的方向，网关机制另见 [[entities/amazon-bedrock-agentcore-gateway-mcp-extension|Bedrock AgentCore Gateway MCP Extension]]。另一对张力值得注意：MCP 协议基于长连接、是有状态的（协议本身另见 [[concepts/model-context-protocol-mcp|MCP]]），而方案却把它适配到无状态的 Lambda 上——每次调用独立、自动扩缩容、按需付费、零运维。这种「有状态协议的无状态化」是全文最具复用价值的设计思想。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md:138-176]

### 渐进式迁移：工具不变，边界移动
真正的工程价值在于「同一套 MCP 工具实现贯穿四个阶段」：迁移中工具零修改，只需更改客户端配置。企业可在 PoC 阶段用最小成本验证可行性，再按业务发展逐级升级认证层，不产生重构债。架构选择判据也很清晰：开发测试选 Method 1/2；生产环境在「最高安全 + 完整 MCP 协议兼容」（Method 3）与「多客户端 + 配置简单」（Method 4）之间二选一。原文的性能基线（Lambda 256MB/60 秒超时、Token 过期前 60 秒自动刷新、urllib3 连接池）说明「有状态协议跑在无状态环境」并非免费红利，而是靠 Token 生命周期与连接管理工程换来的。^[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式.md:180-190]

## 实践启示

1. **从 Method 1 起步，但绝不停留在 Method 1**：直接 Lambda 调用仅适用于 PoC 与本地调试，不支持标准 MCP 客户端、无法扩展到多用户；验证完成后立即升级到 Method 2/3/4。
2. **优先将 Method 4 作为生产默认**：Lambda Proxy 把 OAuth2 复杂度封装在服务端，客户端只拿 API Key，天然兼容 Claude Desktop、Kiro、Cline 等标准 MCP 客户端。
3. **把认证当作分层设计而非单一开关**：API Key（简单场景）、OAuth2（企业级细粒度权限）、直接调用（内部系统、最小延迟）各司其职；先明确调用方的信任等级，再选认证方式。
4. **守住「工具零修改」的迁移承诺**：设计 MCP 工具时把业务逻辑与接入层彻底解耦，四阶段升级只改客户端与网关配置，避免技术债。
5. **Serverless 化长连接协议必须处理 Token 生命周期**：缓存 Token 并在过期前自动刷新、用连接池管理 HTTP、合理配置 Lambda 内存与超时，是「有状态协议跑在无状态环境」的必要配套。
6. **公网暴露时让 API Gateway 承担企业级能力**：限流、API Key 管理、监控日志、CORS 都应放在网关层，而不是在 Lambda 里重复造轮子。

## 与相关实体的关联

→ [[raw/articles/使用-amazon-bedrock-agentcore-构建企业级-mcp-服务器四种架构模式|原文存档]]

这一架构设计与以下实体相关：
- [[entities/agentcore-harness|AgentCore Harness]] — AgentCore 基础能力
- [[entities/agentcore-managed-harness|AgentCore Managed Harness]] — 托管式 AgentCore
- [[entities/amazon-bedrock-agentcore-adds-quality-evaluations-and-policy-controls-for-deploying-trusted-ai-agents|Bedrock AgentCore 质量评估与策略控制]] — AgentCore 的评估治理能力
- [[entities/amazon-bedrock-agentcore-gateway-mcp-extension|Bedrock AgentCore Gateway MCP Extension]] — AgentCore Gateway MCP 扩展
- [[entities/anthropic-12-mcp-production-patterns|Anthropic 12 MCP Production Patterns]] — MCP 生产模式参考
- [[entities/agentops-operationalize-agentic-ai-amazon-bedrock|AgentOps + Bedrock]] — Agent 运维
- [[entities/smartsheet-remote-mcp-server-aws-architecture|Smartsheet Remote MCP Server on AWS]] — MCP 服务器架构参考
