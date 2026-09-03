---
title: "AI 研究的科学方法论"
created: 2026-07-02
updated: 2026-08-01
type: concept
tags: [science, methodology, evaluation, research]
provenance_state: inferred
confidence: 0.7
---

# AI 研究的科学方法论

> 本页填补知识库在科学方法论维度的不足。

## AI 研究的可重复性危机

AI 领域的研究存在系统性可重复性问题：随机种子敏感、数据集泄露、评估基准污染、cherry-picking 报告。一个严谨的智库需要用科学方法论审视每项"突破"。

## 评估方法论的核心原则

1. **基准隔离**：训练集与评估集严格分离（防止 benchmark contamination）
2. **多种子报告**：报告均值+方差，而非单次最佳
3. **成本-性能曲线**：性能不能脱离成本讨论（Noam Brown 原则）
4. **Harness 变量控制**：隔离 Harness 影响（Claw-SWE-Bench 方法）
5. **统计显著性**：差异是否在噪声范围内？

## 对知识库的启示

知识库的 review 评分体系（value × confidence）是方法论意识的体现，但需要更强的批判性视角：对于每项"SOTA"声明，应附带评估方法论的审查。

## 关联

- [[entities/swe-bench-agent-evaluation|SWE-bench 评估方法论]]
- [[entities/claw-swe-bench-harness-evaluation-benchmark-tokenrhythm|Claw-SWE-Bench]]
- [[entities/noam-brown-ai-evaluation-reasoning-budget-performance-cost-curve|Noam Brown 评估方法论]]
- [[entities/k-dense-the-model-is-no-longer-the-bottleneck|Model Is No Longer the Bottleneck]]

## 所属 MOC

- [[moc/evaluation-and-benchmarks|Evaluation And Benchmarks]]
