---
title: "ATLAS: Evaluating Agents on Search-Intensive Tasks"
created: 2026-10-12
updated: 2026-11-18
type: entity
tags: [agent, benchmark, search, evaluation, web-search]
sources: [raw/articles/atlas-agentic-search-benchmark-exa]
confidence: 0.75
---

# ATLAS: Evaluating Agents on Search-Intensive Tasks

## 概述

Exa 于 2026-10-08 发布 ATLAS（Evaluating Agents on Search-Intensive Tasks），一个面向 agentic search 的基准：以真实搜索需求为锚点的 query 配对经过验证的 golden answers，配自动化管线随 Web 与模型演进持续刷新。核心度量是三档 F1（precision/recall 谐和）：Discovery F1（实体）、Row F1（整行正确才算对，headline 指标）、Item F1（单元格级，覆盖 discovery 与 enrichment）。^[raw/articles/atlas-agentic-search-benchmark-exa.md]

## 核心发现

- **穷尽式搜索远未解决**：max-effort search agent 系统性优于低算力版本，但每任务成本低于 $1 的 agent 没有一个 Row F1 超过 0.5。^[raw/articles/atlas-agentic-search-benchmark-exa.md]
- **头部 agent 也漏 1/3**：即使最贵的 search agent 仍 miss 约 1/3 golden results——对完整性敏感的场景（如广度信息收集）存在显著改进空间。^[raw/articles/atlas-agentic-search-benchmark-exa.md]
- **固定 model harness 后搜索后端是变量**：Exa 定义了 cost-performance Pareto 前沿，不同 search backend 间分数差达 16%。^[raw/articles/atlas-agentic-search-benchmark-exa.md]

## 定位

ATLAS 的叙事转折是"agentic search 的瓶颈已从智能转移到搜索基础设施"：早期 agent 卡在长上下文管理与跨文档推理，如今 frontier model 已能处理，瓶颈转向搜索栈本身（backend 选择、成本、完整性）。基准引用系把自身置于 WANDR、BrowseComp、WideSearch、DeepSearchQA 等同类系谱中，差异点是行级完整性 F1 + 持续刷新的 golden answers。^[raw/articles/atlas-agentic-search-benchmark-exa.md]

## 深度分析

### 三档 F1 与评分器：把完整性变成可测量的量

三个指标共用 precision/recall 谐和形式，只换计数单元：Discovery F1 数实体，Item F1 数单元格（覆盖发现与充实），Row F1 数整行——每个单元格都对整行才算对，是 headline 指标。两个设计细节最值得注意。其一，约 3.8% 的 golden 单元格是 blank（构造 agent 已证明该信息无公开记录），只有 agent 留空才得分，把"推断某事不存在"也变成可评分能力；另有 7.1% 无法仲裁的单元格标记 ungraded 直接排除，而不是强行给答案。其二，评分便宜且确定：先用实体名与别名集合做确定性对齐，失配才回退 LLM aligner，仅 4.5% 单元格交 LLM 裁决；给一个系统全部 547 任务评分约 $0.24，对 7,658 份答案重评 98.6% 的行级 F1 完全一致、任何系统均值移动 <0.001。对比 WANDR 每份答案判分 $0.88、7.6 分钟且每条记录都过 LLM——固定答案键加确定性评分，正是 ATLAS 能廉价反复回归的原因。

### 与既有基准的对照：记忆化、饱和与"是否真的在测搜索"

系谱表把老问题量化了：BrowseComp（2025-04，1,266 题）记忆化 48% 且已饱和；WideSearch（2025-08）59%；DeepSearchQA（2026-01）61%；WANDR（2026-07）无固定答案键、无法测量，评 10 次完整 run 的判分成本 $18k–50k；ATLAS 是 547 题、记忆化 9%、判分 $2。两个消融最能说明差异：把每 10 条搜索结果的前 7 条隐藏后，同一 searcher 在 ATLAS 上丢掉约一半分数，而在 BrowseComp/WideSearch/DeepSearchQA 上仍保留 81–89%——说明后三者对搜索质量几乎不敏感；在 WideSearch 上强 agent 48% 的任务只引用单一网站，ATLAS 上仅 0.2%。合起来读：不少流行基准测出的是参数化记忆而非检索，这与 [[entities/model-evaluation-from-benchmark-worship-to-self-built-evals|从膜拜 benchmark 到自建 eval]] 对基准失效方式的批评一致。

