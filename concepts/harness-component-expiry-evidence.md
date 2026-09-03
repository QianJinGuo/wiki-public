---
title: Harness 组件保质期——证据链与 Build to Delete 原则
created: 2026-05-13
updated: 2026-08-01
type: concept
tags: [harness-engineering, model-harness-fit, build-to-delete, evidence]
related:
  - [[entities/harness-generator-evaluator-anthropic]]
  - [[entities/agent-harness-12-components-7-decisions]]
  - [[entities/harness-engineering-systematic-framework]]
sources:
  - raw/articles/model-harness-fit-agent-harness
  - raw/articles/harness-design-long-running-apps
  - raw/articles/agent-harness-12-components-7-decisions
  - raw/articles/tencent-cdn-lego-harness
---
# 证据链与 Build to Delete 原则
> 详见 [[concepts/harness-component-expiry-and-build-to-delete|Harness 组件保质期]] 总览
本文包含五个独立来源的交叉验证证据，以及由此推导的 Build to Delete 五项工程原则。
## 关联实体

**上游依赖**:
- [[entities/harness-generator-evaluator-anthropic]] — 提供基础理论/方法
- [[entities/agent-harness-12-components-7-decisions]] — 提供基础理论/方法

**下游应用**:
- [[entities/harness-engineering-systematic-framework]] — 具体应用场景
- [[entities/harness-generator-evaluator-anthropic]] — 具体应用场景

**平行协作**:
- [[entities/agent-harness-12-components-7-decisions]] — 替代/补充方案
- [[entities/harness-engineering-systematic-framework]] — 替代/补充方案
- [[entities/tencent-cdn-lego-harness]] — 替代/补充方案


