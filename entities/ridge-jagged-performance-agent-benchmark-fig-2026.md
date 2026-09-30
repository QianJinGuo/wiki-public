---
title: "RIDGE：Agent 基准均值掩盖 jagged performance — Fig 五域任务级评测"
created: 2026-10-01
updated: 2026-10-01
type: entity
tags: [agent, evaluation, benchmark, web-agent, robotics, multimodal, frontier-model, item-level-analysis]
sources: [raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr]
confidence: 0.75
provenance_state: extracted
---

# RIDGE：Agent 基准均值掩盖 jagged performance

## 概述

Fig 团队（Fig + Georgia Tech）2026-09-29 发布技术报告，对前沿模型（Astra、Opus 5.5、Fable 5、GPT-5.6、Opus 4.7、Opus 5、Kimi K3）在五个任务域做**任务级（item-level）**评测：闭环 web 任务（VisualWebArena 177 题 × 7 模型）+ 四个离线物理域（Bench2Drive-VL 3,746 段、VLABench 644 回合、IndEgo 300 段、Assembly101 520 项 × 3 模型），并发布 **RIDGE 数据集**（全五域 item-level 结果 + 模型 trace）。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]

**核心概念 — jagged profile**：模型成功率在同一基准的各个任务间剧烈波动，而基准只报告一个均值。部署决策若只看均值，会掩盖模型在具体任务上的失败分布。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]

## 四个核心发现

1. **没有明确的性能领先者**。全部 21 个模型对中，低分模型至少有 1 个任务是高分模型失败的；最宽的对差达 21 题。Astra 均值最高 78.5%（139/177），但仅领先 Fable 5（78.0%）1 题，在 ±3pp 噪声带内；Kimi K3 落后 Astra 20.9pp 仍持有 6 个 Astra 失败的任务。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]

2. **Web 任务上模型-任务拟合方差极高**。单个模型跨三个网站（classifieds/reddit/shopping）的成绩波动（Opus 4.7 达 23.9pp）与七个模型在单一网站上的波动同宽。基准自带的难度标签（visual/workflow grade）对模型成败的解释力很弱——七个模型中五个在 workflow-hard 上得分**反而更高**（Fable 5 +11.2pp）。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]

3. **模型更新改变 profile 而不动均值**。Opus 4.7 → Opus 5 全套均值 -1.1pp（McNemar p=0.868），但 21 个任务类别中 11 个变差、双向交换任务（17 vs 19）。GPT-5.6 → Astra 均值 +7.9pp（p=0.009）仍丢失 6 个 GPT-5.6 能解的任务。**均值不变 ≠ 任务集合不变**。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]

4. **物理域同样 jagged**。四个物理域（驾驶/装配/操作/工业流程）上，每个域均值都超过 Gemini 3.5-Flash 的模型，仍在某些任务类别上更差。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]

## 方法设计要点

- **双轴变化**：模型轴（7 模型 × 1 闭环域 VWA）+ 域轴（3 模型 × 4 离线域），每个 cell 单次 pass，用重复 Opus 4.7 run 估计的 ±3pp 噪声带读数。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]
- **contested set 分解**：177 题中 66 题（37%）全部模型解出、21 题（12%）全部失败——这两组对每个模型加同样的分；模型间 20.9pp 的差距**完全来自中间 90 题（51%）contested set**。七模型合计解 156 题（88.1%），比最强单模型多 17 题。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]
- **按模型自身结果的分桶轴**区分力最强（bucket 1→5 每模型降 32-76pp），但作者自己指出这是循环轴（从被排序的同一批模型构造）。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md]

## 与 wiki 已有评测观点的关联

- 佐证 [[concepts/agent-evaluation-benchmark-frameworks]] 的"评测不是单一数字"立场：item-level 数据 + trace 公开（RIDGE）是比 benchmark mean 更诚实的报告粒度。
- "任务类别对可靠性的影响 ≈ 模型选择的影响"（引 [15]）与 [[entities/agent-eval-counterintuitive-insights-langfuse]] 的反直觉发现同族。
- VWA 上七模型 88.1% 集合覆盖 vs 最强单模型 78.5% 的差距，为 [[entities/anthropic-cyber-evals-incidents]] 一类"评估环境任务覆盖"问题提供了量化参照。

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks]] — 评测框架设计原则
- [[entities/anthropic-cyber-evals-incidents]] — 评估运行的事故面
- [[entities/kimi-k3-2-8t-params-open-source]] — 参评模型（VWA 57.6%，最低分但仍持 6 题 non-dominated）
- [[entities/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡]] — Opus 5.5 家族
- [[entities/openai-astra-looped-transformer-technical-clarification-raschka-2026]] — Astra 模型族

→ [[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr|原文存档]]
