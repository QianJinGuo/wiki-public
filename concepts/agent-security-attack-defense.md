---
title: "Agent 安全攻防"
created: 2026-06-11
updated: 2026-08-01
type: concept
tags: [security, agent, data, llm, prompt, attack-surface, defense-in-depth, threat-model, sandbox-escape, tool-call, supply-chain]
description: "Agent 安全攻防：Prompt 注入、越狱、供应链攻击、防御机制"
---

# Agent 安全攻防

> Agent 系统的安全是**攻防双向工程**——既需要理解攻击面（输入污染 / 工具滥用 / 供应链 / 推理资源盗窃），也需要部署可量化的防御体系（白名单 / 沙箱 / 红队 / 监控）。本概念页汇总 wiki 中安全相关的 6 个核心实体（已剔除 6 个主题漂移的实体），给出"**威胁建模 → 攻击面分类 → 防御层级 → 评估方法**" 四步法。

## 一、威胁建模

[AI Agents Security Survey](entities/ai-agents-security-survey-attack-defense) 系统梳理了 Agent 的独特威胁面，相比传统 Web 安全多出 3 类新攻击：

| 攻击类型 | 描述 | 风险 | 引用 |
|---------|------|------|------|
| **Prompt 注入** | 恶意指令覆盖系统提示 | 任意指令执行 | [[concepts/prompt-injection-defense]] |
| **工具滥用** | Agent 工具被诱导执行危险操作 | 系统破坏 / 数据外泄 | [代码质量 5 防线](entities/ai-coding-agent-quality-defense-five-control-mechanisms-tutu-agi) |
| **推理资源盗窃** | 高利润端点被批量爬取 | 单 prompt $2 vs HTTP $2/million 价差 | [Vercel Token 防御](entities/vercel-inference-theft-ai-endpoint-economics-2026) |
| **沙箱逃逸** | Cursor 远程隧道利用链等 | 跨进程权限提升 | [[concepts/prompt-injection-defense]] |
| **供应链攻击** | Skill / 工具 / 插件含恶意代码 | 大规模后门 | [Agent Skill 写作指南](entities/agent-skill-writing-guide) |

## 二、攻击面分类

### 2.1 推理资源盗窃（AI Endpoint Economics）

[Vercel Token 防御文章](entities/vercel-inference-theft-ai-endpoint-economics-2026) 揭示了一个被低估的攻击面：**单次 LLM 推理成本 $2，而 1M HTTP 请求只 $2**——这中间 6 个数量级的价差形成了完美的" 套利窗口"。

攻击者典型路径：
1. 用 residential proxy 绕过 IP 限流
2. 批量调用 LLM endpoint 转售 / 自用
3. 单次成本压到 0 但产生全量 LLM 推理费用

**防御对比**：

| 防御类型 | 实现 | 攻击者绕过成本 |
|---------|------|----------------|
| IP 限流 | 按 IP 计数 | 低（residential proxy） |
| Per-session 验证 | 按 session 计数 | 中（session 池化） |
| **Per-request 验证** | 每次请求独立验证（BotID / 行为指纹） | 高（需要真实浏览器指纹） |

### 2.2 工具调用滥用

[代码质量 5 防线](entities/ai-coding-agent-quality-defense-five-control-mechanisms-tutu-agi) 给出 5 个 AI 编程 Agent 的代码质量控制机制，本质是"**工具调用的事前 / 事中 / 事后校验**"：
1. **白名单工具集**——只允许注册过的工具
2. **参数 schema 强校验**——工具参数不能是任意字符串
3. **执行前 lint**——代码生成后必须经独立 lint
4. **执行后 test**——必须通过测试集才能合并
5. **行为审计日志**——所有工具调用可追溯

### 2.3 供应链与 Skill 生态

[Agent Skill 写作指南](entities/agent-skill-writing-guide) 强调"**每个 Skill 上线前必须经过对抗性测试**"——这本质是 npm/pip 时代的供应链问题在 LLM 时代的重演。Skill 是一段指令 + 一段代码，**两者都可能被恶意构造**。

## 三、防御层级（Defense in Depth）

[Agent Harness 生产指南](entities/agent-harness-architecture-design-production-guide) 给出工业级 Agent 的安全架构是 **4 层防护**：