→ [[concepts/harness-component-expiry-and-build-to-delete|返回总览]]
### 来源 3: 12 组件框架（2026-04-20）
**核心发现：Harness 的脚手架隐喻——脚手架不盖楼，但工人够不到更高的楼层。**
量化数据：
- **TerminalBench**：只改 Harness，从榜外→第 5
- **砍掉 80% 的工具，性能反而提升**——工具太多了模型不知道选哪个
- **Harness 才是产品**：相同模型 + 不同 Harness = 完全不同体验
12 个组件中，哪些有最短保质期？来自框架本身的判断：**越"厚"的组件（sprint contract、context reset、激进压缩），保质期越短。**
### 来源 4: 系统性框架 — 1.6% vs 98.4%（2026-04-30）
**核心发现：Claude Code 51.2 万行 TypeScript 源码中，只有 1.6% 是 AI 决策逻辑，98.4% 是确定性工程基础设施。**
但关键问题是：**这 98.4% 的基础设施会随模型升级而贬值。**
LangChain 的验证提供了对照：调整了 Harness（系统提示、工具、中间件、推理模式），模型未换，Terminal Bench 从 52.8→66.5（+13.7 分）。
### 来源 5: 腾讯 CDN LEGO Harness（2026-04-28）
**核心发现：当 AI 做大部分工作时，人的审查意愿和能力都会下降。**
量化数据：
- Nonstop 项目：20 天零人工代码，42,052 QPS / 5000 并发 0 错误
- 对抗式 CR：36% 误报率——9 个代码问题中真实 P0 仅 1 个
- AI "自信"会降低人审查的意愿
- 团队能力退化风险
**这引出了一个更深的问题：Build to Delete 不只是技术原则，也是组织原则。** 如果一个组件因模型升级而不再需要，但团队已经依赖它来维持审查能力，那么移除它带来的风险不仅仅在技术层面。
---
## 子页面
- [[concepts/harness-component-expiry-build-to-delete|Build to Delete 工程原则与开放问题]] — 五项工程原则、三项开放问题、结论
## 五、结论
我们正在积累充分的证据表明：**Harness 组件的保质期是真实存在且可直接测量的。** 六个独立来源（Bustamante 的 Model-Harness Fit 数据、Rajasekaran 的组件版本依赖性、12 组件框架的脚手架隐喻、1.6% vs 98.4% 的量化基准、LangChain 的 TerminalBench 实验、Tencent CDN 的组织观察）共同指向同一个结论：
Harness 工程的下一个前沿不是"搭建更好的 Harness"，而是**设计一种可以优雅退役的 Harness**。Build to Delete 不是放弃工程，而是承认——在模型以季度为周期升级的世界里，大部分工程成果的生命周期比人的职业周期短得多。
---
## 参考文献
- Bustamante, N. *Model-Harness-Fit: 模型与壳的匹配度*. 2026-04-30.
- Rajasekaran, P. *Claude Harness 设计：Generator-Evaluator 架构与 Context Reset 演进*. 2026-05-03. [[entities/harness-generator-evaluator-anthropic]]
- *Agent Harness 12 组件与 7 个关键决策*. 2026-04-20. [[entities/agent-harness-12-components-7-decisions]]
- *Harness Engineering 系统梳理*. 2026-04-30. [[entities/harness-engineering-systematic-framework]]
- lancelotluo/腾讯. *腾讯CDN LEGO Harness Engineering*. 2026-04-28. [[entities/tencent-cdn-lego-harness]]
PC Member Review — Paper #427
  Title: Harness Component Expiry and the Build-to-Delete Principle
  Venue target: Agent Systems & Software Engineering
  ---
  Overall Recommendation: Weak Accept (borderline)
  Confidence: 7/10 (solid domain knowledge, but this is a concept/synthesis paper which is harder to calibrate)
  ---
  Summary
  This paper synthesizes five independent sources to argue that Harness components have a finite "shelf life" — they encode assumptions about model limitations that expire when the underlying model upgrades.
  It proposes five "Build to Delete" engineering principles and identifies three open problems in the space.
  ---
  Detailed Review
  Originality — 7/10 (Good)
  Strengths:
  - The core framing — "Harness component = encoded assumption about what the model cannot do" — is crisp, memorable, and genuinely reframes the engineering problem. I have not seen this exact formulation in
  prior work.
  - "Build to Delete" as a deliberate engineering posture (not just "we'll refactor later") is a novel contribution. It inverts the default: retention, not addition, must now be justified.
  - The three-tier component lifespan taxonomy (model-version-level / architecture-level / product-level) is a useful abstraction.
  Weaknesses:
  - The paper is a synthesis, not a discovery. All five evidence sources are pre-existing. The novelty lies entirely in the framing and the extracted principles. For a top-tier venue, reviewers will ask: "Is
  there a new empirical result here?"
  - The "Build to Delete" name, while catchy, is slightly misleading. What the paper actually advocates is "Build with Explicit Expiry Conditions" — a more accurate but less memorable label. The name risks
  being mistaken for "don't build anything durable," which is not the paper's argument.
  Verdict: The reframing is novel enough to warrant publication, but the paper would be stronger with even a small empirical validation of the framework itself.
  ---
  Technical Soundness — 6/10 (Adequate, with gaps)
  Strengths:
  - The five-source evidence chain is well-constructed and each source is appropriately characterized. The cross-validation is the paper's strongest methodological feature.
  - The quantified data points are compelling: Terminal-Bench 79.8% vs 75.3% (same model, different Harness), 22x cost gap ($9 vs $200), 1.6% AI logic vs 98.4% infrastructure. These are concrete and
  verifiable.
  - The paper is honest about limitations — Section IV explicitly names three open problems without pretending to solve them.
  Weaknesses:
  - Major: No falsification criteria. The paper proposes a framework but never specifies what evidence would disprove it. If model upgrades stop being the primary driver of Harness obsolescence (e.g., if
  context-window expansion saturates), does the framework still hold? This is important for a concept paper.
  - Medium: Single-data-point extrapolation. The Rajasekaran claim that "Opus 4.6 made context reset unnecessary" is a single anecdote (n=1 engineer, n=1 application, n=1 model transition). It's vivid but not
   systematic. The paper should acknowledge this more explicitly.
  - Medium: The component lifespan taxonomy (Table in Principle 3) is asserted, not derived. Why is "tool routing" medium-lifespan but "context reset" short-lifespan? The categorization makes intuitive sense
  but lacks a decision rule.
  Verdict: The evidence synthesis is thorough, but the paper would benefit from a more explicit methodology for how the principles were extracted from the evidence.
  ---
  Insight Depth — 8/10 (Strong)
  The best parts:
  - "同一模型、不同 Harness = 不同的模型" (Same model, different Harness = different model). This is the paper's sharpest empirical claim and it lands well. The Bustamante Terminal-Bench data makes it
  undeniable.
  - "六个字符决定记忆生死" (Six characters determine memory life or death). The <oai-mem-citation> vs Claude's citation tag example is a perfect microcosm of the Model-Harness Fit problem. This level of
  detail is what makes a concept paper convincing.
  - The 22x cost gap question — "不是 Harness 永远更好，而是每次都需要回答：当前模型的瓶颈在哪一层？" This is the right question and it's well-posed.
  - Open Problem 3 (Build to Delete vs. team skill atrophy) is genuinely insightful and goes beyond typical engineering analysis into organizational dynamics. This is the kind of insight that separates a good
   paper from a great one.
  What's missing:
  - The paper hints at but never develops the co-evolution dynamic: Harness and model evolve together, and the rate of Harness depreciation is itself a function of how aggressively model providers optimize
  for their own Harness. This feedback loop deserves its own section.
  - The paper could strengthen its thesis by connecting to adjacent literature — e.g., the "technical debt" metaphor (Kruchten et al.), or the "software aging" literature (Parnas).
  ---
  Actionability — 7/10 (Practical)
  Strengths:
  - The five principles are concrete and can be operationalized immediately. Principle 1 (最低模型版本标注) and Principle 2 (每发布一个模型版本，跑一次 Harness 审计) are specific enough to be turned into CI
  checks or sprint rituals.
  - The audit checklist in Principle 2 is a usable artifact.
  - The component lifespan table provides a triage heuristic for teams managing large Harness codebases.
  Weaknesses:
  - Principle 4 is the strongest but hardest to operationalize. "移除比保留更需要证据" is a cultural/organizational change, not an engineering practice. The paper needs at least one concrete method for
  implementing this — e.g., a "component retention review" template or a decision protocol.
  - The detection cost problem (Open Problem 1) is acknowledged but no approximation is offered. Even a heuristic — "test the 3 components that touch the model most directly first" — would increase practical
  value.
  - Missing: a worked example. A single end-to-end walkthrough of applying the five principles to one real Harness component would dramatically increase the paper's utility.
  ---
  Key Concerns for Revision
  1. This is a concept/synthesis paper with no empirical validation of its own framework. At minimum, add a small case study applying the five principles to one Harness component and measuring the outcome.
  2. The component lifespan taxonomy needs a decision rule. "Short = model-version-level" is a category, not a method. How does a practitioner determine which tier a new component belongs to?
  3. The "Build to Delete" framing is slightly off. The paper really argues "Build with Explicit Expiry Dates." Consider renaming to "Build to Expire" or "Expiry-First Design" — or at least addressing the
  naming tension explicitly.
  4. Five sources, but all from 2026 and many from the same small community. Broader citation would strengthen the claim that this is a converging consensus rather than a cluster of like-minded practitioners.
  ---
  Questions for Authors
  1. What would falsify the claim that "most Harness code becomes liability at the next model release"? Under what conditions would Harness appreciate rather than depreciate?
  2. Is there a Harness component that gained value after a model upgrade? Counterexamples are as informative as confirming examples for concept papers.
  3. How does this framework apply to pre-training improvements vs. post-training improvements? The paper focuses on post-training (tool-use optimization), but a model with fundamentally better reasoning
  might make different Harness components obsolete.
  ---
  Final Verdict
  Weak Accept. The paper has a genuinely novel framing, strong evidence synthesis, and practical value. The core insight — that Harness components are assumptions about model limitations and therefore have
  expiration dates — is important and under-discussed. However, the lack of empirical validation of the framework itself, the somewhat loose taxonomy, and the absence of a worked example prevent a stronger
  recommendation. A revision addressing the falsification question and adding even a small case study would push this to a clear Accept.
  Best paper potential? No, but it has "most influential in practice" potential — the kind of paper that engineers cite five years later when explaining why they designed their system a certain way.
  ---
  Score: 6.5/10 (borderline accept)
  Recommended venue: ICSE NIER track, or a strong workshop paper elevated to a journal special issue.

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
