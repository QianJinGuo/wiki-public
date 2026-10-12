---
title: "VeriLoop E2：循证收敛递归可验证智能"
created: 2026-10-12
updated: 2026-10-12
type: entity
tags: [verifiable-intelligence, harness, post-training, benchmark]
sources: [raw/articles/黑洞信息悖论推进至定量完备性模量清华团队开源veriloop-e2迈向可验证智能]
confidence: 0.75
---

# VeriLoop E2：循证收敛递归可验证智能

## 深度分析

清华深圳国际研究生院团队的 VeriLoop E2：把「探索权」与「提交权」拆开——模型负责候选生成/路线探索/诊断修订，但不拥有对自身结果的最终确认权。机制 VGR（VeriLoop-Governed Recurrence，循证收敛递归）：每个候选外显为可检查制品，由预先固定的外部验证合同按受保护证据坐标裁决，返回证据秩向量；只有所有受保护项不退化且至少一项严格改善才提交写入状态链，否则保留已验证状态并把失败证据反馈下一轮。同一证据语义组织后训练监督：训练目标从「生成看似合理的答案」转向「提高产生可验证可提交候选的概率」。分工：VeriLoop Harness（上下文装配/工具调用/验证器调度/状态持久化/失败回滚/证据绑定）+ E2（候选生成/问题重构/路线选择/失败后策略调整）。已跑 9 项公开 Benchmark（仓库级软件工程/真实终端/长程代码修复/竞赛数学/科学问答），发布 BF16-IQ1_M 官方 GGUF 量化并以 canonical BF16 为冻结参照测 PPL/KL 保真。 ^[raw/articles/黑洞信息悖论推进至定量完备性模量清华团队开源veriloop-e2迈向可验证智能.md]

## 关联

- [[concepts/agent-harness-engineering-paradigm|Harness Engineering]]
- [[concepts/agent-self-improvement-loops|Agent 自改进循环]]
- [[entities/ai-evals-methodology|AI Evals 方法论]]

→ [[raw/articles/黑洞信息悖论推进至定量完备性模量清华团队开源veriloop-e2迈向可验证智能|原文存档]]
