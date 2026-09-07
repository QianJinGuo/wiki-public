---
title: "WANDR Benchmark — 评估 Research Agent 的 Wide-and-Deep 研究能力"
created: 2026-07-16
updated: 2026-09-07
type: entity
tags: [agent-evaluation, benchmark, research-agent, perplexity, search-as-code, wide-and-deep]
sources: [raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# WANDR Benchmark — 评估 Research Agent 的 Wide-and-Deep 研究能力

WANDR（Wide ANd Deep Research）是 Perplexity 发布的开源 benchmark，用于评估 Research Agent 在**高容量、强证据**的知识工作中的表现。包含 500 个真实场景的数据收集任务，共计 170,495 条可验证记录。与 DRACO（评估深度报告质量）互补，WANDR 聚焦于能否**大规模构建证据支撑的集合**。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md]

## 核心设计

### 分层资格键（Qualification Key Hierarchy）

任务使用灵活的树状结构定义：`company(n) → employee(m) → url(k)` 表示找 n 家公司，每家 m 名员工，每人 k 个支持页面。每个完整路径可独立验证。支持扁平列表、嵌套搜索、矩阵和多证据分支。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md]

### 无参考评分（Reference-Free Grading）

使用固定答案集（answer key）不适合开放式研究。WANDR 在每个记录中同时要求 item、URL、摘要（excerpts）和答案。评分器重新抓取页面，检查页面可用性、声明清晰度、摘要忠实度和证据充分性。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md]

评分指标：
- **Soft scores**：对不完整成员给部分分
- **Hard scores**：仅计完全正确的成员
- **Precision/Recall/F1**：在两级粒度上分别计算

## 主要发现

Perplexity Search as Code 在 0.363 soft F1 / 0.133 hard F1 领先，Anthropic 以 0.249 / 0.072 第二，其余系统上限为 0.121 / 0.035。

**四个核心发现**：
1. **部分进展常见，完整覆盖困难** — 最佳 hard precision 仅 0.150
2. **规模放大问题** — 从最小到最大任务桶，Perplexity hard precision 从 0.235 降至 0.096
3. **层级深度是杀手** — 三层 vs 零层层级 hard precision 从 0.392 降至 0.019
4. **发现（Discovery）是首要瓶颈** — top-level 发现完成率 0.611-0.951，最大损失在 URL 附上前

## 结构化的失败诊断

每个维度可独立定位：发现不足、丰富化失败、身份处理错误、语义资格判断失误、证据构建失败。这种结构化为 RL 训练提供细粒度信号——可以在记录和分支级别给予奖励，替代单一的稀疏终端奖励。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md]

## 与现有实体的关系

- 与 [[entities/miroflow-deep-research-agent-harness-mirothinker|MiroFlow Deep Research Agent]] 构成同一主题的评估与实现两极——MiroFlow 是 deep research 脚手架，WANDR 是测评框架
- 补充 [[entities/perplexity-search-as-code-generation|Perplexity Search as Code]] 的评估维度
- 与 Agent Evaluation Benchmarks 相比，WANDR 聚焦于"wide"而非"deep"维度

→ [[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026|原文存档]]
