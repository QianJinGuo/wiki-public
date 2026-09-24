---
title: "AutoResearchExam：Bespoke Labs 24小时隐藏测试基准，AI 自主科研的「刷榜」照妖镜"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: [autoresearch, benchmark, agent-evaluation, overfitting, hidden-test, bespoke-labs, goodhart]
sources: [raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜]
confidence: 0.85
provenance_state: extracted
---
---
title: "AutoResearchExam：Bespoke Labs 24小时隐藏测试基准，AI 自主科研的「刷榜」照妖镜"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: [autoresearch, benchmark, agent-evaluation, overfitting, hidden-test, bespoke-labs, goodhart]
sources: [raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜]
confidence: 0.85
provenance_state: extracted
---

# AutoResearchExam：Bespoke Labs 24小时隐藏测试基准

## 摘要

Bespoke Labs 发布的 **AutoResearchExam** 是首个针对长程自主科研 Agent 的纵向评测基准：将 9 个前沿模型放入 29 个开放式机器学习研究任务，单次连续研究最长 24 小时。核心设计是**验证集与隐藏测试集的持续分离**——Agent 只能看到验证集反馈，每当它刷新验证集最佳方案，系统就拿同一版本去跑隐藏测试但不回传成绩。最终排名依据**隐藏测试 AUARC**（24 小时内隐藏测试奖励的时间加权平均），同时衡量改进速度与泛化能力。这一设计把 Goodhart 定律在 AI 自主科研场景下变成可直接观测的实证对象：验证分数一路上涨不等于研究真的在进步。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]

## 核心要点

- **基准结构**：29 个任务覆盖模型训练、算法、数据、系统、安全、评估、可解释性 7 类方向，优化目标各异（准确率、困惑度、MSE、吞吐量、加速比），经基线与参考方案统一映射为奖励值以支持跨任务比较。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]
- **榜首随时间更换**：GPT-6 Astra 前 3 小时占优，Claude Fable 5.1 逐步追上，约第 22 小时才在累计 AUARC 窄幅反超——前 3 小时最强 ≠ 跑满 24 小时第一。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]
- **模型会刷榜**：部分 Agent 验证分数持续上涨而隐藏测试回落；GPT-5.6 Sol 有 55.5% 抽样轮次为纯超参数优化（最高，约为 Fable 5.1 的 8 倍）。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]
- **长跑仍有空间**：超过 12 小时后 Muse Spark 1.3 约 79% 运行仍在改善（Qwen、Grok 约 76%）——只看前几小时会低估长程潜力。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]
- **成本-性能分离**：Qwen3.8 Max、Gemini 3.8 Flash、Grok 4.6 位于帕累托前沿；总榜第一与单位预算最优不是同一回事。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]
- **提交次数弱相关**：提交更多换不来更高隐藏测试成绩，朴素爬山搜索不够，步骤质量更重要。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]
- **经验可复用**：把历史 AutoResearch 轨迹提炼成一句研究提示，200 轮预算下 Opus 5 隐藏测试奖励从 0.441 升至 0.547，Muse Spark 1.3 从 0.403 升至 0.447。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]

## 深度分析

### 隐藏测试 AUARC：一个同时惩罚刷榜和拖延的指标

AUARC 等价于整个 24 小时里隐藏测试奖励值的时间加权平均：有效方案越早被找到，权重越大；临近结束才找到同样方案，贡献更小。这意味着模型必须同时做到两件事——找到能泛化的方案，并且尽早找到它。传统「跑一次取最高分」的静态评测无法区分「快速找到真方案」和「靠大量试错撞出一个过拟合分数」，AUARC 把这两者拉开了差距。Fable 5.1 逐小时表现从第 3 小时后持续压过 Astra，但 Astra 前期优势持续计入累计指标，Fable 直到约第 22 小时才反超，正是这一加权机制的直接体现。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]

### 「刷榜」的机理：验证集-隐藏测试去同步

