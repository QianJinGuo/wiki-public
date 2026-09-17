---

title: "The State of the AI Economy — $110B Revenue, Bottom-Up Deduplicated Model"
created: 2026-06-27
updated: 2026-09-17
type: entity
tags: [ai-economy, market-analysis, data-driven, exponential-view, azeem-azhar]
source: [[raw/articles/the-state-of-the-ai-economy]]
sources: [raw/articles/the-state-of-the-ai-economy]
description: "Azeem Azhar (Exponential View) first bottom-up, deduplicated AI economy model: $110B annual revenue, $175B annualized run rate, covering consumer + enterprise full-stack spending."
review_value: 9
review_confidence: 10
review_recommendation: strong
review_stars: 5
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# The State of the AI Economy — $110B Revenue, Bottom-Up Deduplicated Model

> **Background**: Based on Exponential View's 2026-06-25 inaugural AI Economy report, using bottom-up, deduplicated financial modeling covering consumer and enterprise AI spending across the full stack.

## Core Numbers

- **Past 12 months AI revenue**: $110B (deduplicated)
- **Annualized run rate**: $175B
- **Methodology**: Bottom-up per-company P&L modeling, cross-validated, no double-counting
- **Coverage**: Consumer + enterprise AI spending full stack (excluding internal AI uplift, professional services)

## Methodology Innovation

Traditional AI market sizing has a severe **double-counting** problem: user pays Anthropic $1, Anthropic pays AWS $0.5, traditional methods report $1.5. Exponential View reports only **end-user actual spend** of $1. ^[raw/articles/the-state-of-the-ai-economy.md]

Approach:
1. Build item-by-item financial models (P&L, balance sheet, cash flow) for largest contributing companies and business units
2. Cross-validate using high-confidence public statements, supplier data, customer feedback
3. Assign confidence weights to leaked information and self-reported data
4. All numbers are auditable — traceable to specific data points with confidence weights

## Supply Side vs Demand Side

**Supply side** (well-understood):
- Chips, memory, power transformers, cooling, data center components
- Mostly public companies, trackable via disclosures, sales, forward order books

**Demand side** (much harder):
- OpenAI, Anthropic, Cursor, ElevenLabs etc. are private — no disclosure obligations
- Hyperscalers (AWS/GCP/Azure) inconsistently disclose AI segment revenue
- Requires piecing together public statements, leaks, self-reports

## Exclusions

- Internal AI uplift (e.g., recommendation system improvements increasing ad revenue)
- Efficiency savings from big tech internal tools
- Professional services and systems integration

## 深度分析

### Bottom-Up Deduplication: What It Proves and What It Cannot

Each end-customer dollar is counted once — a $1 Claude payment that routes 50 cents onward to AWS is reported as $1 ^[raw/articles/the-state-of-the-ai-economy.md:34] — giving a floor on end-customer vendor spend with a provenance chain, not precision: deduplication removes double counting, not estimation. Supply-side lines rest on public disclosures and forward order books; demand-side lines are rebuilt from statements, leaks and self-reports, since OpenAI, Anthropic, Cursor and ElevenLabs need not disclose and hyperscalers do not break out AI revenue consistently ^[raw/articles/the-state-of-the-ai-economy.md:36-38]. Residual error is widest where growth is fastest, so treat $110B as an order-of-magnitude anchor, not a precise time series.

### Decomposing the Number: Which Components Are Estimates

$110B is trailing-twelve-month revenue after deduplication; $175B annualizes the most recent month ^[raw/articles/the-state-of-the-ai-economy.md:54] — a 12x extrapolation, so seasonality or one large enterprise contract moves the headline, making that gap the report's most fragile component. The trajectory is the claim: revenue grows roughly three times faster than the mobile or Internet waves ^[raw/articles/the-state-of-the-ai-economy.md:58] — a growth-rate statement, never a size comparison. Deduplication also fixes the market's shape: chip and cooling revenue cannot be added back, and executive intent to invest more heavily is sentiment, not money.

### What the Exclusions Imply

Four exclusions bound the figure: internal AI uplift (recommendation systems lifting ad revenue), internal efficiency savings, professional services and systems integration, and China, modeled but omitted from v1. Services matter most because the omission is asymmetric: a Fortune 500's AI investment only partly reaches AI companies, with a large share paying for implementation support ^[raw/articles/the-state-of-the-ai-economy.md:50], so enterprise AI budgets reported elsewhere include a layer this model excludes. All four point one way: $110B is a floor for vendor revenue and a larger understatement of AI's economic value. Do not add the pieces back — internal-uplift models exist but are deliberately unreported ^[raw/articles/the-state-of-the-ai-economy.md:52].

### Convergence and Tension with the Nadella and Amodei Entities

This model and [[entities/nadella-token-capital-microsoft-ai-economy-2026|Nadella Token Capital]] converge on one test. Token capital calls AI spend capital formation but needs a measuring instrument; Exponential View supplies it by separating AI-oriented CapEx from ordinary CapEx — hyperscalers already spent around $120B a year before ChatGPT — then depreciating compute over six years and other infrastructure over fourteen, finding revenue that "just about clears" depreciation ^[raw/articles/the-state-of-the-ai-economy.md:68]. Both rest on measured elasticity: a 10% price cut yields 12-18% more tokens ^[raw/articles/the-state-of-the-ai-economy.md:74]. It refines the wiki's token-economy pages ([[entities/the-token-economy-pt2-the-intelligence-company-gets-built|Token Economy Pt. 2]], [[entities/anthropic_cache_tokenomics|Claude cache tokenomics]]) by treating tokens as a billing metric, with quality-adjusted output tokens the better quotient ^[raw/articles/the-state-of-the-ai-economy.md:76].

