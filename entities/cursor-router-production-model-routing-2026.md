---
title: "Cursor Router: 生产流量驱动的模型路由系统"
created: 2026-08-10
updated: 2026-09-11
type: entity
tags: [cursor, model-routing, inference-cost, llm, production, routing]
sources: [raw/articles/cursor-router-how-cursor-chooses-model-2026]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Cursor Router: 生产流量驱动的模型路由系统

## 摘要

Cursor Router 是 Cursor 于 2026-07-22 随 Auto Intelligence / Auto Balance 一起发布的在线模型路由系统，核心命题是：**模型选择应从真实开发者工作的生产表现中学习，而非从 benchmark 分数外推**。它用「Compass 复杂度预测器 + 任务分类学」两段式结构做逐 turn 决策——先判断该 turn 是否简单到可留在性价比模型，再用 taxonomy 为复杂 turn 挑最合适的前线模型。截至 2026-08 复盘，Auto Intelligence 在成本比 Fable 低 68% 的前提下取得高于 Fable 级的用户满意度，Auto Balance 超越 Opus 4.8 且成本低 41%。^[raw/articles/cursor-router-how-cursor-chooses-model-2026.md]

## 核心要点

- **数据来自生产流量**：数十万个 turn、跨多模型采样，每条含路由可见的对话信号 + 两个结果度量。^[raw/articles/cursor-router-how-cursor-chooses-model-2026.md]
- **性能信号用用户下一步行为反推**：继续下一个任务是强正向信号，纠正 agent 是强负向信号——无需人工标注。
- **成本 = API 定价 × token 用量**，且因来自实时流量，还捕获了 benchmark 常遗漏的成本，例如**切换模型导致的 cache miss**。
- **两段式路由**：Compass 先做模型无关的复杂度估计，用阈值 τ 切分；分数 ≥ τ 才交给任务路由（taxonomy）挑前线模型。
- **任务分类学三维**：Domains（后端 / 数据库 schema / 前端）、Tasks（修 bug / 跑命令 / 写测试）、Modifiers（有界编辑 / 产品问题 / 视觉密集变更）。^[raw/articles/cursor-router-how-cursor-chooses-model-2026.md]
- **没有模型在所有类别都占优**：Grok 胜在常规工作的性价比（Git 命令、通用数据库操作），Sol 强于规划与代码库理解，Opus 擅长执行密集型（devops / 数据库查询 / 性能优化），Fable 则在调试与视觉实现上最优。
- **只在性能明确更好时才路由**：候选模型需在其任务标签上通过相对性价比模型的单侧 75% uplift 阈值（约需 75% 置信度确认改进为真）才具备入选资格。
- **预算内选最优组合**：优化器在平均单 turn 成本不超该模式预算的前提下，选期望性能增益最大的流量加权组合。

## 深度分析

### 1. 路由器的真正难点不是「分类」，而是「无标签的性能估计」

Compass 不是传统意义上的硬分类器，它输出 0–1 的连续分数，本质是以「用户是否会满意」为训练目标训出来的满意度预测器，再把预测值当作复杂度的代理量。真正的巧思在于**用用户下一步行为当作免费的监督信号**：无需人工标注、也不依赖离线 benchmark，就能把「这个 turn 难不难」变成可测量、可校准的量。代价是信号本身含噪——「继续下一个任务」多数时候代表满意，也可能代表用户放弃了这个方向。因此必须上线校准：被评为最可能成功的 turn，96% 收到正向性能信号；被评为最不可能成功的，仍有 71% 收到正向信号。^[raw/articles/cursor-router-how-cursor-chooses-model-2026.md]

### 2. 分层解耦：复杂度门控（model-agnostic）与能力匹配（model-aware）

两级结构是这套系统的工程骨架。第一级 Compass 只回答「难不难」，与具体模型无关；第二级 taxonomy 路由只回答「谁最合适」，依赖对每个模型在各标签上表现的观测。解耦带来的实际收益是**模型池变动时的可维护性**：加入 Opus 5 时第一级逻辑不受影响，只需重学第二级的「模型 × 任务标签」强弱映射。这与「用一个端到端 LLM judge 直接选模型」的路线形成对比——judge 更灵活，但更难做统计显著性与预算约束，而这两点恰恰是生产路由不能放弃的（也是 classifier/heuristic 路线相对 judge 路线的优势所在）。这里 taxonomy 路由本质是一张**标签级、可观测、可用统计检验门控的 route 表**。^[raw/articles/cursor-router-how-cursor-chooses-model-2026.md]

