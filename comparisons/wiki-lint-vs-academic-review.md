---
title: "Wiki Lint vs 学术同行评审：质量保证机制对比"
created: 2026-06-12
updated: 2026-06-12
type: comparison
tags: [comparison, wiki-lint, peer-review, quality, knowledge-base, academic]
sources: [entities/hermes-wiki-9-step-auto-growing-knowledge-network, concepts/agent-evaluation-benchmark-frameworks]
---

## 对照表

| 维度 | Wiki Lint | 学术同行评审 |
|------|---|---|
| 核心机制 | 自动 lint 规则 | 专家人工评审 + 双向匿名 |
| 覆盖面 | 100% 全库 | 单篇论文 2-5 评审 |
| 速度 | 秒级 | 数周-数月 |
| 发现类型 | 结构（失链/孤儿/缺源） | 内容（论据/方法/创新） |
| 可量化 | ✅ 0 errors 是硬 gate | ❌ 标准不统一 |
| 成本 | 零（一次性投资） | 高（专家时间） |
| 适用阶段 | 持续/每次 commit | 重要出版前一次 |

## 判断

互补：lint 是「24/7 守门员」抓结构错（失链、缺源、重复），人工评审是「专家定期审计」抓内容错（方法、创新、可复现）。最佳实践：每次 commit pre-hook 跑 lint（必须 0 errors），关键 entity 定期 (季度) 人工 review。

## 对比方来源

- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes-Wiki lint 实战]]
- [[concepts/source-first-knowledge-compilation|source-first 编译]]
- data quality pipeline
- [[concepts/evaluation-harness-design|evaluation harness 设计]]

## 进一步阅读

- [[entities/hermes-wiki-9-step-auto-growing-knowledge-network]]
- [[concepts/agent-evaluation-benchmark-frameworks]]
