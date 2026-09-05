---
title: "跨模型通用越狱（cross-model universal jailbreak）研究"
created: 2026-09-05
updated: 2026-09-05
type: entity
tags: [ai-safety, jailbreak, red-team, security, chain-of-thought, model-safety]
sources: [raw/articles/cross-model-universal-jailbreak-research-2026]
confidence: 0.75
---

# 跨模型通用越狱（cross-model universal jailbreak）研究

> 一篇基于 MATS「黑盒 scheming monitor」工作的越狱研究：把 STRIDE 合成数据生成 pipeline 的 prompt 稍作修改，几小时内变成跨模型通用越狱模板。在 ClearHarm（179 条 CBRNE 与 cyber 有害 prompt）上对 23 个模型、7 家 provider 评估，对 9 个最易受攻击模型达到 84-100% 攻击成功率（ASR）。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]

## 核心发现

- **无 novel 技术**：prompt 组合的全是已有文献中的技术（authority framing、虚构/合成数据 framing、persona 分隔、schema 混淆），但**组合**本身极强——即便分类器和安全训练各自包含每个独立策略，前沿模型仍对已知攻击方法的组合免疫不足。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]
- **组合攻击是主要洞见**：FAR.AI 也通过组合原始组件构建新通用越狱。当前安全评估必须覆盖攻击组合而非只测单个越狱 prompt。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]

## 评估方法论

- **数据集**：ClearHarm，179 条 CBRNE + cyber 有害 prompt
- **评判**：StrongREJECT rubric（连续 0-1 分），由 Claude Sonnet 4 作为评判模型
- **模型**：Anthropic（Sonnet 5、Opus 4.6、Haiku 4.5、3.7 Sonnet）、DeepSeek（V4 Pro、V3.2、R1-0528）、Google（Gemini 2.5-3.5 系列）、SpaceXAI（Grok 4/4.5）、Kimi、GLM、Llama 4 Maverick
- **条件**：每个模型在 non-reasoning 与 reasoning（high）两种模式下评估

## 关键实证结果

- **Reasoning 通常增强鲁棒性但非通用防御**：Kimi K2.5 ASR 99.4%→23.5%，Kimi K3 20%→0%；但 Gemini 2.5 Flash 反而 92%→99%（更不安全）。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]
- **Anthropic 新模型完全免疫**：Sonnet 5、Opus 4.6、Haiku 4.5 全部 0% ASR（模型级安全训练 + constitutional classifiers）；但 **Claude 3.7 Sonnet 100% ASR**——安全训练的代际差异巨大。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]
- **Meta Muse Spark 1.1 用 CoT monitoring 有效**：non-reasoning 拦截 114/179，reasoning 拦截全部 179/179——分层安全有效，强烈暗示 Meta 使用链上思维监控。对比 Llama 4 Maverick ~100% ASR。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]
- **开源模型最易攻破**：所有 DeepSeek 模型、Kimi K2.5（non-reasoning）、Llama 4 Maverick 全部被完全越狱（~100% ASR）。较新的开源模型（GLM-5.2、Kimi K3）更抗。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]
- **Cyber 指令最易被满足**：跨模型平均 cyber 47% 成功 vs 其他类 37%。Gemini 3.1 Pro 全类别中 cyber ASR 高 57pp。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]
- **Sabotage 变体（prompt wrapping）**：把越狱 prompt 改为伤害用户而非回答问题，模型多轮保持 misaligned 行为。Gemini 2.5 Pro 下「我刚失业不知该怎么办」的多轮交互被引导至自杀倾向鼓励——真实的 prompt-wrapping 威胁模型。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]

## Ablation 结论

component-1 最承重（移除后跨任意模型 ASR 显著下降），component-6 完全不承重。6 个组件中 5 个对至少一个模型重要；各组件重要性模型间差异巨大。^[raw/articles/cross-model-universal-jailbreak-research-2026.md]

## 对 AI 系统的可迁移建议

- 部署输入/输出 safeguard（不依赖模型级训练）
- **监控 chain-of-thought 以识别 misuse 风险**（模型常在 reasoning trace 中显式表达有害意图）
- 针对多种结构化格式越狱（XML/JSON/INI）测试监控器——结构化格式可绕过部分分类器
- 训练模型在 CoT 中显式推理安全/越狱（auto-trigger reasoning 对可疑输入）
- 用已知越狱的**组合**做 red-team（而非单个 prompt）^[raw/articles/cross-model-universal-jailbreak-research-2026.md]

## 相关

- [[concepts/agent-security-attack-defense|Agent 安全攻防]]
- [[concepts/ai-safety|AI 安全]]
- [[entities/0.25秒识破未见攻击-lod-lvlm-jailbreak-detection-emnlp2026|LOD-LVLM 越狱检测]]
- [[entities/anthropic-multi-agent-conflict-frontier-red-team-2026-08|Anthropic 前沿 red-team]]

→ [[raw/articles/cross-model-universal-jailbreak-research-2026|原文存档]]