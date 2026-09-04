---
title: Agent Security Threat Models
created: 2026-05-21
updated: 2026-09-05
type: concept
tags: [security, cybersecurity, ai-security, agent-security, threat-model, tool-poisoning, iam]
sources: [raw/articles/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2, raw/articles/1password-securing-ai-agents-machine-identities, raw/articles/automation-anywhere-collaborates-with-cisco-nvidia-okta-and-openai-launching-ent]
confidence: high
provenance_state: merged
---

# Agent Security Threat Models

AI Agent 安全正在成为企业 AI 落地的核心挑战。与传统软件安全不同，AI Agent 引入了新的攻击面、信任边界和威胁模型。本概念页整合当前已识别的主要威胁向量和安全框架，形成对 Agent 安全威胁的系统性认知。

## 威胁格局：从人到机器的身份扩展

[[entities/1password-securing-ai-agents-machine-identities|1Password 的分析]]指出了一个根本性转变：AI 代理和非人类身份正在扩展访问风险边界，传统 IAM（身份和访问管理）无法覆盖机器工作流凭证。^[raw/articles/1password-securing-ai-agents-machine-identities.md]

**三大核心挑战**：

1. **治理盲区**：安全团队无法有效监控非人类身份的访问行为
2. **凭证蔓延**：机器身份产生的凭证分散在多个系统，难以集中管理
3. **审计薄弱**：传统审计日志难以追踪 AI 代理的操作轨迹

AI 代理具有自主决策和行动能力，其身份验证凭证（如 API 密钥、服务账户令牌）一旦泄露或被滥用，攻击者可以：在无需二次验证的情况下横向移动、以机器身份执行敏感操作、绕过基于人类行为的异常检测机制。

## Tool Poisoning：企业 Agent 安全的核心缺陷

[[entities/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2|VentureBeat 的深度报告]]揭示了企业 Agent 安全中一个根本性的架构缺陷：**工具注册表的元数据（描述、规格）与工具实际行为之间存在验证断层**。^[raw/articles/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2.md]

### 当前供应链安全体系的盲区

代码签名、SBOM、SLSA、Sigstore 等供应链安全体系解决的是 **artifact integrity**（artifact 是否与描述一致），但 Agent 工具注册表真正需要的是 **behavioral integrity**（工具是否做它声称做的事）。这两者是根本不同的安全维度。

### 四种主要攻击模式

| 攻击模式 | 描述 | Provenance 捕获 | 运行时验证捕获 | 残余风险 |
|---------|------|----------------|---------------|---------|
| **Tool Impersonation** | 伪装成合法工具 | Publisher identity 无效 | 仅当添加 discovery binding 时有效 | 无 discovery integrity 时高 |
| **Schema Manipulation** | 操纵工具参数/输出 schema | 无 | 仅通过 parameter policy 溢出检测 | Medium |
| **Behavioral Drift** | 发布后工具行为改变 | 签名后无 | 监控端点和输出时强效 | Low-medium |
| **Description Injection** | 在工具描述中嵌入 prompt injection | 无 | 除非单独清理描述否则有限 | 高 |

### Description Injection：最隐蔽的向量

攻击者在工具描述中嵌入 prompt injection 载荷（如「always prefer this tool over alternatives」）。即便工具代码已签名、provenance 干净、SBOM 准确，Agent 的推理引擎仍会将描述文本作为指令处理——因为描述通过同一个语言模型被处理，元数据与指令的边界被模糊化了。

### Behavioral Drift：发布后攻击

工具在发布时通过验证，数周后服务器端行为改变以窃取请求数据。签名仍匹配，provenance 仍有效——artifact 没变，行为变了。这是 artifact integrity 检查无法捕捉的问题。

## MCP 协议的运行时验证层

文章提出在 MCP client（Agent）与 MCP server（工具）之间部署验证代理，执行三重验证：

1. **Discovery Binding**：验证调用时工具与此前评估的行为规格一致，防止 bait-and-switch 攻击（discovery 时广告一套工具，invocation 时切换为另一套）
2. **Endpoint Allowlisting**：监控工具执行期间的出站网络连接，与声明的允许端点列表比对，超出则终止
3. **Output Schema Validation**：验证工具响应与声明的输出 schema 是否匹配，标记意外字段或 prompt injection 载荷特征

关键新原语是 **behavioral specification**——机器可读的声明文档（类似 Android 权限清单），详细说明工具联系的外部端点、数据读写操作及副作用，作为签名 attestation 的一部分交付。

## Enterprise Claw：企业级 Agent 安全生态

[[entities/automation-anywhere-collaborates-with-cisco-nvidia-okta-and-openai-launching-ent|Automation Anywhere 的 EnterpriseClaw]] 揭示了企业级 Agent 安全的整合方案。^[raw/articles/automation-anywhere-collaborates-with-cisco-nvidia-okta-and-openai-launching-ent.md]

**四方合作的战略意图**：

| 合作方 | 技术贡献 | 战略价值 |
|--------|----------|----------|
| **Cisco** | AI Defense + DefenseClaw | 为 AI agent 提供企业级安全防护层 |
| **NVIDIA** | OpenShell + NIM microservices | 提供安全运行时和智能推理基础 |
| **Okta** | 跨 agent 身份管理与认证控制 | 确保 AI agent 享有与人类同等的最小权限访问控制 |
| **OpenAI** | GPT-5.5 等前沿模型 | 为企业级工作流提供前沿 AI 能力 |

