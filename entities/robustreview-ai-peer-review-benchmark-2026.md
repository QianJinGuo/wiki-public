---
title: "RobustReview：AI 审稿人稳健性基准"
created: 2026-10-14
updated: 2026-10-14
type: entity
tags: [benchmark, peer-review, llm-evaluation, ai-for-science]
sources: [raw/articles/robustreview-ai-peer-review-benchmark-2026]
confidence: 0.7
---

# RobustReview：AI 审稿人稳健性基准

弗吉尼亚理工、马里兰大学与 MBZUAI 团队的 AI 审稿评测基准：60 篇 ICLR 2026 投稿，由 GPT-5.5 与 Claude Opus 4.8 按 10 种策略改写（方法与数据保持原样，仅调整创新性/贡献/实验结果的表述），生成 1200 份改写稿，再交 30 种 AI 审稿配置打分 ^[raw/articles/robustreview-ai-peer-review-benchmark-2026.md]。

## 两个反直觉发现

1. **写法确实能撬动评分**：部分 AI 审稿人的分数受表述方式影响显著；
2. **最稳定的配置最平庸**：评分最稳定的那套配置，区分不同论文真实质量的能力接近随机水平。

团队据此提出 RobustReview 基准与 SciCore 双分支审稿方法，试图解耦「表达质量」与「内容质量」两个评分维度。该工作把 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]的评测思路推进到 AI-as-reviewer 场景——审稿 agent 本身也需要 robustness 评测。

## 关联
- [[concepts/evaluation-harness-design|评测 harness 设计]]
- → [[raw/articles/robustreview-ai-peer-review-benchmark-2026|原文存档]]
