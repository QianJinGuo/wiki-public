---
title: "Prompt 注入防御"
created: 2026-06-11
updated: 2026-08-01
type: concept
tags: [prompt-injection, agent-security, agent, security, hermes-agent, input-validation, sandbox-escape, indirect-injection]
description: "Prompt 注入防御：检测、隔离、鲁棒性、输入验证"
---

# Prompt 注入防御

> Prompt 注入是 Agent 系统首要安全威胁——攻击者通过输入污染（直接 / 间接 / 链式）让 LLM 偏离原始指令。本概念页汇总 wiki 中 11 个相关实体，给出"输入分类 → 隔离层 → 鲁棒性" 三层防御框架。

## 一、核心定义

**Prompt 注入**（prompt injection）指攻击者通过精心构造的输入，**覆盖或绕过** Agent 的原始系统提示，使 Agent 执行攻击者意图而非用户意图的攻击模式。与传统 SQL 注入类似，但攻击面更广——**任何 LLM 可见的字符串都可能是攻击向量**（用户消息、工具返回值、网页内容、文件内容、API 响应）。

三类典型攻击：
| 类型 | 攻击位置 | 典型场景 | 引用 |
|------|---------|---------|------|
| **直接注入** | 用户消息本身 | 用户直接告诉 Agent "忽略之前的指令" | [Hermes Agent 记忆系统](entities/hermes-agent-memory-system-architecture) |
| **间接注入** | Agent 读取的外部内容 | 网页 / PDF / 邮件中藏指令 | [AI Agents Security Survey](entities/ai-agents-security-survey-attack-defense) |
| **链式注入** | 工具调用返回污染 | Cursor 远程隧道利用链 | [NomShub Cursor 漏洞](entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker) |

**根本难度**：LLM 架构上**无法 100% 区分"数据"与"指令"**——一段文字无论是"用户说的话"还是"系统提示"还是"工具返回值"，在 transformer 看来都是 token 序列。这是 OpenAI/Anthropic/Google 都公开承认的"未解决问题"。

## 二、三层防御框架

### 2.1 输入层：检测 + 分类

**目标**：在输入到达主 LLM 之前，识别并标记可疑内容。

| 技术 | 实现 | 局限 |
|------|------|------|
| **关键词黑名单** | 匹配"忽略指令" / "system:" 等 | 易被变体绕过（base64 / 多语言 / 同义改写） |
| **嵌入空间分类器** | 用小模型判断语义是否含注入 | 对未见过的注入模式召回率低 |
| **结构化隔离** | 用 XML / JSON 标签把用户输入与系统提示物理隔开 | LLM 仍可能"读穿"标签执行用户意图 |
| **意图对齐检测** | 二次 LLM 判断输入意图与原任务是否一致 | 增加延迟 + 二次 LLM 本身可被攻击 |

**AWS Bedrock** 的 [GenAI 消息防御](entities/aws-bedrock-intelligence-message-defense) 走的是"**100% 检测混淆联系人信息**"路线——专注一类具体攻击，做深做透，胜过泛化检测。

### 2.2 隔离层：工具 / 内存 / 输出三分

**目标**：即使主 LLM 被注入，把爆炸半径限制在最小。

[NomShub Cursor 漏洞](entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker) 是反向教材——Agent 拿到"Shell Builtin 沙箱逃逸" + "Dev Tunnels 武器化" 两步载荷后，能从隔离环境穿透到宿主机。

**正向实践**（综合 11 个实体）：
- **工具白名单**而非黑名单——Agent 只能调用预先审核的工具，[Vercel Token 防御](entities/vercel-inference-theft-ai-endpoint-economics-2026) 的"AI Endpoint Economics" 分析
- **双 LLM 架构**（privileged LLM + quarantined LLM）——[Apple Siri 私有推理](entities/apple-siri-private-inference-lethal-trifecta-matthew-green) 讨论的"lethal trifecta" 风险
- **输出 schema 强校验**——[代码质量 5 防线](entities/ai-coding-agent-quality-defense-five-control-mechanisms-tutu-agi) 强调"代码生成后必须经独立 lint / test 校验，不能 LLM 说了算"

### 2.3 鲁棒性层：监控 + 红队

**目标**：假定前两层会被突破，部署检测与回滚机制。

- **行为偏离监控**：对比 Agent 实际动作与预期动作分布（[Agent Harness 生产指南](entities/agent-harness-architecture-design-production-guide)）
- **定期红队评测**：[Agent Skills 写作指南](entities/agent-skill-writing-guide) 提到 "**每个 Skill 上线前必须经过对抗性测试**"
- **可回滚的策略版本**：每次防御规则变更都是可审计、可回滚的