### 构造管线：五阶段如何造出可信的 golden answer

任务从 Exa 聚合的匿名化真实搜索需求中聚出 300+ 主题作种子，由两个模型家族的 agent（Codex + GPT-6 Astra、Claude Code + Opus 5.5）独立构造：实体发现阶段逐行点检并核对引用页面，行充实阶段每个值附页面引用与逐字引文，分歧进定向修复，终检用 GPT-5.6 Sol。去偏的关键是一个 provider 中立搜索工具：每次查询同时调 Exa、Brave、Perplexity，返回隐藏来源名的交错列表，实测构造 agent 对三家的引用率接近相等，golden 不偏向任何单一索引。压力测试让 12 个系统跑全部任务；审计由盲判 agent 复核每个分歧，抽样估计 golden 值错误率 <1%（0.9%，95% CI 0.6–1.4%）。单任务构造中位投入 8 agent 小时、约 1,200 次搜索；管线全自动，故可每 3–6 个月从最新需求再生成数据集，让基准随 Web 演进而非腐化。

### 宽度与深度即难度

74% 的任务要求发现 ≥10 个实体，每个实体再补 2–10 个多跳属性；据构造日志估计，实体发现中位跨 4 个域名，发现+充实中位跨 18 个域名——不是单 URL 或单域检索可完成的"名义上的宽"。实验显示宽度与深度任一增加，所有被测 searcher 的 Row F1 都下降，找到行中全对的份额也随列数增加而衰减。最好系统 0.66 Row F1 的代价是每任务 $8.92、18 分钟——这解释了 headline 发现：瓶颈已不在模型推理，而在搜索基础设施能否以可控成本做到又宽又深的穷尽覆盖。

## 实践启示

1. **把 search backend 当一等优化变量**：harness 与模型固定时，仅换 backend 分数即差 16%。选型应自建 cost–performance Pareto 曲线而非只比 API 单价，同思路见 [[entities/harnesspro-pareto-self-evolution-cost-performance-thu-zhipu-2026-11-04|HarnessPro Pareto 自进化]]。
2. **检索类评估优先整行指标**：单元格级均值会掩盖"漏整行"式完整性失败——最好系统仍 miss 约 1/3 golden 结果，Item F1 却可能不难看。完整性敏感产品（广度收集、监控、汇编）应直接用 Row F1 型指标做回归门禁。
3. **<$1/任务的穷尽式搜索当前不可行**：所有 <$1 run 都没过 0.5 Row F1。成本敏感场景要么把任务设计成无需穷尽枚举（top-k 足够），要么明确向用户标注"结果可能不完整"，而不是指望提示词解决。
4. **评测面要轮换与刷新**：老基准近半到六成任务被闭卷记出，分数随训练数据更新而虚高；对策是从真实使用需求播种加自动化再生成管线，见 [[concepts/eval-surface-rotation|评测面轮换]]。
5. **golden set 构造流程可直接借用**：双模型家族独立构造 + 逐项验证 + provider 中立搜索（隐藏来源）+ 盲审 + 人工兜底仲裁，适用于任何自建 eval 的参考答案生产；垂直领域对应解见 [[entities/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026|LoHoSearch 用知识图谱校准评测]]。
6. **把"信息不存在"纳入评分**：blank 单元格只有留空才得分，专测 agent 能否判断"该事实无公开记录"。事实抽取类系统都应加此类负例，否则会倾向编造而非留空。

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]
- [[entities/anysearch-agent-search-engine-product-hunt-2026|AnySearch 搜索 agent]]
- [[entities/arbor-hypothesis-tree-research-agent-2026|Arbor 研究假设树 agent]]
- [[entities/anthropic-dynamic-workflows-ultracode-deep-research-lyuyuebannzi|Deep Research harness 实践]]

→ [[raw/articles/atlas-agentic-search-benchmark-exa|原文存档]]
