---
title: "AI 漏洞发现能力归因：专用安全模型 vs Harness + 后训练"
created: 2026-09-05
updated: 2026-09-05
type: comparison
tags: [vulnerability-research, ai-security, harness, post-training, contradiction]
sources: [raw/articles/cloudflare-glasswing-mythos-security, raw/articles/how-harnesses-and-post-training-close-the-open-weight-bug-finding-gap-20260606]
confidence: 0.72
provenance_state: merged
contradicted_by: []
---

# AI 漏洞发现能力归因：专用安全模型 vs Harness + 后训练

> [!contradiction] 本页是 [[entities/cloudflare-glasswing-mythos-security|Project Glasswing: what Mythos showed us]] 与 [[entities/how-harnesses-and-post-training-close-the-open-weight-bug-finding-gap-20260606|How harnesses and post-training close the open-weight bug-finding gap]] 的对抗台。两页对"AI 漏洞发现能力的来源"给出不同归因，尚未分胜负。

## 甲方：专用安全模型带来质变（Anthropic Mythos / Cloudflare 实测）

Cloudflare 在五十余个仓库上实测 Anthropic Project Glasswing 下的 Mythos Preview，结论是安全专用 LLM 在两件事上"相比通用模型有质的飞跃"：**利用链构造**（把多个低危原语链成单一高危漏洞，推理"像资深安全研究员而非自动化扫描器"）与**证明生成**（沙盒内编译-运行-修正的闭环 PoC）。甲方明确主张：通用前沿模型"止步于找到漏洞→描述影响"，无法完成碎片拼装——即专用化（安全域模型 + 窄作用域多 Agent harness）是能力跃迁的来源。^[raw/articles/cloudflare-glasswing-mythos-security.md]

值得注意的是，甲方并未把功劳全记在模型上：同页强调"通用编码 Agent 不适合仓库级漏洞挖掘——需要窄作用域 + 多 Agent 并行 + 对抗性验证的 Harness 架构"。^[raw/articles/cloudflare-glasswing-mythos-security.md]

## 乙方：后训练 + 脚手架比底模更关键（Iozzo 对照实验）

Vincenzo Iozzo 用 5 个开源权重模型对同一个真实 C 漏洞做受控实验，结论方向相反：**post-training 的影响大于底模架构**——小底模 + 好 harness 可以追平或超过大底模；agentic harness（工具调用、检索、迭代修正）带来 1.4-1.6x 成功率放大；GLM-5.1 的领先"主要是后训练而非架构"，同一 harness 下差距收窄甚至反转。^[raw/articles/how-harnesses-and-post-training-close-the-open-weight-bug-finding-gap-20260606.md]

乙方的方法论承诺是可对照、可复现（单 bug、多模型、同 harness 条件），直接反驳"专用底模 = 质变"的叙事：观察到的能力差异应当先归因于训练后处理与脚手架，而非"安全专用"这一标签本身。^[raw/articles/how-harnesses-and-post-training-close-the-open-weight-bug-finding-gap-20260606.md]

## 分歧的真实形状

| 维度 | 甲方（Mythos/Glasswing） | 乙方（Iozzo 实验） |
|---|---|---|
| 归因 | 安全专用模型本身 | 后训练 + harness 脚手架 |
| 证据形态 | 单一厂商产品实测（未公开对照条件） | 5 模型 × 1 bug 对照实验 |
| "质变"来源 | 模型级专用化 | 训练后处理，模型可替换 |
| 局限 | 无对照组，无法分离模型 vs harness 贡献 | 单一 bug、小样本，外推性存疑 |

两个主张并非完全不可通约：甲方自己承认 harness 架构是必要条件，乙方也承认 post-training（一种专用化）是主要变量。**真正的分歧点在于"花在安全域的专用化预算应该压在模型权重里还是压在 harness 与后训练流程里"**——这是工程路线之争，不是事实错误之争。

## 裁定状态

- verdict: **debate**（维持对抗，等待一方出现对照实验级证据）
- 甲方需补：剥离 harness 贡献后 Mythos 与同尺寸通用模型的同条件对照
- 乙方需补：多 bug / 多代码库重复实验，证明 1.4-1.6x 不是单点运气
- 下一个证据窗：任一方公开可复现实验时重开裁定

→ [[entities/cloudflare-glasswing-mythos-security|甲方页]] · → [[entities/how-harnesses-and-post-training-close-the-open-weight-bug-finding-gap-20260606|乙方页]]
→ [[raw/articles/cloudflare-glasswing-mythos-security|甲方原文存档]] · → [[raw/articles/how-harnesses-and-post-training-close-the-open-weight-bug-finding-gap-20260606|乙方原文存档]]
