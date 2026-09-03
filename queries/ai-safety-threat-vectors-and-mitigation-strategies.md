---
title: "AI Safety 的主要威胁向量与防护策略是什么？"
created: 2026-05-21
updated: 2026-05-21
type: query
tags: [ai-security, threat-vector, mitigation, agent-security, adversarial]
sources:
  - raw/articles/tsinghua-agent-security-fangcun
related:
  - concepts/agent-security-full-lifecycle-system
  - entities/skill-issues-compromising-claude-code-with-malicious-skills-agents-part-1
  - entities/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security
  - entities/ai_threat_readiness_framework
confidence: high
---

# AI Safety 的主要威胁向量与防护策略是什么？

> 本页综合 [[concepts/agent-security-full-lifecycle-system|方寸 Agent 安全全生命周期体系]]、恶意 Skills 威胁情报、工具投毒案例，结构化回答 AI Safety 的核心威胁向量与对应防护策略。

---

## 一、威胁向量全景图

AI Agent 安全威胁可归类为三大向量，贯穿部署前（Pre-deploy）、运行中（In-flight）、事后（Post-incident）全生命周期：

| 威胁向量 | 攻击面 | 典型案例 | 占比估算 |
|---------|--------|---------|---------|
| **第三方 Skills 恶意化** | 部署前供应链 |  | ~40% |
| **工具/工具链投毒** | 运行期调用链 |  | ~35% |
| **Agent 越权行为** | 运行期决策层 | 权限 escalation、未授权文件访问 | ~25% |

^[raw/articles/tsinghua-agent-security-fangcun.md]

---

## 二、核心威胁向量详解

### 2.1 第三方 Skills 恶意化（Skill Ward 层）

**攻击原理**：第三方 Skills（如 Claude Skills / OpenAI Apps / Claw Hub）是 Agent 的"App Store"，但恶意 Skills 的真实武器只在运行时激活——静态代码审查完全不可见：

- 配置在执行时才抓取远程载荷
- "Debug 日志"在触发后发送出站请求
- 合法包在特定参数下激活后门

**关键数据**：对 5,000+ 真实 Skills 扫描发现，纯静态分析**漏掉约 1/3 的运行时威胁**，必须依赖 Docker Honeypot 沙箱才能捕获。^[raw/articles/tsinghua-agent-security-fangcun.md]

### 2.2 工具/工具链投毒（Guard 层）

**攻击原理**：Agent 依赖的工具生态（搜索 API、代码执行环境、文件读写工具）本身可能被污染或恶意构造。攻击者通过污染工具链的一个环节，使 Agent 在执行正常任务时产生副作用。

详见 。

### 2.3 Agent 越权行为（Observer 层）

**攻击原理**：Agent 在复杂任务执行中可能产生非预期的权限升级行为——读取敏感文件、向未授权地址发送数据、执行危险系统命令。传统基于声明（"我不会做 X"）的防护无法捕捉这类行为。

---

## 三、防护策略框架：三层九项

基于 [[concepts/agent-security-full-lifecycle-system|方寸三件套]] 的实践验证，防护策略分为事前、事中、事后三层：

### 3.1 事前（Pre-deploy）：Skill Ward 三阶段检测

| 阶段 | 方法 | 能力边界 |
|------|------|---------|
| **Phase 1 静态分析** | 恶意签名、危险调用、异常依赖检测 | 快速过滤已知威胁，漏报率高 |
| **Phase 2 LLM 判断** | Skill 真实意图、混淆复制、社会工程识别 | 捕捉语义隐藏的恶意逻辑 |
| **Phase 3 Docker Honeypot** | 运行时行为——命令执行、路径访问、外网连接、持久化尝试 | **唯一能捕获 ~33% 静态盲区威胁的手段** |

**实践要点**：三个阶段缺一不可。Phase 1 快而不准，Phase 2 语义理解但可被欺骗，Phase 3 行为观测是最后防线。

### 3.2 事中（In-flight）：Fangcun Guard 输入输出护栏

