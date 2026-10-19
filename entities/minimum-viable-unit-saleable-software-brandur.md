---

title: "The Minimum Viable Unit of Saleable Software"
description: "Brandur 分析 LLM 对软件经济的影响：最小可售单元的重新定义"
source: "[[raw/articles/minimum-viable-unit-saleable-software-brandur]]"
tags:
  - software-economics
  - llm-impact
  - pricing
  - indie-dev
  - ai-native
created: 2026-06-22
updated: 2026-10-20
type: entity
review_value: 8
review_confidence: 8
review_recommendation: worth-reading
review_stars: 4
sources:
  - raw/articles/minimum-viable-unit-saleable-software-brandur
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# The Minimum Viable Unit of Saleable Software

> 原文存档：[[raw/articles/minimum-viable-unit-saleable-software-brandur|原文存档]]

## 摘要

Brandur 用一则轶事——某团队为省 $400/mo 的 Jira 账单，用 Claude 自建了任务追踪器——把"AI 让软件变便宜"拆成一道可算的题。结论：LLM 压低了自建成本但没压到零，每个软件品类因此存在一个"可行性区间"（zone of viability），其下沿即**最小可售单元**（minimum viable unit of saleable software）。低于这条线，客户自建比购买更划算，产品在结构上就卖不出去。

## 核心要点

- LLM 让 build 变便宜但没变免费：真正的成本是人在回路里的监督、验证与维护劳动 ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:23-25]
- $200k/年、每周 40 小时的工程师约合 $96/小时；省下 $400/mo 只够换每月 4 小时维护预算 ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:29-47]
- 即便乐观地把维护压到 2 小时/月，自建 Jira 仍需 37 个月才回本 ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:47]
- 定价越高自建诱惑越大：50 个 Salesforce 席位 ≈ $25k/mo，约合 1.5 个全职工程师 ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:53-55]
- 可行性区间两个边界：足够的 novelty 使 LLM 重写非平凡，且定价没贵到强烈鼓励重写 ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:67-69]
- 区间内的判据是一条不等式：累计许可费 < 自建一次性投入 + 持续维护成本 ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:71]
- River 的赌注：开源铺广度，Pro 版把 API 设计与性能特性做成"重写也不易复现"的 novelty ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:86]

## 深度分析

### "Saleable" 与 "useful" 的分界

The distinction here is economic, not technical. "Useful" asks whether the software does something worth doing; "saleable" asks whether the buyer's cheapest alternative is *worse* than paying you. Historically they coincided: building was expensive enough that anything useful cleared the bar. LLMs decouple them: usefulness becomes cheap and abundant, so the price floor is no longer set by the value you deliver but by the cost your buyer avoids.

The Jira anecdote is the demonstration. The homegrown tracker was almost certainly useful — tooled to their exact workflow — and still not a sensible purchase. Brandur's definition makes the framing explicit: below the minimum viable unit, "a rebuild is the same or less effort compared to going through the purchasing process for a third party" ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:71-73]. That formulation quietly includes procurement friction — purchasing, security review, integration, vendor risk — the one asymmetry working in the vendor's favour, and exactly what vanishes when the alternative is a package one engineer can prompt into existence.

### 边际成本趋零如何打碎按席位/按调用的定价直觉

Software's marginal cost was already near zero; LLMs push the *first copy* cost toward zero. That matters because per-seat pricing was never cost-plus. The seat was a proxy for "how much would this cost to replace." When replacement cost falls by an order of magnitude, the seat loses its defence from both ends at once: the vendor's production cost is trivial, and the buyer's avoidance value is trivial too. What survives is only what the buyer cannot reproduce — on-call rotation, roadmap, accumulated edge cases, operational confidence.

Seats also mis-measure value in a way LLMs make worse: value is increasingly not per-human. A tracker serving 50 people may generate the same operational value as one serving 5, and an agent consuming an API produces load without a seat at all. Brandur's table is the numeric version: the same buyer rationally *buys* Jira at $400/mo and *builds* Salesforce at $25k/mo ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:75-78]. Same customer, opposite decisions, because the engineer-months each bill converts into differ completely. Your price is only defensible relative to the engineering capacity it can be exchanged for.

### 可行性区间的两条边界

Both edges are self-inflicted failure modes rather than competitive ones. Too cheap and the vendor cannot staff the maintenance loop that keeps the novelty alive — and a price far below the buyer's build cost is itself a signal that the thing is trivial, an advertisement for the rebuild you meant to prevent. Too expensive and the licensing bill converts pound for pound into headcount: the $25k/mo Salesforce case is a build decision dressed as a purchase, and the market seemed to agree ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:53-55].

Between them the decision variable is payback period, not sticker price. The 37-month breakeven is the shape of the whole argument — the customer is amortising, and a payback beyond their planning horizon reads as "never." The band also moves: up with engineer salaries, narrowing from below as models improve, outward with switching and re-integration costs.

### AI 产品把计价单元从席位推向结果/Token/任务

Two failures arrive together. Cost is per-token while revenue is per-seat, so a single agentic workload can consume a seat's entire margin while a light user subsidises it — revenue decouples from cost exactly where variance is highest. The unit therefore migrates to tokens, tasks, or outcomes, the only units tracking what is consumed and delivered.

Less discussed is the second: passing infrastructure cost through directly imports model-pricing volatility into the customer's invoice, and makes your price legible as a markup on something the buyer can also buy. If the bill reads "model cost × 2," you have itemised the build-side arithmetic for them and the zone of viability collapses. The defensible move is pricing an outcome whose value is not reconstructible from a token count — precisely the novelty Brandur claims for River's Pro tier, where the moat is API design and performance properties rather than feature coverage ^[raw/articles/minimum-viable-unit-saleable-software-brandur.md:84-86]. Metering granularity is thus a product decision, not a billing detail.

## 实践启示

1. 先算客户的回本周期，再算自己的毛利：把"几周自建 + 每月几小时维护"写进定价推导，payback 超过约 24 个月等于主动邀请重写。
2. 把 novelty 建在难复现的层：API 设计、性能特性、运维默认值、迁移成本——而非功能清单，功能清单最容易被 prompting 复现。
3. 定价要覆盖"人在回路"的真实成本：验证与维护是 LLM 时代唯一没被压价的环节，别把毛利建立在免费的人类注意力上。
4. 计价单元与基础设施成本解耦：按任务/结果收费，避免按 token 加成，否则等于把客户自建预算的上界写在账单上。
5. 用组合视角看产品线：同一客户会对 A 品类买、对 B 品类建，不要在客户必然自建的品类里做低价防守。
6. 低价既是收入问题也是信号问题——它在告诉客户"这东西 LLM 半小时能写完"。

## 相关实体

- [[entities/enterprise-software-moats-agent-era|Enterprise Software Moats in the Agent Era]] — AI 时代软件护城河的框架化讨论，可与本文的 novelty 边界互读
- [[entities/posts-repricing-of-software-engineering-labor|Repricing of Software Engineering Labor]] — 本文 $96/小时的换算，是工程劳动重新定价的微观样本
- [[entities/ai-true-moat-not-llm-but-organization|AI 的真护城河不是 LLM 而是组织]] — 与"novelty 须长在难复现层"互补的组织侧论证

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
