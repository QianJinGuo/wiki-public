---
title: "WANDR Benchmark — 评估 Research Agent 的 Wide-and-Deep 研究能力"
created: 2026-07-16
updated: 2026-11-24
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

## 深度分析

### 与 DRACO 的互补分工：wide 与 deep 是两种不同的失败模式

WANDR 的定位是 DRACO 的"wide 兄弟"：DRACO 检验 agent 能否产出准确、完整、客观的长篇深度报告，WANDR 检验它能否构建大规模集合并为每个成员附上具体证据。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:12-14] 文章强调两者结合后问题性质改变——少量精彩案例不够，建立在残缺研究上的流畅叙事也不够，agent 必须在维持大范围发现的同时保住逐条记录的事实质量；竞争映射、尽调、文献综述、人才搜寻等真实委托全部落在这一形态上。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:14,20-23]

### 任务生成：从生产使用模式半自动扩展到 170,495 条记录

任务不从合成提示词出发，而是从去标识化的生产使用模式中挖掘可复用的 wide-research 模式，再经 Seeding（挖掘模式）→ Authoring（作者-批评者交替循环）→ Admission（校验目标量可达）→ Curation（打标并选出平衡集）四阶段半自动生成，人力只投入在质检而非穷举式答案标注。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:31-39] 中位任务要求 50 个成员、每成员 4 条记录、共 245 条；已释放的 ceo_cfo_appointments 任务要求至少 70 家美国公司 × 两类权威页面 = 140 条记录，展示了资格键树如何落到具体任务。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:27,39]

### 评分体系如何量化"部分完成"

每条记录须同时提交 item、URL、摘要（excerpts）与答案，评分器重新抓取页面，检查可用性、声明清晰度、摘要忠实度与证据支持度；二值判定沿资格键层级向上汇总为 Precision（提交质量）、Recall（质量加权完成度）与 F1。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:43-45] 关键区分在 soft/hard：soft 给不完整成员部分分，hard 只计完整子树全部正确的成员。所有系统的 soft recall 均低于 soft precision，说明"提交的内容质量尚可、但集合远未找全"是普遍形态。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:58]

### 规模与层级深度的复合衰减

两条衰减曲线勾勒出 wide-and-deep 的难度结构：目标量从最小桶增至最大桶，Perplexity 的 hard precision 从 0.235 跌至 0.096；中间资格键从零层增至三层以上，hard precision 从 0.392 跌至 0.019、hard recall 从 0.378 跌至 0.017——层级每加深一层，完成质量近乎坍缩。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:60-62] 结构损失定位显示 top-level 发现完成率仅 0.611-0.951，且最大原始损失发生在最终 URL 附着之前：发现而非证据验证才是首要结构性瓶颈。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:64] 成本面同样不可忽略：领先系统每任务 $5.20、中位求解 14.9 分钟、3.82M tokens。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:54]

## 实践启示

1. **同时看 soft 与 hard 两个口径再下结论**：0.363 soft F1 对 0.133 hard F1 的落差说明"看起来完成了"与"子树完全正确"差距巨大，生产验收应以 hard 口径为准。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:45,51-58]
2. **把发现（discovery）列为第一优化对象**：最大结构损失出现在 URL 附着之前，改进检索/采集/去重管线比打磨证据摘要的回报更高。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:64]
3. **为大规模任务预留人工抽检并压低预期**：目标量越大 hard precision 越低（0.235→0.096），数千条记录的委托不能按小任务桶的表现线性外推。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:60]
4. **用分层资格键设计可验证的采集任务**：company(n) → employee(m) → url(k) 这类树状结构让每条完整路径可独立验证，是给内部 agent 布置数据收集工作的可直接复用的任务模板。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:25]
5. **把记录级判定用作 RL 奖励信号**：失败维度（发现/丰富化/身份处理/语义资格/证据构建）可逐条定位，用记录与分支级奖励替代单一稀疏终端奖励来训练 research agent。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:68]
6. **领先幅度与成本要一起读**：[[entities/perplexity-search-as-code-generation|Perplexity Search as Code]] 以 0.363 对 0.249 的 soft F1 领先 Anthropic，但单任务 $5.20、3.82M tokens，规模化部署前需核算性价比。^[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026.md:51-54]

## 与现有实体的关系

- 与 [[entities/miroflow-deep-research-agent-harness-mirothinker|MiroFlow Deep Research Agent]] 构成同一主题的评估与实现两极——MiroFlow 是 deep research 脚手架，WANDR 是测评框架
- 补充 [[entities/perplexity-search-as-code-generation|Perplexity Search as Code]] 的评估维度
- 与 Agent Evaluation Benchmarks 相比，WANDR 聚焦于"wide"而非"deep"维度

→ [[raw/articles/perplexity-wandr-benchmark-research-agents-wide-deep-2026|原文存档]]