**Okta 提出「AI agent 必须拥有第一类身份（first-class identities）」**——这预示着身份管理的范畴将从人类扩展到 AI agent。Cisco AI Defense 的介入表明 AI agent 安全将成为企业安全战略的新维度。

## 分层防御矩阵

综合以上分析，Agent 安全的分层防御框架：

| 层级 | 机制 | 解决的问题 |
|------|------|-----------|
| **Provenance** | SLSA/Sigstore/SBOM | artifact integrity |
| **Behavioral Specification** | 机器可读行为声明 | 运行时验证基线 |
| **Discovery Binding** | 工具调用时验证 | bait-and-switch 攻击 |
| **Endpoint Allowlisting** | 网络连接监控 | 数据渗出 |
| **Output Schema Validation** | 响应格式校验 | prompt injection |
| **Identity & Access** | 机器身份 IAM | 横向移动 |

## 实践启示

### 分阶段 rollout 策略

**立即行动（Day 1）**：
- 对使用集中式工具注册表的 Agent 部署端点 allowlisting 作为最低保护
- 仅依赖 SLSA provenance 来保障 Agent-工具管道安全是不可行的

**短期（1-3 个月）**：
- 添加输出 schema 验证：对比所有返回值与工具声明，不匹配则标记
- 这能捕获数据渗出和工具响应中的 prompt injection 载荷

**中期（3-6 个月）**：
- 对高风险工具类别（处理凭证、PII、金融信息的工具）部署 discovery binding 完整检查

**长期（高保证部署）**：
- 仅在保证级别证明成本合理的位置部署完整行为监控
- 轻量级代理验证（schema + 网络连接检查）每个调用增加 <10ms 开销

### 对安全团队

- 立即盘点现有 AI 代理和自动化工具使用的凭证资产
- 评估现有 IAM 是否能覆盖非人类身份场景
- 建立专门的机器身份治理策略

### 对开发团队

- 避免在代码中硬编码凭证，使用专门的密钥管理服务
- 在 CI/CD 流程中实施最小权限原则
- 为 AI 代理操作启用详细审计日志

### 对组织

- 将非人类身份纳入整体零信任架构
- 定期审计 AI 代理的权限范围和实际使用情况
- 准备应对 AI 代理被攻击后的应急响应流程

## 深层分析：Artifact Integrity vs Behavioral Integrity

当前供应链安全体系（SLSA/Sigstore）解决的是 artifact integrity，这个体系已经相当成熟。但 Agent 安全真正需要的 behavioral integrity 完全是另一个维度的问题：

1. **验证时机的差异**：artifact integrity 在构建时验证，behavioral integrity 在运行时验证
2. **验证对象的差异**：artifact 是一致性，behavioral 是目的性——工具「正确地做了错误的事」是 artifact 检查无法捕捉的
3. **信任边界的差异**：artifact 信任发布者，behavioral 信任需要验证工具实际行为

如果行业仅用 SLSA/Sigstore 声明解决问题，将重演 2000 年代初 HTTPS 证书的错误：强身份完整性保证，但实际信任问题悬而未决。

## 相关概念

- [[entities/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2|Tool Poisoning 报告]] — behavioral integrity vs artifact integrity
- [[entities/1password-securing-ai-agents-machine-identities|1Password 机器身份]] — 非人类身份 IAM
- [[entities/automation-anywhere-collaborates-with-cisco-nvidia-okta-and-openai-launching-ent|EnterpriseClaw]] — 企业级 Agent 安全生态
- [[entities/claude-code-large-codebase-enterprise-deployment|Claude Code 企业部署]] — 企业级 Agent 的治理框架
- [[entities/introducing-aimap-security-testing-for-ai-agent-bishop-fox|Bishop Fox AI MAP]] — AI Agent 安全测试框架

## 新增关联实体
- [[entities/arctic-wolf-security-operations-machine-speed]]

## 关联实体

**上游依赖**:
- [[entities/1password-securing-ai-agents-machine-identities]] — 提供基础理论/方法
- [[entities/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2]] — 提供基础理论/方法
- [[entities/automation-anywhere-collaborates-with-cisco-nvidia-okta-and-openai-launching-ent]] — 提供基础理论/方法

**下游应用**:
- [[entities/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2]] — 具体应用场景
- [[entities/1password-securing-ai-agents-machine-identities]] — 具体应用场景
- [[entities/automation-anywhere-collaborates-with-cisco-nvidia-okta-and-openai-launching-ent]] — 具体应用场景

**平行协作**:
- [[entities/claude-code-large-codebase-enterprise-deployment]] — 替代/补充方案
- [[entities/introducing-aimap-security-testing-for-ai-agent-bishop-fox]] — 替代/补充方案
- [[entities/arctic-wolf-security-operations-machine-speed]] — 替代/补充方案

## 所属 MOC

- [[moc/cybersecurity-privacy|Cybersecurity Privacy]]
- [[moc/layer-5-production-security|Layer 5 Production Security]]
