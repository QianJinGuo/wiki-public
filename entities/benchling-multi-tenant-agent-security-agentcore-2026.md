---
title: "Benchling 多租户 AI Agent 安全架构：AgentCore Code Interpreter VPC 模式 + DNS Firewall 纵深防御"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: [agent-security, multi-tenant, sandbox, bedrock, agentcore, dns-firewall, data-exfiltration, benchling]
sources: [raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed]
confidence: 0.85
provenance_state: extracted
---

# Benchling 多租户 AI Agent 安全架构

## 摘要

Benchling 在 Amazon Bedrock AgentCore Code Interpreter（VPC 模式）上为数千生命科学租户执行 AI 生成的科学代码：2026 年 4 月初上线以来，扩展到每天 600+ 代码执行会话、每周服务 250+ 独立租户，零安全事件、零跨租户数据泄漏。其威胁模型假设任何 agent 或用户写的代码都可能是恶意的，要求除 S3 外零网络访问。本文的核心洞见是：标准网络控制（封 HTTP、限出站端口、限出站连接）之外，**DNS 解析常常仍被放行**，成为被遗忘的外泄通道——本文是这一威胁在 multi-tenant agent 代码执行场景的完整工程案例。^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

## 核心要点

- **三层纵深组合**：账号级隔离（独立 Untrusted Code Account）+ Route 53 Resolver DNS Firewall（封死 DNS 通道）+ VPC endpoint policies（按 job 强制数据面访问控制）。
- **拒绝 role sprawl**：数千租户不可能一个租户一个 IAM 角色，隔离由网络层而非身份层承担；凭证用 AWS STS 按 job 动态注入。
- **Sandbox 模式不够**：AgentCore 默认 Sandbox 模式只限制出站到 S3，Benchling 要求客户自主定义哪些域名可解析、哪些端点可达，并能用自己的集成测试持续验证 → 选 VPC 模式。
- **三优先级 DNS 策略**：P10 显式 denylist（威胁情报快速路径 + 日志预警）、P100 最小 allowlist（实践中仅 S3 端点）、P200 catch-all 返回 NODATA。
- **防御独立于 IAM**：VPCE policy 在网络层拒绝未列名 bucket，凭证被盗单独不构成越权路径。
- **持续验证**：DNS 隧道、直连、越权 S3 访问等外泄模拟写进 CI 测试套件，任何削弱安全边界的配置漂移在进生产前被拦下。

^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

## 深度分析

### DNS 是最后一条未被枚举的通道

Benchling 安全评审时逐个评估 Code Interpreter 网络模式对其威胁模型的隔离性质：Sandbox 模式仅约束出站到 S3，而对处理数千受监管生命科学租户敏感数据的应用，「仅依赖应用管理的网络限制」不充分——他们需要端到端自有安全控制，包括封堵 DNS 这类非 HTTP 向量，且不暴露主生产账号。^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

三优先级解析策略遵循 denylist → allowlist → deny all 模式。P10 高优先级封禁已知恶意域名（外泄工具包、C2 基础设施），既提供不依赖 allowlist 缺席的快速路径，也通过 DNS Firewall 日志给出沙箱内代码尝试可疑解析的早期预警。P100 allowlist 刻意最小化——每个被允许的域名都是潜在外泄向量，实践中只列 job 所需的 S3 端点，即使是合法 AWS 服务端点，未经批准也无法解析。P200 catch-all 对一切未列名查询返回 NODATA：典型 DNS 隧道把窃取的数据编码为子域标签（如 `base64payload.example.com`）依赖递归解析送达攻击者权威 NS，catch-all 使递归链在第一跳断裂，VPC 从「受限」变为「密封」。^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

### 身份层不动员：per-job STS 凭证注入替代 per-tenant IAM role

生产账号决定每个 job 能访问什么数据，在 dispatch 时生成限定到特定租户 path prefix 的 session policy 临时凭证注入会话；Code Interpreter Execution Role 本身的 S3 访问也只限主 stack bucket。这样即使沙箱内代码完全失陷，也只能触达它被派去服务的那个租户。Benchling 评估过「给 ACCI role 广泛访问所有租户 bucket」的替代方案并否决——那意味着单个失陷会话就有通往任意租户数据的路径。这是对「一人一角色」不可持续性的正面回答：身份只做单任务最小授权，不做租户间长期隔离。^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

