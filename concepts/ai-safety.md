---
title: AI 安全（AI Safety）
created: 2026-09-05
updated: 2026-09-05
type: concept
tags: [ai-safety, red-team, alignment, governance, prompt-injection, safety]
confidence: 0.7
provenance_state: merged
---

# AI Safety（AI 安全）

> `ai-safety`(32) 簇的收拢页（2026-09 概念缺位第二轮批量消化）。与既有页面的分工：[[concepts/ai-ethics-responsible-ai|AI 伦理]]管"应该做什么"（公平/问责/社会影响），本页管"怎么不让系统伤人"（攻击面/对齐/治理机制）；[[concepts/agent-security-attack-defense|Agent 攻防]]与 [[concepts/prompt-injection-defense|注入防御]]是本页在 agent 场景的战术深化，[[concepts/eval-optimizer-firewall|评测防火墙]]是验证侧的隔离机制。

## 簇内三条线

**1. 攻击面的理论化**：[[entities/role-confusion-github-io|Role Confusion]] 把 prompt injection 重新定义为**角色感知失败**——LLM 靠角色标签（system/user/tool）理解输入边界，注入本质是角色边界的可协商性。这给战术页（[[concepts/prompt-injection-defense|三层防御]]）提供了理论地基：防御的对象不是"恶意字符串"而是"边界协商"。

**2. 红队作为常态化职能**：[[entities/latent-space-p-gray-swan|Red-Teaming after Mythos（Zico Kolter）]]与 [[entities/anthropic-multi-agent-conflict-frontier-red-team-2026-08|Anthropic 多智能体冲突实验]]（"安全是整体属性而非个体属性"）把红队从"发布前活动"重定义为**与系统共存的持续过程**；[[entities/openart-agent-red-team-evolving-environment-2026|OpenArt]] 则证明静态评测面在自适应攻击者面前必然失效——防御评测需要 [[concepts/channel-enumeration-criterion|通道枚举]]与轮换。

**3. 治理与对齐的制度层**：[[entities/welcome-to-the-agi-era-of-ai-governance-interconnects|Nathan Lambert 论 AI 治理]]（治理互连）与 [[entities/openai-beneficial-rl-broadly-persistently|OpenAI beneficial RL]] 代表制度/训练两条对齐路径；[[entities/claude-opus-48-system-card-analysis|系统卡片分析]]是厂商自查的透明化载体。

## 未决张力

系统卡片的自查红队 vs 外部红队（Gray Swan 等安全公司）存在结构性利益差——库内 [[entities/anthropic-mythos-glasswing-30days-vulnerability-report|Mythos 30 天漏洞报告]]事件是这一张力的实证案例。自查充分性与外部可验证性如何兼得，簇内无页处理（登记为开放问题）。

## 检索入口

[[entities/role-confusion-github-io|Role Confusion]] · [[entities/latent-space-p-gray-swan|Gray Swan 红队]] · [[entities/anthropic-multi-agent-conflict-frontier-red-team-2026-08|多智能体冲突]] ｜ [[concepts/ai-ethics-responsible-ai|AI 伦理（姊妹页）]] · [[concepts/agent-security-attack-defense|Agent 攻防]] · [[concepts/channel-enumeration-criterion|通道枚举判据]]
