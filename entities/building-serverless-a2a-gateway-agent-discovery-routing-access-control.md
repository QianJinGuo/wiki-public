---
title: "构建 Serverless A2A 网关：Agent 发现、路由与访问控制"
created: 2026-07-02
updated: 2026-10-24
type: entity
tags: [agent, a2a, gateway, agent-communication, aws, serverless, architecture]
sources: [raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 构建 Serverless A2A 网关：Agent 发现、路由与访问控制

> 本文基于 AWS Machine Learning Blog 的技术文章整理。原文介绍了如何在 AWS 上构建一个 serverless 的 Agent-to-Agent (A2A) 网关，实现 agent 的注册、发现、路由和访问控制，支持异构环境（AWS、非 AWS 云、混合环境）下的 agent 互联。^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

## 背景与问题

企业在多个团队、供应商和基础设施上部署 AI agent 后，agent 间的通信管理成为越来越大的运维负担。如果没有中心化的通信层，每个新 agent 的集成都需要：^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]


- **点对点连接**：20 个 agent 最多需 190 个点对点连接（20×19/2）^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]
- **独立凭据**：每个 agent 需要独立的认证配置
- **自定义路由逻辑**：每次集成都要写特殊的转发逻辑
- **分散的访问控制**：没有统一的地方来控制哪个 client 可以访问哪个 agent

结果是：新 agent 工作流上线慢、安全风险高、运维开销随 agent 数量平方增长。^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

网关模式通过在 agent 前面放置单一入口点来解决这个问题，无论 agent 运行在 [[entities/agentic-overlays-rest-to-a2a-enterprise|ECS]]、[[entities/agentrun-multi-agent-a2a-alibaba-cloud|Lambda]]、Bedrock AgentCore Runtime、非 AWS 云还是混合环境。^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]


## 三层架构

网关设计分为三个逻辑层：^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

### 管理层（Management Layer）

中心化的 agent 注册中心，支持发现和语义搜索：^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]


- **Agent Registry**：DynamoDB 表存储 agent ID → 后端 URL、认证配置、缓存的 agent card
- **语义搜索**：用 Amazon Titan Text Embeddings 对 agent 描述做 embedding，存入 S3 Vectors，支持自然语言查找
- **Agent Card 缓存**：URL 重写为指向网关，client 不用知道后端 URL

### 控制层（Control Layer）

基于 JWT scope 的细粒度访问控制：^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]


- **Lambda Authorizer**：验证 JWT，查 Permissions 表确定 scope→agent 映射，生成 IAM policy
- **速率限制**：DynamoDB 原子计数器，per-user per-agent，TTL 自动过期。超限返回 429 + Retry-After header
- **权限集中管理**：修改 Permissions 表即可，无需改每个 agent

### 执行层（Execution Layer）

单一域名的请求路由：

- **API Gateway (REST API)**：单一入口，支持 SSE streaming
- **Proxy Lambda**：OAuth 后端认证，从 Secrets Manager 获取凭据，透明转发
- **路径路由**：`/agents/{agentId}` 格式，A2A 协议原生兼容

## A2A 协议端点

网关支持 A2A 协议规范中的两种 binding：^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

**A2A Native 端点**（JSON-RPC）：^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

- `GET /agents/{agentId}/.well-known/agent-card.json` — 获取 agent 能力
- `POST /agents/{agentId}` — 发送消息（buffered response）
- `POST /agents/{agentId}` with `SendStreamingMessage` — SSE 流式响应

**网关管理端点**：
- `GET /agents` — 列出可访问的 agent
- `POST /search` — 语义搜索 agent
- `POST /admin/agents/register` — 注册新 agent
- `POST /admin/agents/{agentId}/sync` — 刷新 agent card 缓存
- `POST /admin/agents/{agentId}/status` — 激活/停用 agent

## 安全考量

### 后端信任模型

网关采用"认证后信任"模型。agent 注册并验证 OAuth 凭据后，网关直接将响应代理给 client，不做内容审查。后端 agent 负责自身的 prompt injection 防护和输入验证。生产环境中需要实现 agent 注册审批工作流。^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

### Amazon Bedrock AgentCore 认证细节

配置 `customJWTAuthorizer` 时使用 `allowedClients`（验证 `client_id` claim）而非 `allowedAudience`（验证 `aud` claim）。Cognito M2M token 包含 `client_id` 但不包含 `aud`。^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

### 私有部署

支持 VPC 私有部署模式，Lambda 在私有子网中运行，API Gateway 使用私有端点。通过 AWS Direct Connect 或 Interconnect (preview) 可达，使网关可以管理跨环境的 agent。^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

## 部署

使用 Terraform 部署，代码在 `aws-samples/sample-a2a-gateway` GitHub 仓库。包含示例 agent（Weather Agent + Calculator Agent）用于测试。^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

```bash
git clone https://github.com/aws-samples/sample-a2a-gateway
cd sample-a2a-gateway
cp terraform/terraform.tfvars.example terraform/terraform.tfvars
./scripts/build_lambda_package.sh
cd terraform && terraform apply
```

## 深度分析

### 网关把隐式契约显式化

单团队点对点能用，因为身份、授权、寻址由共享代码库、流水线与凭据库隐式保证；跨组织后这三件事必须写成契约。190 条链路只是表面成本，真成本是每链路一对凭据，要单独签发、轮转、吊销，且没有任何一处能回答"X 团队能否调用 billing agent"。网关把 O(N²) 凭据折成 O(N) 次注册加一张权限表，把"谁能调谁"收敛成唯一可审计的产物，并因组织边界即网络边界而成为唯一需暴露的入口。代价是耦合搬到中央：权限表的评审、审计与回滚能力决定架构能活多久。

