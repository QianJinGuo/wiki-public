---
title: Agent Security 全生命周期安全体系（方寸跃迁）
created: 2026-05-07
updated: 2026-08-29
type: concept
tags: [agent-security-full-lifecycle-system]
related:
  - security
  - agent
  - architecture
  - tsinghua
sources:
  - raw/articles/tsinghua-agent-security-fangcun
confidence: high
---
# Agent Security: Full Lifecycle System (方寸跃迁)
> Multi-layer security architecture for Agent production environments, proposed by Tsinghua AI School × the Institute for Interdisciplinary Information (quared). Covers: pre-deployment (Skill Ward) × in-flight (Guard × Observer) × post-incident (audit).

 and  represent the two most prevalent Agent security threat vectors in production — aligning directly with the Skill Ward and Guard layers described below.  provides complementary external threat intelligence.
---
## Architecture Overview
Three products, three layers, covering the complete Agent lifecycle:
```
┌─────────────────────────────────────────────────────┐
│           Agent Security Full Lifecycle             │
├─────────────┬──────────────────┬────────────────────┤
│   事前       │      事中         │       事后          │
│ Skill Ward  │  Fangcun Guard   │  Fangcun Observer  │
│ (Pre-dep)   │  (In-flight)      │  (Post-incident)   │
├─────────────┼──────────────────┼────────────────────┤
│ 3-Phase     │  8ms auditing    │  OS-level behavior │
│ scanning    │  10 risk types    │  Real-time block   │
│ Static+LLM  │  Independent      │  Full-chain trace  │
│ +Docker     │  thresholds       │  Local audit       │
│ honeypot    │                  │  Self-evolving     │
└─────────────┴──────────────────┴────────────────────┘
```

The  provides a reference implementation for how these security layers integrate with the broader Agent runtime framework.
---
## Fangcun Observer (事中行为感知)
**Philosophy**: Don't ask what the Agent intends to do — watch what it actually does.
### Technical Approach
- **OS-level behavior sensing** — no framework plugins, no SDK interfaces, no model-vendor integration
- Zero modification to business code; Agents are unaware of being observed
- Captures: system commands executed, files read/written, network requests made, privilege escalation attempts, high-risk behavior sequences
### Five Core Capabilities
1. **Complete Harness decoupling**: Works regardless of framework (OpenClaw / Hermes / etc.); no re-integration needed on every tech stack migration
2. **Unaware runtime observation**: Agent doesn't know it's being monitored; negligible compute overhead
3. **Real-time intervention**: Dangerous command execution, sensitive file ops, anomalous network access, privilege escalation — all assessed in real-time before behavior lands; strategies trigger: notify / pause / block
4. **Full-chain traceability**: Runtime behavior + Agent decision actions + model context → complete behavioral graph; catches "indirect attackers" who never act directly but manipulate other Agents
5. **Local audit + self-evolving defense**: All data stays on-premise; policy model iterates on real runtime data; security capability grows with business scale

The Observer's behavior-based approach contrasts with traditional declaration-based security, similar to how  shift from static rules to behavioral anomaly detection.
---
## Fangcun Guard (事中输入输出护栏)
**Challenge**: A single Agent conversation passes through 2-4 security checks (user input → tool call params → model output → tool return). Each must not degrade UX.
### Benchmark Results (6 public benchmarks, 7 open-source models)
| Metric | Fangcun Guard | Open-source range |
|--------|--------------|-------------------|
| Detection accuracy | **91.1** | 70–88 |
| p99 latency | **8ms** | 130ms+ (8B); ~50ms (0.6B, but lower F1) |
### Key Capabilities
1. **Accurate across all categories**: General harmful content + jailbreak attacks + deeply disguised gray-area content
2. **Millisecond-level response**: 4 checks total 30ms — imperceptible to users and business
3. **Chinese-language专项**: 10 independent risk categories with Chinese-specific synthetic data and alignment training; stable recall for oralized jailbreaks, long-tail edge cases
4. **10 risk types independently tunable**: Each risk category exposes independent threshold config; finance / healthcare / education / gaming each configure independently via web console or API
5. **One-click主流生态 integration**: Major Agent frameworks supported out-of-box; zero business code changes
---
## Skill Ward (事前检测)
**Threat model**: Third-party Skills (Claude Skills / OpenAI Apps / Claw Hub) are Agent's "App Store". Malicious Skills' real weapons only activate at runtime — not visible in static code: configs that fetch remote payloads on execution, "debug logs" that send outbound requests post-trigger, legitimate packages that activate backdoors under specific parameters.

 and  exemplify the exact threat vectors Skill Ward is designed to detect before deployment.
