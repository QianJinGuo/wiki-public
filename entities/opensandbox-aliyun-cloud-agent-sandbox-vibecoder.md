---
title: "OpenSandbox：阿里开源的云端 Agent 安全沙箱（凭据 Vault + egress sidecar）"
created: 2026-06-28
updated: 2026-09-18
type: entity
tags: [sandbox, security, credential-vault, egress-sidecar, cloud-agent, aliyun, opensandbox, kubernetes, docker]
sources: [raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder, raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# OpenSandbox：阿里开源的云端 Agent 安全沙箱（凭据 Vault + egress sidecar）

阿里开源的通用 sandbox 平台，解决云端 Agent 安全执行代码的问题。提供 SDK、CLI、MCP、统一生命周期 API，以及 Docker / Kubernetes 两套 runtime。^[raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]

## 定位：执行面，不是完整 Agent 产品

OpenSandbox 负责执行面：创建隔离环境、执行命令、处理文件、控制出站请求。调度、会话、记忆、任务语义等上层能力需要外部系统自己接。^[raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]

与 [[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents]] 的关系：OpenSandbox 更接近 Environment、Sandbox、Vault、Permission policy 里偏运行时的部分，Agent harness 那层不会替你做。^[raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]


## 凭据设计（核心亮点）

**问题**：最省事的做法是把 token 放到环境变量里，但 Agent 进程能读到，命令能打印出来，恶意代码也能转手带走。^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]


**方案：Credential Vault**：^[raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]

- 真实凭据写进 egress sidecar
- 容器里只给 fake key 或空值
- Agent 发 HTTPS 请求，sidecar 透明拦截检查目标 host/method/path
- 匹配绑定规则才注入真实请求头
- 工具链不用大改（Claude Code、Git、curl、npm、模型 SDK 按原来方式访问）

**限制**：
- 依赖透明 MITM 和 CA 信任
- 与 Istio/Envoy sidecar 冲突（两层拦截在同一个网络命名空间内冲突，当前不支持同时工作）
- 不做 response body 的 secret 重写

**Credential Vault 绑定规则**（2026-06-30 补充）：^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]
- 检查维度：scheme、host、port、method、path
- 典型绑定：Anthropic API → `https://api.anthropic.com:443` + `GET/POST` + `/v1/*` + `x-api-key` Header
- 也支持 Basic Auth binding（Git 私有仓库）、自定义 Header/Token 绑定
- 沙箱内设 fake key（如 `ANTHROPIC_API_KEY=fake-k...`）让 CLI 正常启动，真实凭据由 sidecar 在出站时注入
- 沙箱进程从未见过真实密钥

**Credential Vault + 默认拒绝出站策略**：^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]
- 沙箱默认不能访问任意外部地址
- 只允许工具真正需要的服务 Host
- 凭据绑定到更窄的请求范围（如 `/v1/*`、具体仓库路径、具体 Method+Path）
- 收益：密钥不进入沙箱、不残留命令行/文件/日志、凭据只在匹配出站请求上可用、不同服务可绑定不同凭据、平台可审计收敛

## 云端 Agent 接入架构

外部控制面接收任务 → 判断权限和资源配额 → 创建短生命周期 sandbox → sandbox 内运行 Agent CLI → egress sidecar 管出站策略和凭据注入 → 任务结束后日志/结果回传控制面 → sandbox 销毁。^[raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]

## 适用场景与前置条件

**适合**：运行不完全可信的代码，Agent 需要访问 Git、模型 API、包仓库，真实凭据不能直接进容器。^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]


**前置条件**：先有调度器、队列、会话管理、权限策略和审计链路，再接 OpenSandbox 作为 runtime。网络环境（MITM、CA 信任、K8s sidecar 兼容性）必须按真实生产网络验证。^[raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]

## 深度分析

### 凭据不进入沙箱：拆开「能做什么」与「能拿到什么」

把 API Key、Git Token 塞进环境变量或配置文件，等于把凭据的可读范围扩大到整个沙箱：一旦出现 Prompt Injection、恶意依赖或被劫持的第三方 CLI，凭据就可能被命令回显、写进日志、dump 内存或随出站请求外带。Credential Vault 把真值留在沙箱外，由 egress sidecar 在出站链路上注入，容器内只留 fake key——沙箱进程从未持有真值，上述泄露通道同时失效。^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]

更一般的原则是：执行面应把「代码能做什么」与「代码能拿到什么凭据」分开控制。工具链照旧运行，只是提交的是被代理过的凭据；这为 [[concepts/agent-sandbox|Agent 沙箱]] 惯常谈的进程/文件系统/网络/资源四层隔离，补上了凭据隔离。^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]

### 绑定粒度：五维匹配，命中才注入