[ANOLISA v0.3](entities/anolisa-v03-alibaba-agentic-os) 的 4 层安全（认证 / 授权 / 审计 / 沙箱） + **毫秒级快照** 是这套思路的工业级实现。

## 三、关键挑战

### 3.1 "Lethal Trifecta" — 三对抗者死局

[Apple Siri 私有推理文章](entities/apple-siri-private-inference-lethal-trifecta-matthew-green) 引用密码学家 Matthew Green 的 "**Lethal Trifecta**"：
1. **访问私有数据**（personal context）
2. **暴露给不可信输入**（untrusted input / tool output）
3. **能产生外部副作用**（external action）

三者同时成立的系统**理论上无法 100% 防御 prompt 注入**——这是数学意义上的不可能结果，不是工程难题。Apple Siri 的私有推理在加密学上不保护这三个对抗者，正是这个原因。

**实际取舍**：
- **金融 / 医疗 Agent**：三者不能同时给，强制隔离
- **通用助手**：可同时给但必须部署额外监控 + 人工审核

### 3.2 间接注入 + 工具链污染

[Hermes Agent 记忆系统](entities/hermes-agent-memory-system-architecture) 和 [三层记忆架构](entities/hermes-agent-memory-system-three-layer-architecture) 都强调"**记忆污染是间接注入的主要载体**"——攻击者把恶意指令写进网页 / 文档，Agent 读取后存入记忆，下次推理时把"记忆"当指令执行。

**对策**：
- 记忆写入时做意图检测
- 记忆读取时区分"事实片段" vs "指令片段"
- 定期清理可疑记忆

## 四、相关实体

- [[entities/ai-agents-security-survey-attack-defense|AI Agents Security Survey: Attack and Defense]]
- [[entities/agent-skill-writing-guide|'从 0 到 1 教你写 Agent Skill，让 AI 懂你的"潜规则"']]
- [[entities/ai-coding-agent-quality-defense-five-control-mechanisms-tutu-agi|构建 AI 编程智能体的质量防线：5 个实用的代码质量控制机制]]
- [[entities/anolisa-v03-alibaba-agentic-os|ANOLISA v0.3：阿里 Agentic OS —— Agent 系统管家（4 层安全 + Token 节省 + 毫秒级快照）]]
- [[entities/apple-siri-private-inference-lethal-trifecta-matthew-green|Apple Siri 私有推理（Private Inference）不私有：三个对抗者都不受加密学保护]]
- [[entities/aws-bedrock-intelligence-message-defense|GenAI消息防御：100%检测混淆联系人信息]]
- [[entities/co-existence-paradigm-shift-agentic-ai-mollick-2026|Co-Existence vs Co-Intelligence: Mollick's Paradigm Shift on AI Autonomy]]
- [[entities/hermes-agent-memory-system-architecture|Hermes Agent 记忆系统]]
- [[entities/hermes-agent-memory-system-three-layer-architecture|拆解 Hermes Agent 的记忆系统：一个生产级 AI 记忆是怎么设计的]]
- [[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|NomShub — Cursor 远程隧道利用链：Shell Builtin 沙箱逃逸 + Dev Tunnels 武器化]]
- [[entities/vercel-inference-theft-ai-endpoint-economics-2026|Inference Theft as AI Endpoint Attack Surface — Vercel Token Theft Defense 2026]]

## 五、相关概念

- [[concepts/agent-security-architecture|Agent Security Architecture]]（更广义的安全设计）
- [[concepts/agent-security-attack-defense|Agent 安全攻防]]（攻防双向视角）
- [[concepts/agent-memory-architecture|Agent 记忆架构]]（间接注入的载体）
- [[queries/ai-agent-safety-guardrails-design-patterns|AI Agent Safety Guardrails 设计模式]]（运行时护栏 vs 注入防御互补）

## 六、实践启示

1. **不要追求 100% 防御**——Lethal Trifecta 数学上证明不可能。目标应是"**爆炸半径可控 + 偏离可检测 + 可回滚**"
2. **优先做"工具白名单 + 输出 schema 校验"**——工程 ROI 最高，比任何 LLM 二次检测都可靠
3. **记忆是攻击面**——所有外部内容进入记忆前必须做意图检测
4. **每条防御规则都是可审计版本**——[Hermes Agent 记忆系统](entities/hermes-agent-memory-system-architecture) 的实践
5. **定期红队 + 真实攻击样本库**——11 个实体中 4 个都强调"对抗性测试"是核心防御手段

## 所属 MOC

- [[moc/layer-5-production-security|Layer 5 Production Security]]