### Three-Phase Detection
| Phase | Method | What it catches |
|-------|--------|----------------|
| 1 | Static analysis | Malicious signatures, dangerous calls, suspicious dependencies |
| 2 | LLM judgment | Skill true intent, disguised copy, obfuscated logic, social engineering |
| 3 | Docker honeypot sandbox | **Runtime behavior** — commands executed, paths accessed, external addresses contacted, persistence attempts, lateral movement |
### Critical Finding
> 5,000 real Skills tested: static-only scanning misses **~1/3 of runtime threats** — all caught by the honeypot sandbox phase.
---
## Key Insights
1. **Behavior vs. Declaration**: "Past solutions see what the Agent *says*. Observer sees what the Agent *does*. Declarations can be faked; behavior never lies."
2. **Framework independence is survival**: As Agent ecosystems evolve rapidly (OpenClaw → Hermes → new entrants), OS-layer observation is the only approach that doesn't require re-integration with every stack migration
3. **Security as infrastructure, not tax**: 8ms Guard makes security auditing a default-on, everywhere-added infrastructure component — not a performance penalty
4. **Honeypot is irreplaceable**: ~33% of malicious Skills only reveal their true nature at runtime; static scanning is structurally blind to these
---
## 安全体系的评估维度与选型框架

评估一个 Agent 安全体系是否适合生产环境，需要从以下六个维度进行系统性考量，方寸三件套和外部参考体系可以分别对应到这些维度上：

| 评估维度 | 关键问题 | 方寸体系表现 | 其他体系参考 |
|---------|---------|------------|------------|
| **威胁覆盖完整性** | 覆盖了多少种攻击向量？是否包含事前/事中/事后全链路？ | Skill Ward × Guard × Observer 覆盖完整链路，10 个独立风险类型 | 单一厂商通常只做输入过滤或行为监控 |
| **检测方法有效性** | 静态分析是否能替代运行时监控？误报率多少？ | 静态+LLM+Docker honeypot 三阶段，honeypot 捕获~33% 静态盲区威胁 | 纯静态扫描误报漏报双高 |
| **性能损耗** | 安全检查是否引入可感知的延迟？ | Guard p99 延迟仅 8ms，4 次检查总耗时 30ms | 输入过滤方案 p99 通常 >130ms |
| **框架耦合度** | 安全能力是否与特定 Agent 框架绑定？ | Observer OS 级监控，完全框架无关 | SDK 集成方案换框架需重接 |
| **可演进性** | 安全规则是否能随威胁演变自动更新？ | Observer 本地审计数据驱动规则迭代 | 规则库依赖厂商更新 |
| **合规与数据主权** | 监控数据是否出境？是否满足数据本地化要求？ | 全量数据本地保留，无境外传输风险 | 云端监控方案存在数据主权风险 |

 提供了威胁情报层面的外部视角——它解决的问题是"当前 Agent 生态中存在哪些已知威胁"，而方寸体系解决的问题是"这些威胁在你的系统中是否存在"。两者是互补关系：Threat Readiness Framework 提供威胁清单，方寸体系提供威胁检测能力。

