---
title: "Govern AI Agent Tool Access: 四阶段治理成熟度框架"
created: 2026-08-22
updated: 2026-09-17
type: entity
tags: [agent, security, governance, mcp, access-control, aws, agentcore]
sources: [raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Govern AI Agent Tool Access: 四阶段治理成熟度框架

## 核心问题：谁有权访问客户数据

文章以一个反复出现的客户问题开场："**哪些 AI Agent 能访问客户数据？谁授予的？如果凭证今天泄露，暴露面是什么？**"如果组织无法在一分钟内回答，就需要治理。该框架源自 MCP 部署在企业系统中暴露的结构性失效模式。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

## 五个结构性失效模式

企业 MCP 部署存在五种结构性问题：**凭证蔓延**（secrets 散落在每个本地 config）、**策略漂移**（N×M 配置静默发散）、**审计缺口**（无法回答"谁在何时调用了什么"）、**成本不透明**（支出无法归因到团队）、**影子 IT**（审查之外部署的集成）。以策略漂移为例：10 个助手连接 5 个内部 API，维护 50 套独立凭证集，每套手工配置，一处策略变更要在 50 处更新。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

## 四阶段成熟度旅程

| Scope | 治理问题 | 关键技术 |
|-------|---------|---------|
| **Connect** | 一个受治理的门让 Agent 触达组织资源 | SSO 认证、集中凭证、CloudTrail 审计 |
| **Control** | 知道谁做了什么、路上清洗敏感数据 | Cedar RBAC/ABAC、PII 脱敏、3LO consent、DCR |
| **Catalog** | 团队自己发布/发现工具（含本地工具） | Registry、Resources MCP、OPA、per-tool 成本归因 |
| **Harden** | 锁死边缘、全量监控、规划失败 | 私有连接、治理仪表板、废弃流程、多区域故障转移 |

每个 Scope 独立交付价值，**只在下个痛点出现时才推进**（"matching controls to actual needs"），避免一次性构建完整网关数月才上线错误的东西。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

## 深度分析

### 四阶段是信任边界的四次重画，不是功能叠加

Connect 把信任锚定在**客户端身份**（JWT authorizer + `allowedClients`），授权粗到"任何已认证客户端可调用任何已注册工具"，实质收益只有两项：backend 凭证不出 AWS、CloudTrail 开始有记录。Control 把锚点移到**用户身份**（token 的 `sub` 是真人），才第一次能回答"谁做了什么、依据哪条策略"。Catalog 把边界推到**组织之外**（on-prem 经 Direct Connect/PrivateLink、SaaS 经 outbound OAuth），并把手工注册换成 manifest-in-git 的 policy-as-code。Harden 取消**公共可达性**本身：CloudFront → 带 shared-secret header 的 ALB → VPC Endpoint → PrivateLink，叠加 Runtime 的 inbound-only enforcement，防止调用方绕开策略、guardrail 与审计。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

每道边界断的东西各不相同：Connect→Control 断在 M2M 的 `sub` 是客户端而非用户，且 `tools/list` 对所有客户端返回同一份目录，无法表达分组可见性；Control→Catalog 断在 intake 仍是工单驱动、target 出不了 AWS；Catalog→Harden 断在网关仍挂在 public DNS，没有 circuit breaker 与 DR 路径。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

### 五种失效模式共享同一个根因

credential sprawl / policy drift / audit gaps / cost opacity / shadow IT 不是五个独立问题，而是"每个 assistant 自带一份 `mcp.json`"这一拓扑的必然产物——10 个助手 × 5 个 API = 50 套手工凭证，一处策略变更要在 50 处同步。所以控制手段要按根因配而非按症状配：凭证蔓延 → 集中凭证池（Secrets Manager/Vault + 轮换，或 KMS 私钥的 Private Key JWT）；策略漂移 → 收敛到单一 policy 求值点（网关内 Cedar + interceptor 内 OPA），策略与 manifest 全进 git；审计缺口 → 每条决策记录带 principal 与 matched policy ID；成本不透明 → per-tool/per-group 标签 + Budgets，并按 principal/tool 限流；影子 IT → `mcp.json` 由 MDM 集中分发，并在 corporate proxy/EDR 阻断到非网关联 MCP endpoint。只有"集中"能同时压缩全部五项，其余都是局部补丁，代价则是单点：网关不可用等于所有依赖它的助手不可用。这也是五种模式在 [[concepts/agent-security-threat-models|Agent 安全威胁模型]] 里常被归为同一类"控制面缺失"的原因。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

### agent→tool 的身份语义：M2M、per-user 与 OBO 双主体

M2M 客户端凭证的 `sub` 是客户端，任何拿到该凭证的进程都能以 agent 名义调用全部可见工具——这是 confused deputy 的结构性温床。per-user 模式（DCR + Authorization Code + PKCE，配 RFC 9728/8414/7591 发现链）把 `sub` 换成真人，同时买到两件事：用户级审计，以及按策略过滤的 `tools/list`（不同组看到不同工具）。target 是 SaaS 时，3LO elicitation（-32042 + `authorization_url`，浏览器 consent 后由 `CompleteResourceTokenAuth` 重试）必然带来跳转；On-Behalf-Of token exchange（RFC 8693/7523）则把入站 token 换成同时携带**用户身份 + agent 身份**的窄作用域 token，无跳转无额外 consent。OBO 的双主体是防 confused deputy 的关键：下游能同时校验"谁在调用"与"哪个 agent 代其调用"；只有单一主体时，下游无法区分用户本人调用与 agent 代表调用，授权放大不可避免。这套语义与 [[entities/aws-bedrock-agentcore-identity-security|AgentCore Identity 安全]] 中"凭证不进客户端、身份在下游可验证"是同一件事的两面。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

### 求值点决定可绕过性

网关级强制（Cedar / OPA / Amazon Bedrock Guardrails）的价值不在规则表达力，而在**单一强制点**：Kiro、Claude Code、Amazon Quick、Glean 共用一条路径，策略改一处全局生效；代价是必须先把流量收敛干净（MDM 分发 + proxy/EDR 阻断旁路），否则强制点形同虚设。SDK 内 guardrail 能看到应用语义上下文，但每 app 一份、N×M 复制且可绕过。合理分层：Cedar 管身份/资源/参数级 ABAC（如把 `DeployCI___invoke` 限制在 `context.input.environment == "staging"`），OPA/Rego 补 Cedar 表达不了的时间窗、payload 内容检查、rate-based 与变更单条件，Bedrock Guardrails 管 PII/内容策略/prompt-attack（原生以 `suppressOutput` + guardrail 条件嵌入 Cedar，超出覆盖面的结构变换仍可退回 interceptor Lambda，参见 [[entities/securing-ai-agents-temporal-policies-agentcore|AgentCore 时序策略授权]]）。最后是 IAM SCP 兜底：`aws:CalledViaAWSMCP` / `aws:ViaAWSMCPService` 只对 AWS 托管 MCP server 生效，自有网关要改为收紧 target execution role——它的意义是"策略评审漏了一条，最坏结果仍被挡住"。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

### 让框架可运行的是审计契约与指标

每条决策记录必须自带 principal、action、resource、decision、matchedPolicy、reason、guardrail 标记与 latency——缺 matchedPolicy 的日志无法回答"哪条规则拒绝了谁"，合规提问就仍然悬空。运营指标：per-policy deny rate、top-denied principals（高拒绝率往往是策略过紧的 friction 信号，而非威胁信号）、guardrail intervention rate、延迟分位与零调用工具数；Scope 4 用 Athena 查 OTel spans / CloudTrail 回答合规级问题。这与 [[entities/agent-observability-5-layer-architecture|Agent 可观测性五层架构]] 的诉求一致：治理不是加一个面板，而是让每次调用都留下可查询的决策证据，再用夜间 deprecation Lambda 把 30 天零调用的工具送进 PR → LOG_ONLY → 90 天移除的退役流程，避免注册表堆积僵尸工具。^[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga.md]

## 实践启示

1. **先立单一受治理入口，再谈策略**（Scope 1，一天可完成）：Cognito + Gateway + 一个只读低风险 target，`mcp.json` 经 MDM 分发。没有收敛的流量，Cedar 与 Guardrails 都只是装饰。
2. **升级触发器用"组织提问"，不用时间表**：每个 scope 的出口都是一组具体问题（PII 怎么防？用户 consent 在哪？能否按组区分工具？能否自助发布？public DNS 合规吗？）。回答不了就别急着进下一阶段。
3. **身份先做 per-user，再做下游委托**：把 M2M 的 `sub=client` 换成 Authorization Code + PKCE 的 `sub=真人`，是审计从"哪个 assistant"变成"哪个人"的唯一路径；下游是 SaaS 时优先 OBO 双主体 token exchange，而非只做 3LO 浏览器 consent。
4. **策略先 LOG_ONLY 再 ENFORCE，把翻转指标当上线门槛**：Guardrails 同样先 detect-only；用 `aws.agentcore.policy.log_only_decision_flipping_policies` 量化"上强制会翻掉多少决策"，而不是凭直觉开强制。
5. **把配置变成代码**：tool manifest 进 git（owner / allowed_groups / risk_tier / environments），CI 做安全扫描 + 自动 `create-gateway-target` + 更新 Cedar；gateway/policy/registry 全 IaC，dev/staging/prod 用独立账户与 IdP client，策略变更走 PR 由安全与平台共同评审。
6. **把网关当生产服务，并准备它宕机**：工具设计成幂等，Route 53 健康检查 + 多区域 active-passive，复制 policy 而非数据，定期做 failover 演练；同时用 per-principal/per-tool 限流 + 标签 + Budgets 把成本归因到团队，零调用工具 30 天告警、90 天移除。

## 与既有治理体系的关系

该框架与 [[entities/agent-data-governance-crewai-credential-patterns|CrewAI 凭证治理]] 和 [[entities/ai-gateways-vs-mcp-gateways-what-security-teams-need-to-know|AI 网关 vs MCP 网关]] 互补，但提供了更完整的**组织级成熟度路径**（Connect→Control→Catalog→Harden）。它也呼应 [[concepts/agent-security-architecture|Agent 安全架构]] 中对"工具访问作为攻击面"的关注。

→ [[raw/articles/govern-ai-agent-tool-access-with-amazon-bedrock-agentcore-ga|原文存档]]
