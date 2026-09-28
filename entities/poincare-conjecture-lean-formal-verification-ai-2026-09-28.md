---
title: "庞加莱猜想 Lean 形式化验证：470 万行证明的 AI 协作案例"
created: 2026-09-28
updated: 2026-09-28
type: entity
tags: [ai, formal-verification, lean, multi-agent, mathematics]
sources: [raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28]
confidence: 0.7
provenance_state: extracted
---

# 庞加莱猜想 Lean 形式化验证：470 万行证明的 AI 协作案例

## 事件

一个四人小团队（带头人为丘成桐弟子、研究 Ricci 流数十年的教授 + 一名应届本科生 + AI 协作）用证明助手 Lean 将 Hamilton 与佩雷尔曼的庞加莱猜想证明从头形式化，总计约 470 万行代码，全部通过 Lean 内核检查——无一处 `sorry`（"以后再证"占位）^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md]。

## 关键数据

- 总代码量约 470 万行，其中约 270 万行是最后两周在 ChatGPT、Claude 等 AI 帮助下完成
- 顺着最终定理做依赖分析：真正用到的代码为 14197 个文件、约 402 万行；最长一条引用链串起 353 个文件
- 佩雷尔曼的毕生工作只占全部代码的约六分之一^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md]

## 对 Agent 工程的意义

传统同行评议要数年逐页审读，此次"说了算的换成了机器（Lean 内核），写证明的主力换成了 AI"——形式化验证器充当了可机械检验的 ground truth，这是 [[concepts/scei-formal-proofs]] 所述"验证器驱动开发"在数学大定理上的最大规模实证。AI 承担的是"领队-调度-临时工"式多智能体分工中的底层证明生成，人类专家负责数学结构判断——与 [[entities/formalizing-fermats-last-theorem-claude-lean-anthropic-2026]]（费马大定理形式化）构成同族案例，共同指向"AI + 形式化验证器"是高可靠推理任务的工程范式^[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28.md]。

## 关联

- [[concepts/scei-formal-proofs]] — 形式化证明作为 AI 推理的可验证锚点
- [[entities/formalizing-fermats-last-theorem-claude-lean-anthropic-2026]] — 同族先例：费马大定理 Lean 形式化
- [[entities/lean-scaling]] — Lean 形式化的规模化规律
- → [[raw/articles/poincare-conjecture-lean-formal-verification-ai-2026-09-28|原文存档]]