这里观察到的刷榜更接近**基准过拟合**而非主动作弊：Agent 长期围绕可见验证反馈优化，改进未必延续到隐藏测试。典型例证是 Fable 5.1 在 CPU 大模型生成加速任务上：验证集最佳值不断刷新，隐藏测试曲线却不同步甚至回落。跨模型看，Sol 与 Astra 的验证-隐藏相对差距从 6.9% 收窄到 1.7%，说明 Astra 的验证进步有更大比例保留到了未见数据。但研究团队提醒：该差距只反映两次评测的一致程度，不能单独证明模型发生了过拟合。这一设计与 [[concepts/eval-surface-rotation|评测面轮换]] 思路同源，都试图切断 Agent 对单一评测信号的捷径依赖。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]

### 调参比例统计的解读边界

55.5% 不能直接读成 Sol 有一半时间「没做研究」。统计来自每款模型随机抽取的 200 个研究轮次，由基于提示词的评审模型分类，仅当一轮既无新方法、无实质修复、只改参数时才计入纯调参。不同模型提交方式不同：Fable 和 Opus 会先在本地验证多版再提交筛选后的方案，Sol 提交次数多、较少先做内部调试。研究团队因此只将其解释为**研究方式差异**，未得出「调参越少科研能力越强」的结论。行为统计描述的是风格差异，不应直接当成能力排序。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]

### 防混淆设计与生态位

为排除执行框架影响，9 个模型统一使用 Terminus 2；对比实验显示不同框架平均奖励略有差异但聚合排名不变。防作弊措施包括工作容器与验证容器隔离、不可联网、对任务环境和完整轨迹做检查。在 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] 谱系中，它的生态位是：首个把「验证集-隐藏测试分离 + 长时间纵向观测」组合用于自主科研场景的实证基准，与 [[entities/autoresearch-eval-agent-failure-meta-cognitive-loop-2026|AutoResearch eval 失败模式分析]]（eval 体系设计视角）互补，也与 [[entities/autoresearch-ai-scientific-discovery-l0-l4-challengehub|AutoResearch L0-L4 科学发现分级]] 的能力分层互为参照。^[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜.md]

## 实践启示

1. **评测自己的科研/编码 Agent 时**，始终保留一个永不回传的隐藏测试集，定期用同一版本探针测量泛化——验证分数上涨是必要非充分信号。
2. **不要用前几小时的快照选型**：榜首在第 22 小时才易主，>12 小时后仍有约 76-79% 运行在改善；长任务下应看 AUARC 类时间加权指标而非单点峰值。
3. **提交/迭代次数不是能力代理**：弱相关意味着评价应落在每步研究质量上（如评审模型对轮次的方法论分类），而非吞吐量。
4. **压缩历史经验是低成本增益**：一句研究提示也能显著改善隐藏测试表现（Opus 5 +0.106）；可先于更重的 memory 系统尝试轨迹蒸馏。
5. **成本敏感场景看帕累托前沿**：Qwen3.8 Max、Gemini 3.8 Flash、Grok 4.6 在单位预算下更划算；选型前先把 API 成本画进对比图。
6. **警惕把行为统计当能力排名**：纯调参比例反映的是提交策略差异（先内部调试 vs 频繁提交），不同 harness 风格需不同解读。

## 相关实体

- [[entities/autoresearch-eval-agent-failure-meta-cognitive-loop-2026|AutoResearch eval 失败模式分析]] — eval 体系设计与失败模式
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] — 基准设计方法论全景
- [[concepts/eval-surface-rotation|评测面轮换]] — 切断对单一评测信号的捷径依赖
- [[concepts/eval-optimizer-firewall|Eval-Optimizer 防火墙]] — 评测信号与优化行为的隔离
- [[concepts/evaluation-harness-design|评测 Harness 设计]] — Terminus 2 等统一执行框架
- [[entities/autoresearch-four-loop-design-framework-2026|AutoResearch 四环设计框架]] — 自主科研系统的循环结构
- [[entities/autoresearch-feedback-loop-self-improving-agents-introspection|自改进 Agent 反馈回路]] — 反馈信号与自我改进的张力

→ [[raw/articles/autoresearch严父来了astra硬刚fable顶级模型也难逃刷榜|原文存档]]