```
L1 认证授权：谁可以调用 Agent（API Key / OAuth / Vault）
L2 工具沙箱：Agent 可以调什么（白名单 + 资源配额）
L3 行为审计：Agent 实际做了什么（日志 + 偏离检测）
L4 人工兜底：高风险操作需人类审批
```

[AI Friendly 架构](entities/ai-friendly-architecture-design-taobao) 强调"**面向 LLM 的架构设计**"——把传统 12-factor 应用的安全实践映射到 LLM 系统，强调"**可观测性比隔离更重要**"（LLM 行为本身就难预测，靠隔离堵不如靠监控发现）。

## 四、运营化（AgentOps）

[AgentOps: Operationalize agentic AI at scale with Amazon Bedrock AgentCore](entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr) 给出生产级 AgentOps 的核心要素：

- **指标体系**：成功率 / 平均步数 / 工具调用失败率 / Token 成本 / 用户干预率
- **告警分级**：P0 (完全失败) / P1 (能力退化) / P2 (成本异常) / P3 (体验降级)
- **回滚机制**：每次 Agent 行为版本化，失败可秒级回滚
- **A/B 测试**：新版本 Agent 与旧版本并行运行 24h 对比

## 五、评估方法

5 个安全相关实体中 3 个都强调"**对抗性测试是核心防御手段**"——不靠红队测试的防御体系都是纸老虎。

具体实践：
- **自动化红队**：从历史攻击样本库采样 + LLM 变异生成新攻击
- **离线 benchmark**：维护固定的 attack suite，每次 Agent 升级必须跑全量
- **线上真实流量监控**：用户实际请求中混杂攻击，部署 detection 实时告警
- **季度攻防演练**：组织跨团队对抗，验证防御体系

## 六、相关实体

**核心安全相关（6 个）**：
- [[entities/ai-agents-security-survey-attack-defense|AI Agents Security Survey: Attack and Defense]]
- [[entities/vercel-inference-theft-ai-endpoint-economics-2026|Inference Theft as AI Endpoint Attack Surface — Vercel Token Theft Defense 2026]]
- [[entities/ai-coding-agent-quality-defense-five-control-mechanisms-tutu-agi|构建 AI 编程智能体的质量防线：5 个实用的代码质量控制机制]]
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计与实现：生产级 Agent 系统落地指南]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr|AgentOps: Operationalize agentic AI at scale with Amazon Bedrock AgentCore]]
- [[entities/ai-friendly-architecture-design-taobao|面向 LLM 的架构设计：什么是真正的 AI Friendly 架构？]]

**主题漂移（已剔除，6 个）**：
> ai-agent-engineer-learning-roadmap-backend-2026 / ai-hardware-cambrian-baidu-intelligent-cloud-catalyst-geekpark / ai-native-startup-cyberfund-2026 / ai-techliwen-creaoai-cloud-agent-infrastructure-two-lessons-20260606 / agent-eval-wallezhang-yaml-driven-agent-evaluation / ai-agent-harness-construction-akshay-baoyu — 这些实体与 Agent 安全无直接关联，已从本页"相关实体"列表中剔除（保留在 wiki 其他位置）

## 七、相关概念

- [[concepts/agent-security-architecture|Agent Security Architecture]]（更广义的安全架构）
- [[concepts/prompt-injection-defense|Prompt 注入防御]]（专门深入注入攻击）
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]（编排层的安全）
- [[queries/ai-agent-safety-guardrails-design-patterns|AI Agent Safety Guardrails 设计模式]]（运行时护栏 vs 攻击防御互补）

## 八、实践启示

1. **威胁建模先于防御**——不画攻击面就上 WAF 是浪费
2. **AI Endpoint Economics** 是 2026 年新攻击面，per-request 验证是最低成本防御
3. **工具白名单 > 黑名单**——这是 5 个实体的一致结论
4. **可观测性比隔离更重要**——LLM 行为本身难预测，靠监控发现 > 靠隔离堵
5. **每条防御规则都是版本化、可回滚的**——5 个实体的共同实践
6. **季度红队 + 真实攻击样本库**——对抗性测试是核心

## 所属 MOC

- [[moc/layer-5-production-security|Layer 5 Production Security]]