### 三层沿"变更频率"切，不沿技术切

三层依据是变更频率：加 agent 周级、改策略季度级但须审计、换地址每次部署；故改策略不碰路由、加 agent 不碰 authorizer，网关只需知道 URL、认证配置和 card，从而 runtime 无关。失效模式更值得记：

- **注册表腐化**：card 注册时缓存，后端能力演进后不动，client 按过期能力选 agent；`/sync` 是人工补救——缓存失效是流程不是机制。
- **语义搜索优化召回而非授权**：embedding 匹配散文而非能力契约，"处理退款"与"处理账务争议"向量很近、权限无关；搜索只产候选，权限检查才是闸门。
- **两个易被忽略的坑**：IAM policy 被 API Gateway 缓存（REST 默认 300 秒），"立即吊销"是假的；限流用 DynamoDB 原子计数（user×agent×分钟），条件写冲突正发生在最该限流时，固定窗口还在边界放过双倍突发。
- **共享故障域**：authorizer 一次坏发布即全站 403，而它在点对点架构里根本不存在。

### Agent Card 落在认证与授权的交叉点上

`.well-known/agent-card.json` 按惯例匿名可读；网关把 card 内 URL 改写成指向自己，card 便从描述文档变成携带授权的产物——`GET /agents` 只返回你有权访问者。于是有两份真相：上游全量 card 与网关投影的授权子集，一致性靠缓存加人工 `/sync`。

难点在 per-agent OAuth scope，它把分布式的命名与签发问题伪装成配置项。scope 在 IdP 签发时已被 client 注册固定，新增访问权要同时改 IdP 的 allowed scopes、Permissions 表、后端期望的 scope 串——两处在网关之外；`billing:read` 也只在你拥有整个命名空间时全局唯一，引入供应商 agent 后必有语义漂移。

更根本是粒度错配：scope→agent 的扁平多对多没有资源级、更无参数级约束。持有 `billing:read` 者能向 billing agent 发任意消息，包括让它在别人账户上动手的 prompt injection。授权单元比信任单元（某资源上的某任务）粗，而这个缺口网关关不上——它不解析 A2A 载荷。

还有双跳身份：调用方身份止于网关，proxy 用自身凭据重认证后端，后端只看到"网关"而非"谁"；按调用方的配额、计费与审计只能靠带外身份传递（RFC 8693 token exchange）。参考实现没做，这是生产化最该补的一块。^[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control.md]

### Serverless 的账按 agent 数算，不按请求数

冷启动压在最贵路径：一次 A2A 请求至少两次 Lambda（authorizer + proxy），缓存后稳态一次；proxy 是容器镜像加 Web Adapter，冷启动含拉镜像、runtime 与 adapter 启动，数百毫秒到数秒，正落在首 token 上。而 agent 流量 QPS 低、突发强、数量多，最容易让所有容器同时变冷；预置并发能修，成本却随 agent 数线性涨——成本优势恰在最需要网关的规模上消失。SSE 还要求 proxy 全程存活，墙钟计费让流时长直接变成成本，而 API 集成超时是硬上限——那是 API 类型的属性，不是 agent 的。

爆炸半径是架构性的：单域名+单 authorizer+单 proxy，一次权限表写入即能原子地全开或全关。你用 N² 凭据换来一个 kill switch，它就必须按生产配置治理。

成本随扇出放大：每请求 = 1 次 API Gateway + 1 次 proxy + 1 次计数写，1000 rps 即 1000 WCU/s 按需容量，易超过计算本身；编排 agent 每轮 fan-out 20 个下游时，几十毫秒开销能占两位数百分比。

会破的地方：task 是长生命周期对象（task id、resubscribe），无状态网关既不能把 task 钉到某实例，也不能在 Lambda 死后恢复流。故它适合"请求—响应 + 短 SSE"，对长任务加重订阅明显退化——要么外置会话状态，要么让部分调用绕过网关并接受治理缺口。

## 实践启示

1. **先定身份传递，再定网关形态。** 身份止于网关、proxy 用自身凭据访后端，改成 token exchange 或签名身份头是重写而非加字段。
2. **scope 按资源类设计，不按 agent。** agent 级 scope 一授权即全量；至少 `{域}:{动作}:{资源类}`，参数级约束放后端强制。
3. **card 缓存当缓存，不当快照。** 加 version + TTL，CI 里检测契约变更并触发 `/sync`，补一条过期降级测试。
4. **权限表按生产配置治理。** 改前评审、灰度到小范围 scope、写审计日志、留 break-glass；注册审批接进 CI/CD。
5. **量化两个数再定执行层。** proxy 真实冷启动（镜像实测）与 p95 流时长对照 API 超时；不达标就上预置并发。
6. **写明网关不管什么。** 内容审查、prompt injection 防护、后端侧按调用方计费——它不解析 A2A 载荷，不写清就会有人默认它做了。

## 相关实体

- [[entities/agentic-overlays-rest-to-a2a-enterprise|Agentic Overlays：从 REST 到 A2A 的企业转型]] — REST → A2A 过渡的互补方案
- [[entities/agentrun-multi-agent-a2a-alibaba-cloud|Alibaba Cloud AgentRun 多 Agent A2A]] — 不同云上的 A2A 实现
- [[entities/securing-ai-agents-how-aws-and-cisco-ai-defense-scale-mcp-and-a2a|AWS/Cisco A2A 安全方案]] — A2A 安全治理生态
- [[entities/langgraph-a2a-adversarial-agent-team|LangGraph A2A 对抗 Agent 团队]] — A2A 协议的应用示例

→ [[raw/articles/building-serverless-a2a-gateway-agent-discovery-routing-access-control|原文存档]]
