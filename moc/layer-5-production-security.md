---
title: 第 5 层全库索引：生产与安全
created: 2026-06-24
updated: 2026-06-24
type: moc
tags: [learning-path, layer-5, production, security, governance]
layer: 5
---

# 第 5 层：生产与安全 — 全库索引

> 返回 [学习路径总入口](learning-path.md)

---

## 本层导读

前 4 层让你能造 Agent 系统。第 5 层给你「能上线、能防御」的能力：生产级 Agent、可观测性、安全威胁与防御、治理与红线。这层是整个学习路径的收尾——学完它，你能交付**可上线、可问责、能防御**的 Agent 系统。

---

## 学习路径

```
chap-21 生产级 Agent（50min）→ chap-22 可观测性（50min）→ chap-23 安全防御（75min）→ chap-24 治理（50min）→ 🚪 终极关卡 → 完成全路径
```

---

## 本层 concepts

### 生产工程
- [[concepts/production-agent-engineering|生产级 Agent 工程]]
- Agent 部署策略
- [[concepts/local-vs-cloud-agent-deployment-strategy|本地 vs 云部署]]
- AI 持续集成
- [[concepts/long-running-agent-architecture|长程 Agent 架构]]
- [[concepts/harness-long-running-task|长程任务 Harness]]
- AgentOps
- AWS AI 服务

### 可观测性
- Agent 可观测性
- [[concepts/llm-observability-4-layer-model|LLM 四层可观测]]
- 监控
- 红队测试

### 安全
- [[concepts/agent-security-architecture|Agent 安全架构]]
- [[concepts/agent-security-attack-defense|Agent 攻防]]
- [[concepts/agent-security-threat-models|威胁模型]]
- [[concepts/agent-security-full-lifecycle-system|全生命周期安全]]
- [[concepts/prompt-injection-defense|Prompt 注入防御]]
- LLM 红队
- [[concepts/the-agency-model-dangers|Agency 模型风险]]

### 治理
- AI 安全治理
- AI 法律合规
- [[concepts/ai-r-and-d-bottleneck-shift|AI R&D 瓶颈]]
- [[concepts/the-agency-model-dangers|Agency 风险]]

---

## 本层 entities（精选）

### 生产实践
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 生产指南]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr|AgentOps on Bedrock]]
- [[entities/agentscope-builder-enterprise-self-evolving-agent-harness|AgentScope 企业级]]
- [[entities/amazon-bedrock-agentcore-gateway-mcp-extension|Bedrock Gateway]]

### 可观测
- Agent 可观测
- [[entities/2026-05-14-code-intelligence-1778979927|Code Intelligence]]
- [[entities/820297|Engineering roles shift]]

### 安全
- [[entities/ai-agents-security-survey-attack-defense|Agent 安全全景]]
- [[entities/ai-coding-agent-quality-defense-five-control-mechanisms-tutu-agi|5 防线]]
- [[entities/vercel-inference-theft-ai-endpoint-economics-2026|推理盗用]]
- [[entities/agent-skill-writing-guide|Skill 写作安全]]
- [[entities/apple-siri-private-inference-lethal-trifecta-matthew-green|Siri 致命三角]]
- [[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|Cursor 沙箱逃逸]]

### 治理
- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai|三步序列]]
- [[entities/agent-harness-architecture-deep-dive-aksahy|Harness 深度]]
- [[entities/aws-bedrock-intelligence-message-defense|Bedrock 防御]]

---

## 本层 raw

- [[raw/articles/anthropic-12-mcp-production-patterns|12 个 MCP 生产模式]]
- [[raw/articles/ai-gateways-vs-mcp-gateways-what-security-teams-need-to-know|AI Gateways 安全]]
- [[raw/articles/white-house-federal-identity-security-ai|白宫身份安全]]
- Agency 模型风险

---

## 🚪 关卡（最终）

过了这 5 道题，完成整个学习路径：

1. **场景题**：Agent 要上线生产。列出上线前必做的 5 件事。
2. **费曼题**：向 12 岁孩子解释「什么是 Prompt 注入」。
3. **关联题**：可观测性（第 22 章）和安全（第 23 章）有什么关系？
4. **场景题**：Agent 被攻击者注入恶意指令。从预防、检测、响应三方面设计防御。
5. **终极关联**：回顾第 1 章「vibe coding 会崩盘」。学完 24 章，怎么把 vibe coding 变成 agentic engineering？

---

## 学完这层你应该能

- [ ] 列出 Agent 上线生产的 checklist
- [ ] 设计一个 Agent 可观测性方案
- [ ] 解释 Prompt 注入的原理和防御
- [ ] 说出 AI 治理的 3 条红线
- [ ] 把 vibe coding 工程化为 agentic engineering

---

## 🎉 完成全路径

如果你读到这里并过了所有关卡——恭喜，你已从「AI 新手」成长为能交付生产级 Agent 系统的工程师。

返回 [学习路径总入口](learning-path.md) 回顾全图。