**设计挑战**：单次 Agent 对话经过 2-4 个安全检查点（用户输入 → 工具调用参数 → 模型输出 → 工具返回），每次检查不能影响用户体验。

**核心指标**：

| 指标 | Fangcun Guard | 行业常见水平 |
|------|--------------|-------------|
| 检测准确率 | **91.1%** | 70-88% |
| p99 延迟 | **8ms** | 130ms+（8B 模型）|
| 4 次检查总耗时 | **30ms** | >130ms |

**关键能力**：

1. **10 类风险独立可调**：金融/医疗/教育/游戏等行业可独立配置阈值
2. **中文场景专项**：针对口语化越狱、长尾边缘 case 的中文合成数据和对齐训练
3. **零代码接入**：主流 Agent 框架开箱即用，无需业务代码改造

### 3.3 事后（Post-incident）：Fangcun Observer 行为感知

**设计哲学**：不要问 Agent "你想要干什么"——看它**实际做了什么**。

**技术路线**：OS 级行为感知——无需框架插件、无需 SDK 接口、无需模型厂商集成，Agent 不知道自己正在被监控。

**五项核心能力**：

1. **框架无关**：OS 级别监控，无论 OpenClaw / Hermes / 自研框架都能覆盖
2. **无感知运行**：Agent 不知道自己被监控，计算开销可忽略
3. **实时干预**：危险命令、敏感文件操作、异常网络访问、权限升级——全部实时评估，策略触发：通知 / 暂停 / 阻断
4. **全链路追溯**：运行时行为 + Agent 决策 + 模型上下文 → 完整行为图谱
5. **本地审计 + 自进化**：数据不出境，策略模型基于真实运行时数据迭代，安全性随业务规模增长

---

## 四、选型评估框架

评估一个 Agent 安全体系是否适合生产环境，六个维度：

| 评估维度 | 关键问题 | 核心指标 |
|---------|---------|---------|
| **威胁覆盖完整性** | 覆盖多少攻击向量？是否全链路？ | Skill Ward × Guard × Observer 三层完整 |
| **检测方法有效性** | 静态分析能否替代运行时监控？ | Docker Honeypot 捕获~33%静态盲区 |
| **性能损耗** | 安全检查是否引入可感知的延迟？ | Guard p99 ≤ 8ms |
| **框架耦合度** | 安全能力是否绑定特定框架？ | Observer OS 级，完全框架无关 |
| **可演进性** | 安全规则是否能随威胁演变自动更新？ | 本地审计数据驱动规则迭代 |
| **合规与数据主权** | 监控数据是否出境？ | 全量本地保留 |

---

## 五、行动框架

### 企业选型决策树

```
企业已部署多框架 (OpenClaw / Hermes / 自研)?
├── YES → Observer OS 级监控是唯一不需要重集的方案
└── NO ↓
追求极低延迟 (<10ms)?
├── YES → Guard 是目前公开方案中性能最优
└── NO ↓
需要事前检测第三方 Skills?
├── YES → Skill Ward Docker Honeypot 是唯一能捕获~33%运行时威胁的手段
└── NO → 优先 Guard 输入输出护栏
```

### 安全配置checklist

- [ ] **部署阶段**：第三方 Skills 引入前必须经过三阶段检测（静态+LLM+Honeypot）
- [ ] **配置阶段**：Guard 风险阈值按行业独立配置（金融/医疗/教育/游戏）
- [ ] **运行阶段**：Observer 开启实时干预，配置通知/暂停/阻断策略
- [ ] **应急阶段**：建立 Agent 越权行为的 incident response 流程
- [ ] **审计阶段**：定期审计日志本地化留存，验证数据主权合规

---

## 六、关联参考

- [[concepts/agent-security-full-lifecycle-system|方寸 Agent 安全全生命周期体系]] — 三层安全架构完整解析
-  — 真实攻击链复盘
-  — 企业 Agent 安全缺陷根源
-  — 外部威胁情报视角
-  — 对比：AgentCore 默认最小权限，方寸体系管实际行为，两者正交都需要
