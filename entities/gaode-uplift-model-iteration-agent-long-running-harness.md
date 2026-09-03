---

title: "高德 Uplift 模型迭代 Agent：长时间运行 Harness"
created: 2026-06-10
updated: 2026-08-29
tags: [agent, architecture, code, data, database, evaluation, fine-tuning, game, harness-engineering, llm, memory, mlops, nvidia, observability, open-source, search, tool-use, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/gaode-uplift-model-iteration-agent-long-running-harness
---

# Gaode Uplift Model Iteration Agent Long Running Harness


## 相关实体

- [[entities/arcis-website-pages-dev-blog-posts-xz-utils-and-the-trust-shift|xz, two years on: what scanners still cannot catch]]
- [[entities/factory-missions-multi-agent-shipping-for-days-luke|一个 mission 跑 16 天、烧 7.78 亿 token：factory 公开了多 agent 系统的构建哲学]]
- [[entities/gemma-4-and-what-makes-an-open-model-succeed|gemma 4 and what makes an open model succeed]]
- [[entities/model-harness-fit-agent-harness|model-harness-fit-agent-harness]]
- [[entities/what-ive-been-building-atom-report-post-training-course-fini|what i’ve been building: atom report, post-training course, ]]
→ [[raw/articles/gaode-uplift-model-iteration-agent-long-running-harness|原文存档]] ^[raw/articles/gaode-uplift-model-iteration-agent-long-running-harness.md]

- [[moc/data-infrastructure|MOC]]
## 深度分析

Gaode Uplift Model Iteration Agent Long Running Harness 涉及agent领域的核心技术议题。 ^[raw/articles/gaode-uplift-model-iteration-agent-long-running-harness.md]
### 核心观点
1. > 来源：高德技术
> 作者：信息业务中心
> 原文：https://mp.
2. com/s/LHPA3qlEsKOlrSsDPEnAyA
## 本期导读
高德营销算法团队构建的 AI Agent 系统：只需输入一句话目标（如"训练发券模型，目标击败 online baseline"），便能自主完成"提出假设 → 拼接样本 → 训练模型 → 离线评估 → 迭代决策"的全链路闭环。 ^[raw/articles/gaode-uplift-model-iteration-agent-long-running-harness.md]
3. **效益：** 过去工程师完成一次完整模型迭代通常需要 3–5 天；该 Agent 系统可在1–2 天内无人值守地跑通同等流程，工程师介入次数 = 0。
4. ## 一、它是什么
一个 AI Agent 系统，专做一件事：替算法工程师跑完 **Uplift 模型迭代的完整生命周期**（Uplift 模型预测的是"给用户发券能多撬动多少 GMV"，是营销算法的核心资产）。 ^[raw/articles/gaode-uplift-model-iteration-agent-long-running-harness.md]
5. **输入：** 一段自然语言（例: "训练旅游 uplift 模型, 目标 sim 胜率 > 50%"）
**输出：** 1-2 天后给你一个训练完的模型 + AUUC 评估报告 + 整个过程的审计日志。

### 关联实体

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/你不知道的-agent原理架构与工程实践-v2]]
- [[entities/ethan-he-cosmos-grok-imagine-latent-space-video-agent-20260606]]

