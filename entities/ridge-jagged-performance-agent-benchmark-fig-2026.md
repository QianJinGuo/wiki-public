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

## 深度分析

### 1. Contested-set 数学：排行榜头部差距在统计上是脆弱的

177 题中 66 题（37%）全员解出、21 题（12%）全员失败，这 87 题（49%）对排序零贡献——20.9pp 的模型间差距**完全**来自中间 90 题 contested set。Astra 领先 Fable 5 仅 1 题（78.5% vs 78.0%），远小于 ±3pp 噪声带；用比例检验的语言说，1/177 的差异在任何合理的显著性水平下都不可分辨。更尖锐的推论：**在 contested set 只占一半的基准上，头部模型之间 2-3pp 的均值差几乎必然落在噪声内**——"谁是第一"在当前样本量下是一个不可判定问题，而媒体与选型报告仍按第一名宣读。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:264-265]

### 2. 统计方法学批判：单次 pass + 循环轴 + harness 混淆三重削弱读数

- **单次 pass**：每个 cell 只跑一次，±3pp 噪声带来自单个 Opus 4.7 重复 run（50-step 预算），并非各模型各自测得的方差；作者在 Limitations 中承认重复 pass 才能给出实测 variance。McNemar p=0.150 的 Opus 5→5.5（+5.1pp）严格说连"均值上升"都无法断言。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:806-807]
- **循环轴**：区分力最强的"按模型自身结果分桶"轴是从被排序的同七个模型构造的，对第八个模型是否成立无法验证——作者明确把这列为 future work（模型无关的难度预测器）。这意味着基准自带的 visual/workflow 难度标签与模型真实难度的脱节不是实现瑕疵，而是**静态难度标注的范式缺陷**：难度是模型相关的，基准却发布模型无关的标签。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:802]
- **harness 混淆**：Opus 系用 computer-use 动作、其余模型用 function calling，绝对成功率跨模型比较被 harness 差异污染；Kimi K3 的 4,096-token 输出预算有时在出动作前耗尽，触发重试直到失败。跨家族比较的"排名"读数须打折。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:808-811]

### 3. 均值不可靠是双重的：模型方差之外还有测量误差

任务级分析不仅暴露模型 jaggedness，还暴露**评分器本身在制造 jaggedness**：放宽字符串匹配的修正评分使每个模型翻转 4-9 题（Astra 9 题），BrowserGym checker 移植把两道正确订单（695、699）判为全员失败。更刺眼的是两个域上 trivial baseline 赢过 Gemini 3.5-Flash：IndEgo 上"复读同任务另一录像中查询索引处步骤"得 0.403 vs Gemini 0.377，Bench2Drive 精确匹配下恒答 FOLLOW_LANE/STOP 得 0.233 vs 0.220。结论：**均值误差 = 采样误差 + 评分器误差，后者在 agent 评测里常大于前者且方向任意**——item-level 数据的价值一半在于它让评分器缺陷可见。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:796]

### 4. "均值静止"更新的回归定位问题：visual-ranking 案例的方法论教训

Opus 4.7→5 的 −1.1pp 均值下藏着 36 个 episode 翻转（17 得 19 失）和 11/21 类别变差；但定位回归本身也有陷阱：visual-ranking 是作者自造的类别（手写词表匹配，仅 21 题），其回归与 classifieds 任务占比（72.9% ranking 题 vs reddit 4.5%）和样本量混淆。教训是双层的——**不仅要任务级 diff，还要区分"真实能力回归"与"类别构造 × 任务组合的统计伪影"**；Astra 在 visualwebarena.179 上"在错误构建的候选集上正确排序"（把街机柜贴纸读成 Pac-Man）说明中间推理步骤的错误可以不体现在最终排序上，这类失败只有 trace 级数据才能捕捉。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:537]

### 5. Benchmark 设计的博弈面：已发布类别可被优化

作者明言"已发布的类别可被针对性优化"（published categories can be optimized against），且一次性测量的 profile 在下一次模型发布或 prompt 修改后就过时。这与 Goodhart 动力学同构：一旦"按类别报告 profile"成为惯例，训练侧就会对着类别签名过拟合。这为 [[concepts/evaluation-harness-design]] 的 harness 中立性设计增加了一条 agent 时代约束——**profile 报告粒度本身会改变被测系统的优化目标**，长期方案应是私有/轮换的任务分类轴而非公开固定类别。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:821]

## 实践启示

1. **选型别看榜首差 2-3pp**：先查 contested-set 重叠与非支配任务（Kimi K3 落后 20.9pp 仍持 6 个 Astra 失败任务）——任何模型在你的任务子集上都不必是全局弱者。[[entities/kimi-k3-2-8t-params-open-source]] 一类"低均值"模型可能恰好覆盖你的关键任务。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:264-265]
2. **模型升级验收用任务级 diff 而非均值 gate**：均值 ±0 时仍可能有 36 题翻转、11/21 类别变差；上线前按任务类别出 before/after competence profile，明确"交换了什么"。这对 [[entities/agent-eval-counterintuitive-insights-langfuse]] 所述"评测不是单一数字"的落地是可操作的验收协议。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:530-535]
3. **评测基础设施三件套**：item-level 结果 + trace 全量发布（RIDGE 模式）、重复 pass 估计实测方差、以及**审计评分器**（字符串匹配 verifier 每模型翻转 4-9 题；两域 trivial baseline 赢过前沿模型应触发 metric 换用加权 F1）。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:794]
4. **Router/ensemble 有量化空间**：七模型集合覆盖 88.1% vs 最强单模型 78.5%——9.6pp 的差距完全来自各自持有对方的失败题，这是任务级路由（per-task model selection）高于"选最强单模型"的直接证据，值得在 [[concepts/agent-evaluation-benchmark-frameworks]] 的框架下作为部署架构选项评估。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:264-265]
5. **profile 会过时，要设计再测量机制**：一次测量的 profile 在下次模型发布或 prompt 修改后失效，且公开类别可被优化——把能力画像当作持续运行的基础设施（每次发布重跑 contested set）而非一次性报告。^[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr.md:821]

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks]] — 评测框架设计原则
- [[entities/anthropic-cyber-evals-incidents]] — 评估运行的事故面
- [[entities/kimi-k3-2-8t-params-open-source]] — 参评模型（VWA 57.6%，最低分但仍持 6 题 non-dominated）
- [[entities/opus-55干一半就开溜官方揭秘你的程序替它打了下班卡]] — Opus 5.5 家族
- [[entities/openai-astra-looped-transformer-technical-clarification-raschka-2026]] — Astra 模型族

→ [[raw/articles/blog-astra-opus-5-5-and-other-frontier-models-demonstrate-jagged-performance-acr|原文存档]]