The tension is with [[entities/dario-amodei-policy-ai-exponential-2026|Dario Amodei Policy]], which treats capability trajectories and policy response as first-order. This model admits capability only through price-quality elasticity and assumes compute supply and falling prices persist — the six-year life is defended on demand still exceeding available compute ^[raw/articles/the-state-of-the-ai-economy.md:72]. A policy throttle, export control or power ceiling has no variable here, and Amodei-style exponential impact would land off vendor P&Ls, in the uplift and savings this count excludes ^[raw/articles/the-state-of-the-ai-economy.md:82].

### Capex-versus-Revenue Arithmetic and Its Falsifiers

Ordinary hyperscaler CapEx ran near $120B a year before ChatGPT, including logistics and excluding Meta ^[raw/articles/the-state-of-the-ai-economy.md:94]; only the incremental AI-oriented layer is charged against AI revenue and depreciated over six years for compute and fourteen for other infrastructure ^[raw/articles/the-state-of-the-ai-economy.md:68]. That "just about clears" verdict is fragile: depreciation is non-cash while capex is paid upfront; the six-year life is a judgment call ^[raw/articles/the-state-of-the-ai-economy.md:72], so shorten it toward the observed service life in [[entities/seangoedecke-ai-gpus-live-longer-than-three-years-2026|GPU useful life]] and the margin disappears; and the test covers hyperscalers only, not the private labs carrying demand-side risk. Value is counted at the end customer ^[raw/articles/the-state-of-the-ai-economy.md:34], so supply-chain revenue is not a claim on the $110B, and capital recycling from investors into labs into clouds into suppliers counts once as spend, never as return — see [[entities/nvidia-embraces-ai-investor-topping-40-billion-in-equity-bets-2026|Nvidia's equity bets]] and [[entities/napkin-inference-cost-injuly-2026|inference cost curves]]. Falsifiers next quarter: a run rate failing to extend $110B → $175B, AI segment disclosures below these estimates, shortened depreciation lives, elasticity below 12-18%, or a lab's leaked revenue restated down.

## 实践启示

1. **Quote the unit, not just the number.** $110B is deduplicated *end-customer* spend over twelve months ^[raw/articles/the-state-of-the-ai-economy.md:34]; never add chip, memory or cloud revenue to it or compare it line-for-line with supply-side market sizing.
2. **Use the level as a floor, the growth rate as the claim.** $175B annualizes a single month ^[raw/articles/the-state-of-the-ai-economy.md:54] — require two or three quarters of confirmation, and cite the three-times-mobile figure as a growth-rate result only ^[raw/articles/the-state-of-the-ai-economy.md:58].
3. **Re-run the depreciation test before citing it.** Recompute against a three-to-four-year compute life ^[raw/articles/the-state-of-the-ai-economy.md:68-72]: if "just about clears" flips, the schedule was doing the work — compare [[entities/seangoedecke-ai-gpus-live-longer-than-three-years-2026|GPU useful life]].
4. **Do not use this figure as enterprise TAM.** Services and systems integration are excluded, so a Fortune 500's AI commitment is not a pipeline against $110B ^[raw/articles/the-state-of-the-ai-economy.md:50]; pair it with [[concepts/enterprise-ai-adoption|enterprise AI adoption]] evidence instead.
5. **Locate your position on the deduplication boundary.** Value is counted at the end customer, so upstream revenue shows up only as someone else's cost — the operative point for [[concepts/cloud-ai-infrastructure|cloud AI infrastructure]], [[concepts/ai-cost-optimization-framework|AI cost optimization]] and any thesis resting on [[concepts/context-window-economics|token price economics]].
6. **Treat the exclusions as the honest error budget.** Internal uplift, efficiency savings and China are omitted and all point one way — understatement. Say so whenever the number is used; never present $110B as AI's total economic value.

## Differentiation from Existing Wiki Entities

| Dimension | This entity (Exponential View) | Nadella Token Capital | Dario Amodei Policy |
|-----------|-------------------------------|----------------------|---------------------|
| Angle | Data-driven market sizing | CEO enterprise strategy | AI policy/safety |
| Core contribution | $110B deduplicated revenue model | Token capital dual framework | Exponential growth policy response |
| Methodology | Bottom-up P&L modeling | Strategic vision | Policy analysis |
| Actionability | Investment/market judgment | Enterprise architecture decisions | Regulatory/compliance |

→ [[raw/articles/the-state-of-the-ai-economy|source archive]] ^[raw/articles/the-state-of-the-ai-economy.md]
→ [[entities/nadella-token-capital-microsoft-ai-economy-2026|Nadella Token Capital]] ^[raw/articles/the-state-of-the-ai-economy.md]
→ [[entities/dario-amodei-policy-ai-exponential-2026|Dario Amodei Policy]] ^[raw/articles/the-state-of-the-ai-economy.md]