### 网络层纵深：把攻击面收缩到「Code Interpreter + 已限定 VPCE」

Untrusted Code Account 的 VPC 无 internet gateway、无 NAT gateway，代码无法直连互联网。S3 同区访问走 Gateway endpoint、跨区走 Interface endpoint，各挂显式列名 bucket 的 VPCE policy，未列名请求在网络层、到达 S3 服务之前即被拒——即使恶意代码拿到他租户的合法凭证，**网络本身拒绝承载流量**，形成独立于 IAM 的一层防御。再往下，NACL 限 443 与临时回程端口、prefix list routing 确保流量只能到达 VPC endpoint、Code Interpreter 跑在无默认回退规则的专用 security group 里。Product Security 团队在 PoC 中逐层单独验证：DNS 隧道确认返回 NODATA、直连确认无任意出口、API 调用确认 VPCE policy 拒绝越权 bucket。^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

验证不止于上线时：新端点、IAM 变更、基础设施演进都可能让部署时安全的配置悄然退化。把外泄模拟纳入 CI——任何测试解析了不该解析的域名、连通了外部端点、或把数据移出批准 bucket，流水线即失败阻断发布——是把「外泄抵抗」当作可测试属性而非一次性配置。^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

### 买 vs 造：VPC 模式 Code Interpreter 赢过自建沙箱

自建方案意味着自定义容器编排、沙箱生命周期管理、从零造网络隔离原语，还要持续打补丁追威胁演化——大量持续工程投入，且对客户无差异化价值。AgentCore VPC 模式绕过了整条工作流：会话短命无持久状态、AWS 负责沙箱 patching/scaling/hardening、Benchling 把既有 AWS 安全原语（DNS Firewall、VPCE policy、NACL）直接叠加在自己锁死的 VPC 上，基础设施团队得以专注产品安全控制而非沙箱维护。注意 gVisor 容器沙箱是 Benchling 既有 compute 隔离层，与本模式并存但不是该模式组成部分；两个执行环境各有 scoped IAM role，互不能越界。^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

## 实践启示

1. **审计沙箱的 DNS 出口**：如果你在沙箱中运行不可信代码，标准网络控制之外问两个问题——DNS 解析是否仍被放行？是否有持续验证证明它没有？
2. **「默认拒绝 + 显式允许」要落到解析层**：catch-all NODATA 是安全姿态反转的关键——没有它，任何新域名默认可解析；有了它，一切解析都需明确决策。
3. **让安全边界可测试**：把攻击者会用的向量（DNS 隧道、直连、越权 bucket）写成 CI 测试，配置漂移在进生产前暴露。
4. **隔离由网络层而非身份层承担**：数千租户场景下 role sprawl 不可持续；IAM 做单任务最小授权，跨租户隔离靠账号分离 + 网络控制。
5. **买 vs 造的判据**：当需求是「短命会话、按 job 隔离、无持久状态、可叠加自有网络控制」时，托管执行 + 自有 VPC 安全原语的组合通常优于自建沙箱。

^[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed.md]

## 相关实体

- [[concepts/agent-sandbox|Agent Sandbox]]
- [[concepts/agent-security-architecture|Agent 安全架构]]
- [[entities/abnormal-ai-agentcore-code-interpreter-agent-scratch-pad-email-security|AgentCore Code Interpreter 安全实践]]
- [[concepts/channel-enumeration-criterion|通道枚举判据]]
- [[concepts/agent-security-attack-defense|Agent 安全攻防]]
- [[entities/aws-bedrock-agentcore|AWS Bedrock AgentCore]]
- [[entities/amazon-eks-ai-agent-sandbox-2026|EKS 上的 AI Agent 沙箱]]

→ [[raw/articles/how-benchling-secured-multi-tenant-ai-agents-with-amazon-bed|原文存档]]