绑定规则同时检查 scheme、host、port、method、path 五个维度，再决定是否注入认证 Header。以 Claude Code 调 Anthropic API 为例：`https` / `api.anthropic.com` / `443` / `GET+POST` / `/v1/*` / `x-api-key`，只有落在 `/v1/*` 的请求才被补上真 key，环境变量里只是让 CLI 能启动的假值。私有 Git 同理用 Basic Auth binding（无密钥 URL），内部 API 则把 Token 绑到明确的 Header、Host、Path、Method。凭据由此从「进入沙箱后任意可读可复制的字符串」变成「一条受控的出站授权规则」。^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]

### 默认拒绝出站：把凭据收窄到 method + path

Credential Vault 应与默认拒绝的出站策略配对：沙箱默认不能访问任意外部地址，只放行工具真正需要的 Host，再把凭据绑定收窄到 `/v1/*`、某个私有仓库路径或具体 Method+Path。被限制的不只是「能连哪里」，还有「凭据能用在哪些请求上」——恶意代码即便跑起来，能复用的也只是窄口子，而不是可转发到任意地址的万能 token。^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]

### 架构分层：控制面与执行面各司其职

接入模式是：外部控制面接任务/PR/Issue/队列消息，判断权限与配额，为每个任务创建短生命周期 sandbox；sandbox 内跑 Agent CLI，execd 管命令、文件、会话与指标，egress sidecar 管出站策略与凭据注入；任务结束日志与结果回传控制面，sandbox 随即销毁。它因此不定义任务生命周期，只把最易失控的执行环境、网络、凭据三件事收紧，调度与会话语义留在控制面。^[raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]

### 代价与冲突：透明 MITM 不是免费的

能力建立在透明出站拦截与 MITM 之上：必须维护 CA 信任链、确保工具链接受被代理证书，并按真实生产网络验证而非只看 demo。更硬的约束是网络命名空间——pod 内若同时注入 Istio/Envoy 透明 sidecar，两层拦截会冲突，当前不支持与 Credential Vault 同时工作。它也不做 response body 的 secret 重写，不能当万能脱敏器；接入顺序建议是先补齐调度器、队列、会话管理、权限策略与审计链路，再接 runtime。^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md]

### 生态对比：执行面 vs 完整 Agent harness

对照 [[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents]] 的 Environment/Sandbox/Vault/Permission policy 分层，OpenSandbox 覆盖的是偏运行时的部分，harness 那层不替你做；对照 [[entities/langchain-harrison-chase-sandbox-architecture|LangChain sandbox 架构]] 与 [[entities/microsoft-mxc-execution-containers-agent-sandbox-origin|Microsoft mxc 执行容器]]，差异同样在「只交付执行面」还是「连带交付任务语义」。放进 [[concepts/agent-security-architecture|Agent 安全架构]] 看，凭据治理、出站边界、生命周期治理是三条独立轴线，该项目落地前两条，而 [[entities/agent-data-governance-crewai-credential-patterns|CrewAI 凭据治理模式]] 从数据治理侧切入同一问题。^[raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]

## 实践启示

以下几条是把 Credential Vault 的设计意图落成工程动作时的判断依据，边界大多直接来自原文说明及其接入顺序建议。^[raw/articles/opensandbox-credential-vault-vibecoder-2026-06-30.md, raw/articles/opensandbox-aliyun-cloud-agent-sandbox-vibecoder.md]

1. **先分清责任边界**：项目只交付执行面，调度器、队列、会话管理、权限策略与审计链路必须先在控制面建好，再接 runtime。
2. **用假值让工具链原样启动**：沙箱内设 `ANTHROPIC_API_KEY=fake-...`，真值只由宿主侧写入 Vault，由 sidecar 在出站时补上。
3. **绑定写到最窄**：同时约束 scheme/host/port/method/path，能只给 GET 就别给 GET+POST，能只绑一个仓库就别绑整个域名。
4. **默认拒绝 + 白名单**：只放行工具真正需要的 Host，让「网络可达范围」与「凭据可用范围」同时收敛。
5. **按真实生产网络验证**：透明 MITM、CA 信任链、K8s sidecar 兼容性都是必测项，尤其要确认 pod 内是否还注入 Istio/Envoy。
6. **别把它当脱敏器**：它不做 response body 的 secret 重写，输出侧敏感值需另做处理；凭据隔离也不替代进程与资源隔离。

## 相关实体

- [[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents]] — 类似的 sandbox 架构
- [[entities/langchain-harrison-chase-sandbox-architecture|LangChain Sandbox Architecture]] — 另一种 sandbox 设计
- [[entities/microsoft-mxc-execution-containers-agent-sandbox-origin|Microsoft mxc Containers]] — Microsoft 的 sandbox 方案
