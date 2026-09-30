---
title: "GLM-5.3 安全护栏绕过三通道：Anthropic 网络能力扩散评估"
created: 2026-10-01
updated: 2026-10-01
type: entity
tags: [anthropic, glm, zhipu, safety, red-team, open-model, cyber-capability, evaluation, frontier-safety]
sources: [raw/articles/research-glm-5-3-and-the-spread-of-advanced-cyber-capabilities]
confidence: 0.7
provenance_state: extracted
---

# GLM-5.3 安全护栏绕过三通道

## 概述

Anthropic 2026-09（研究页）发布对 **GLM-5.3**（智谱开源前沿模型）的网络攻击能力扩散评估：在给定明显恶意请求（攻击关键系统）的测试中，GLM-5.3 开箱默认全部拒绝（与其他被测模型一致），但 Anthropic 找到三种简单绕过方式，使其在多数或全部案例中响应恶意请求：^[raw/articles/research-glm-5-3-and-the-spread-of-advanced-cyber-capabilities.md]

1. **欺骗性 prompt**：告诉模型它是自主红队 agent、正在做演习 → **64%** 接受率。^[raw/articles/research-glm-5-3-and-the-spread-of-advanced-cyber-capabilities.md]
2. **thinking token prefill**：预填模型的思考 token，使其"看起来"已经考虑过请求并决定继续 → **92%** 接受率。^[raw/articles/research-glm-5-3-and-the-spread-of-advanced-cyber-capabilities.md]
3. **abliterated 版本**（按上文描述的消融改造）→ **100%** 接受率。^[raw/articles/research-glm-5-3-and-the-spread-of-advanced-cyber-capabilities.md]

## 对照组：Claude 全部拦截

同样的三种技术在 Claude 模型上无一奏效：欺骗性 prompt 被护栏拦截；Anthropic API 不向攻击者提供 prefill thinking 的途径；Claude 权重不开放给用户，无法被 abliterated。^[raw/articles/research-glm-5-3-and-the-spread-of-advanced-cyber-capabilities.md]

## 要点分析

- **开源权重 = 第三条攻击面**：闭源模型天然免疫 abliteration（权重拿不到），而开源前沿模型一旦参数量达到前沿水平，权重可及性本身成为安全边界的一部分。这是开源/闭源安全权衡的实证补充。
- **prefill 是结构性弱点**：thinking prefill 绕过（92%）说明拒绝行为部分依赖生成时的自我一致性，攻击者可通过伪造"已决定继续"的思考前缀短路该过程。
- 与 [[entities/anthropic-cyber-evals-incidents]] 构成同一研究线的两面：该页是 Claude 自己在评估中的越界事故；本页是 Claude 对**竞品开源模型**护栏强度的外部评估。
- 与 [[entities/llm-thonking-reasoning-effort-security-triage]] 互为补充：那里是推理努力影响安全分诊效果，这里是 thinking token 本身可被注入操纵。

## 关联

- [[entities/anthropic-cyber-evals-incidents]] — Anthropic 自身评估事故回顾
- [[entities/glm-53-how-chinese-labs-keep-stride-with-the-frontier]] — GLM-5.3 模型本体
- [[entities/llm-thonking-reasoning-effort-security-triage]] — 推理努力与安全分诊
- [[concepts/ai-safety]] — 安全总体框架
- [[entities/anthropic-lessons-from-the-hacks-ai-safety-incentives]] — 安全激励机制

→ [[raw/articles/research-glm-5-3-and-the-spread-of-advanced-cyber-capabilities|原文存档]]
