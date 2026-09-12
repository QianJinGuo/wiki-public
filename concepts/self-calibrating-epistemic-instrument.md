---
title: 自校准认识仪器
created: 2026-09-13
updated: 2026-09-13
type: concept
tags: [epistemics, temporal, retrieval, half-life, truth-maintenance, emergence, meta]
confidence: 0.65
provenance_state: inferred
---

# 自校准认识仪器（Self-Calibrating Epistemic Instrument, SCEI）

> 定义：一个 LLM 时代的知识库，若以 claim 为原子、以自身修正事件流（预测判定、权威取代、矛盾解决）为钟，实测类条件真值衰减常数，并把衰减与权威状态耦合进检索排序与写入门禁，则它构成一台**测量自身内容真值的仪器**——库既是被测对象，也是测量装置。理论化于 [[drafts/wiki-emergent-viewpoints-2026-09-unification|2026-09 统一透镜轮]]：该轮把前十轮透镜产出与库内一等机制收拢为此单一对象；"本 wiki 此前不存在这一命名理论对象"，本页即其常量化。

## 五命题

| # | 命题 | 库内承载机制（均已运行） |
|---|------|--------------------------|
| P1 本体论 | 知识库的原子单位是 claim，文档只是 claim 的容器 | claim 级引用 `^[raw/...]` 纪律（SCHEMA 约定） |
| P2 动力学 | claim 真值是时间随机过程，类条件半衰期可从库自身修正事件流估计 | [[concepts/claim-half-life]]（首测三类常数，见下表） |
| P3 检索律 | 检索排序是信念修正问题而非纯相似度问题：score = sim × 生存 × 权威 | 真值衰减检索（TDR，本页内核）+ [[drafts/supersedes-relation-design|supersedes 权威边]] + contradicted_by 矛盾边 |
| P4 可证伪闸门 | 时间量化前瞻必须预登记、有到期日、有判定词汇 | [[queries/prediction-ledger]]（五值判定词汇 + 到期回访仪式） |
| P5 反身性 | 库用自己的修正事件为自己的衰减常数定标——测量行为改变测量系统 | `ledger-backpropagate.py` 判定回传（首跑波及 17 页、收割首条半衰期观测 115 天） |

## 真值衰减检索（Truth-Decay Retrieval, TDR）

```
score(q, c) = sim(q, c) × S_κ(c)(Δt) × A(c)
S_κ(Δt) = exp(−λ_κ · Δt)，  λ_κ = ln2 / T½(κ)
```

| 项 | 含义 | 库内现状 |
|----|------|----------|
| sim(q,c) | 语义相似度 | 既有嵌入管线（qmd 索引 + Ollama 旁路） |
| S_κ(Δt) | 类条件生存函数，κ∈{术语, 代际SOTA, 发布交付, 机制} | 三类已首测；n≥5/类 后验重算挂 12 月判定窗（提案卡 #3，勿提前） |
| A(c) | 权威因子：supersedes 链降权 + 矛盾压力衰减 + 台账 falsified 硬门 | supersedes 活边冷启动中（提案卡 #4）；台账判定吞吐见 [[queries/vault-evolution-dashboard|仪表板]] |

TDR 与外部最近邻的可证伪差异不在"要不要衰减"，而在**衰减常数从哪来**：2026-09 摄入的 temporal RAG 实验把半衰期作为手工定值参数（默认 h=14 天），作者自述这些值是"经验启发式、换域需重调"，并把"学习更丰富的时序模型"明确列为未来工作^[raw/articles/arxiv-2509-19376-temporal-rag-freshness-trend-detection.md]。TDR 的回答是：h 不该调出来，该从判定事件流里**估**出来——每个 λ_κ 背后是对账记录而非参数扫描；这就是 P2 与 P5 闭合的回路。

## 形式化（2026-09-13 验证轮增订）

两条可证命题（证明见 [[drafts/wiki-emergent-viewpoints-2026-09-unification-verify|验证轮涌现稿]]观点一）：

- **门控引理**：加性/混合聚合（score = α·sim + (1−α)·g，α 严格为正）不能实现"被证伪断言排在全部真断言之后"的硬门——被证伪页永远保有 α·sim 的分数地板；乘积形式下 A=0 即精确沉底。权威门是 TDR 取乘积形式的存在理由，也是与 temporal IR 凸组合路线的种类差异（非参数差异）。
- **删失估计引理**：类条件指数生存下，失效率由暴露量 Y（页日）与失败数 D 估计：λ̂=D/Y；D=0 时半衰期的置信下界为 ln2·Y/3（"三的法则"，95% 档），推翻检测器敏感度不足时按比例缩水。[[concepts/claim-half-life|半衰期页]]的机制类行已按此口径增订。