对于选型决策：若企业已部署多种 Agent 框架（OpenClaw / Hermes / 自研），Observer 的 OS 级监控是唯一不需要重新集成的方案；若企业追求极低延迟（8ms 级别），Guard 是目前公开方案中性能最优的；若企业需要事前检测第三方 Skills 的恶意行为，Skill Ward 的 Docker honeypot 是目前唯一能捕获~33% 运行时威胁的手段。

---

## Agent 安全体系的横向对比

方寸体系的三个产品并非在真空中竞争，它们与现有 Agent 安全生态中的其他方案形成了清晰的定位差异。理解这些差异有助于在具体场景中选择正确的工具组合。

**Skill Ward vs. 传统代码签名/包管理器验证**：传统代码签名验证的是"发布者是谁"，但无法检测"发布者本身在运行时做了恶意行为"。Skill Ward 的三阶段检测（静态+LLM+Docker honeypot）专门针对这类"可信发布者的恶意运行时行为"，这是传统安全工具的结构性盲区。^[raw/articles/tsinghua-agent-security-fangcun.md]

**Fangcun Guard vs. 输入过滤 API**：大多数输入过滤方案（如 OpenAI Moderation API、Azure Content Safety）设计目标是"检测有害内容"，而非"为 Agent 场景定制"。Guard 的 10 个独立可调风险类型、30ms 完成 4 次检查、中文专项优化，都是为 Agent 场景重新设计的工程指标，而非通用内容检测的适配。

**Fangcun Observer vs. 框架级 SDK 监控**：框架级 SDK 监控（如 OpenTelemetry + Agent instrumentation）需要业务代码接入，一旦换框架，所有监控埋点需要重做。Observer 的 OS 级行为监控完全不看框架脸色，它监控的是进程的实际系统调用——无论 Agent 用什么框架实现，只要它执行了文件操作、网络请求或命令执行，Observer 都能捕捉。这个设计选择使 Observer 成为框架无关的长期安全基础设施。

 的托管 Harness 架构提供了另一种视角：AgentCore 默认集成了安全检查作为平台能力的一部分（工具权限控制、Skills 沙箱），但其安全设计是"默认最小权限"而非"主动威胁检测"。方寸体系和 AgentCore 的关系是：AgentCore 管住了"Agent 能做什么"（权限边界），方寸体系管住了"Agent 实际做了什么"（行为观测）——两者是正交的，都需要。

---

## 关联
- [[concepts/managed-agents-architecture]] — Scaling managed Agents in production
- [[concepts/harness-engineering-framework]] — Agent runtime harness frameworks
- [[concepts/claude-code-source-leak-lifecycle]] — Claude Code source analysis and security mechanisms
-  — Malicious Skills threat intelligence
-  — Enterprise Agent security flaw analysis
-  — External AI threat readiness framework
-  — AgentCore harness architecture reference
- [[entities/cheriot-ibex-memory-safety-hardware-enforcement|CHERIoT-Ibex]] — 硬件级内存安全 enforcement，为 Agent 提供可证明的安全边界
- [[entities/mythos-finds-a-curl-vulnerability|Mythos 发现 curl 漏洞]] — AI 代码安全分析工具的现实检验，Mythos 扫描 curl 代码库的真实案例
- [[entities/tantosec-com-blog-2026-04-route-to-root-in-4g-industrial-router|TANTO Sec: 4G 工业路由器 root 之路]] — CVE-2024-42682：未文档化 uid=0 账户，固件 helper utility 凭证提取，物联网设备的典型攻击面

## 关联实体

**上游依赖**:
-  — 提供基础理论/方法
-  — 提供基础理论/方法
-  — 提供基础理论/方法

**下游应用**:
-  — 具体应用场景
-  — 具体应用场景
-  — 具体应用场景

**平行协作**:
-  — 替代/补充方案
-  — 替代/补充方案
-  — 替代/补充方案

## 所属 MOC

- [[moc/cybersecurity-privacy|Cybersecurity Privacy]]
- [[moc/layer-5-production-security|Layer 5 Production Security]]