### 3. 质量–成本–延迟三角与常被忽略的「切换税」

该系统成本核算里有一个多数人会漏的项：**切模型造成的 cache miss**。多数成本模型只算 token 单价，忽略跨模型切换时 prompt cache 失效带来的额外 token 重算；Cursor 明确把它计入，因为它来自实时流量而非离线估算。这解释了为什么「每 turn 都用最强模型」的真实开销往往比表面单价更贵。延迟维度文章未展开，但从机制看，τ 与任务路由预算两个旋钮共同决定每种模式在成本-性能曲线上的位置：Auto Balance 把更多流量留在性价比路径、给任务路由更小预算，Auto Intelligence 则允许在增益值得时更频繁升级到前线模型。^[raw/articles/cursor-router-how-cursor-chooses-model-2026.md]

### 4. 离线筛选 + 在线 A/B 的评测闭环

评测走两阶段：先用交叉验证调 Compass 阈值与优化预算，避免对某一数据 split 过拟合；再在 held-out test set 上评测选定策略，用于淘汰弱候选、比较期望成本与性能。文章明确表态：离线分析和 benchmark 都无法完整刻画生产行为，因为 token 用量、缓存、跨模型切换成本都难以离线建模，**实时开发者流量仍是最具代表性的测试**——最终必须在线上按用户满意度与真实单 turn 成本做 A/B 裁决。冷启动问题被数据集构建方式部分缓解：预先跨多模型采样，使每个「任务标签 × 模型」组合在部署前已有可观测量，而非让新模型从零开始摸索。^[raw/articles/cursor-router-how-cursor-chooses-model-2026.md]

## 实践启示

1. **把「用户下一步行为」当免费性能标签**：继续 / 纠正是最廉价、也最贴近真实价值的在线信号，比人工标注和 benchmark 都可扩展；这是自建路由系统时最可复制的单点经验。
2. **成本核算必须算进「切换税」**：将切模型造成的 cache miss 与 token 重算计入成本，否则会系统性低估多模型路由的真实开销，做出错误的路由/上新决策。
3. **两级路由优于单级黑箱**：让复杂度门控（model-agnostic）与能力匹配（model-aware）解耦，模型池频繁变动时只需重训第二级，维护成本显著下降。
4. **用单侧显著性阈值做准入门槛**：75% uplift 置信度是可复用的保守策略——宁可少路由，也不制造质量回退（capability regression），因为一次错误升级比省下的一次成本更伤体验。
5. **把预算作为一等参数**：用阈值 τ 与 per-turn 预算两个显式旋钮直接定义各产品档位在成本-性能曲线上的坐标，而不是靠隐式调参。
6. **评测分两段刚性执行**：离线交叉验证只用于筛候选、排除弱策略；线上 A/B 才定生死。承认离线无法建模缓存与切换成本，能避免「离线很美好、上线就翻车」的典型失败。

## 相关实体

- [[entities/cursor-harness-model-production-floor|Cursor Harness 生产运营]] — 同源 Cursor 工程实践：模型 + Harness 组合作为发布单元
- [[entities/state-of-routing-in-model-serving|模型服务中的路由现状]] — 服务端模型路由的通用格局
- [[entities/netflix-switchboard-lightbulb-model-routing|Netflix 模型路由]] — 另一套生产级模型路由实践
- [[entities/ibm-research-model-routing-optimization-2026|IBM 模型路由优化研究]] — 路由优化的学术路线
- [[entities/cursor-evals-benchmark-3-1-2026-07|CursorBench 3.1]] — Cursor 的离线评测体系（与线上路由信号互补）
- [[concepts/ai-cost-optimization-framework|AI 成本优化框架]] — 成本-质量权衡框架

→ [[raw/articles/cursor-router-how-cursor-chooses-model-2026|原文存档]]