聚合形式约定：TDR 取乘积（信念修正语义：证据因子相乘修正可信度）；temporal IR 的凸组合与 TDR 是不同聚合器——同极限（无衰减、无权威时同流）、不同形式（凸组合无法沉底被证伪页）。

## 首批实测常数（2026-09-05 首测，随对账流滚动更新）

| 断言类 κ | 首测半衰期 T½ | 观测来源 |
|----------|---------------|----------|
| 术语类（vibe coding → agentic engineering） | ≈30 天 | [[entities/karpathy-software3-vibe-coding-dead-agentic-engineering]] 对账链 |
| 模型代际"最强"类 | ≈59 天 | Opus 4.8 → Opus 5 系统卡对 |
| 发布预期 vs 交付类 | ≈109 天 | DeepSeek V4 论文 → 正式版实测对账（台账 #1） |
| 机制/原理类 | 未测得（0 条被推翻） | [[concepts/eval-optimizer-firewall]] 等机制页 |

完整的测量协议、读库打折规则与待测假设由 [[concepts/claim-half-life]] 承载；负向实证（"干预 X 使 Y 变差"）由 [[queries/negative-results-registry|负结果登记簿]] 承载——三者加 [[queries/prediction-ledger|台账]] 构成库内断言的三轴对账。

## 与外部最近邻的判别

| 最近邻 | 已有 | 缺（SCEI 增量） |
|--------|------|-----------------|
| Temporal IR 衰减重排（Metzler 2009 → 2509.19376） | 固定 h 的全局新鲜度先验、as-of 查询 | 类条件测量、权威边、判定闭环、反身定标 |
| Zep/Graphiti 时序知识图谱 | 实体关系时序演化、历史关系保留为一等维度 | 衰减常数未测、无预登记台账、无判定回传 |
| TMS / AGM 信念修正（1979/1985 经典） | 真值维护与信念修正的形式理论 | 从未在 LLM 语料上以**实测**衰减常数运行 |
| 科学计量学引文半衰期 | 引用行为衰减的大规模测量 | 测的是引用行为，不是 claim 真值本身 |

外部记忆系统评述（[[raw/articles/what-makes-good-agent-memory-system-yuanrunzi-2026|好的 Agent 记忆系统六维评述]]、[[raw/articles/存之有序治之有矩agent-记忆系统的工程实践与演进|记忆系统工程实践综述]]）确认：把"时序演化"做成一等维度已是行业共识方向，但公开材料止步于"保留历史关系"，没有任何系统报告按 claim 类别实测衰减常数——这正是本库用自家对账流能补上的实证空位。

## 可证伪预测

理论要活到被检验那天：三条预测已预登记于 [[queries/prediction-ledger]] 台账 #12-#14——①类条件半衰期排序稳定性（术语 < 代际 < 交付，随 12 月判定窗首测）；②机制类断言一年存活率 ≥95%；③外部采纳预测（到 2027-06-30 至少一家主流检索/记忆产品把类条件真值衰减或权威失效边做成一等检索特征）。判定走台账五值词汇，判定即收割新半衰期观测。

## 验证状态（滚动更新，2026-09-13 首录）

| 日期 | 验证动作 | 结果 |
|------|----------|------|
| 2026-09-13 | TDR 回溯评测（四案例，预注册脚本 `tdr-retrospective-eval.py`） | 纯相似度在术语链错序（旧版排首位）被 TDR 修复；全局手调衰减在对照组把机制页从榜首误伤至第 9 位、TDR 免疫；权威门行为与门控引理一致 |
| 2026-09-13 | 机制类删失下界（Y=64 页日、D=0） | "机制类≈不衰变"降级为"95% 下界约两周"，12 月判定窗随暴露量机械收紧 |
| 2026-09-13 | 复访率归一（git 触碰历史） | 机制类页面被触碰频率与代际类相当，"零推翻=没人看"被削弱（cron 噪声为残留偏差） |

扩样机制：[[queries/prediction-ledger|台账]]每个新判定自动成为回溯评测新案例；暴露量随时间自然累积，下界自紧。验证轮全记录见 [[drafts/wiki-emergent-viewpoints-2026-09-unification-verify|第十二轮涌现稿]]。

## 关联

- [[drafts/wiki-emergent-viewpoints-2026-09-unification|统一透镜轮涌现稿]] — 本理论的推导现场与查新记录
- [[concepts/claim-half-life]] · [[queries/prediction-ledger]] · [[queries/negative-results-registry]] — P2/P4 与负向轴的常量化容器
- [[drafts/supersedes-relation-design]] — P3 权威边的设计稿
- [[queries/emergence-lens-rotation]] — 透镜目录⑧（统一透镜）的登记处
