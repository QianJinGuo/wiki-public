---
title: "RRSI: Regularized Recursive Self-Improvement of Agent Harnesses"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: [agent, harness, self-improvement, agent-harness, evaluation, benchmark]
sources: [raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses]
confidence: 0.7
provenance_state: extracted
---

# RRSI: Regularized Recursive Self-Improvement of Agent Harnesses

## 核心主张

递归自改进（RSI）的 Agent harness 演化会**过拟合其被评分的 benchmark**——现有方法在 evolve-split 上收益巨大，但换到 out-of-distribution benchmark 后收益缩水甚至消失，其中两个方法的 OOD 成绩甚至低于演化起点。RRSI 是唯一一个 **OOD 收益随演化增长的方案**，held-out 成绩比此前方法平均高出最多 **22.9%**。^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md]

## 三角色演化循环 + 四条正则化规则

循环：proposer（读完整 ledger，花费递减的 edit budget 提出修改）→ critic（评分前先做 leakage 审查）→ gate（候选必须同时满足噪声底线与 token 成本偿付）。

正则化规则：
1. **Leakage 拒绝**：任务名、实体、答案、benchmark-specific logic 在评分前即被拒绝
2. **噪声底线**：收益必须超过 unchanged base harness 上测得的方差
3. **成本偿付**：额外 inference tokens 必须由可测收益支付
4. **组件裁剪**：停止"自付"的组件被标记删除

^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md]

## 演化粒度控制

早期轮次允许捆绑少量协调修改以发现机制；后期轮次强制单次一个可归因修改。每个候选都记录 hypothesis、diff、score、cost change，proposer 据此复用有效改动、停止重测失败项。停滞在噪声带内时，budget 重定向到从未触及的组件。^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md]

## 评测设计

- 每个领域只在一个 suite 上演化，然后 harness 原封不动跑其他所有 benchmark
- 任务类型：coding suites（pass rate）、Harvey LAB / JobBench / APEX-Agents（model-judged rubric）、GDPval（对人类专家 win rate）、EngDesign / Frontier-Eng（frozen simulators）
- RRSI 是最轻的演化 harness（policy tokens per trial 最低），也是唯一超过未演化 baseline H 的

^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md]

## 与 wiki 已有内容的关联

- 补充 [[concepts/agent-self-improvement-loops]]：现有内容覆盖 RSI 循环设计，RRSI 提供防 benchmark 过拟合的正则化机制这一缺失维度
- 与 [[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026]]、[[entities/aide2-recursive-self-improvement-weco-2026]]、[[entities/ai-recursive-self-improvement-nanogpt-prime-intellect]] 同属 RSI 谱系——RRSI 的差异化在于"正则化优先"而非"搜索能力优先"
- harness 组件可编辑性设计与 [[concepts/agent-harness-engineering-paradigm]] 的 harness-as-artifact 视角直接相关

## 相关链接

- 来源：[[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses|原文存档]]
- 项目页：https://regularized-rsi.com/
- 论文：arXiv:2609.24972（Xia, Peng 等 14 人）
