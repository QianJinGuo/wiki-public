---
title: "Harness Engineering 判断地图：7 条可复用判断"
created: 2026-05-15
updated: 2026-11-25
type: query
tags: [harness, agent, synthesis, decision-map, production, evaluation]
sources:
  - raw/articles/harness-engineering-framework
  - raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09
  - raw/articles/claude-code-之父最新访谈编程已经结束harness-将消失claude-code-将只有-100-行代码loop-才是未来
  - raw/articles/cat-wu-anthropic-pm-interview
  - raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16
  - raw/articles/karpathy-vibe-coding-agentic-engineering-v4
  - raw/articles/luo-fuli-ai-paradigm-shift
  - raw/articles/harnesseval-evaluation-harness-mirros-jiqizhixin-2026-08-18
confidence: 0.78
provenance_state: merged
---

# Harness Engineering 判断地图：7 条可复用判断

> 这不是又一篇 Harness 摘要。它把库内已有材料压成 7 条能直接调用的判断；每条都写明机制、证据、边界和动作。遇到新模型或新框架，先改这 7 条，不要先加第 N 篇摘要。

## 先看结论

| ID | 当前判断 | 什么时候用 | 置信度 |
|---|---|---|---|
| H1 | Harness 决定“能不能稳定交付”，模型决定“上限” | 判断问题该怪模型还是怪系统 | 高 |
| H2 | 模型升级会吃掉能力补偿层，但会加厚治理验证层 | 换模型、删组件、做年度重构 | 中高 |
| H3 | 长程任务的瓶颈是上下文与状态治理，不是 prompt 长度 | 任务变长、开始丢约束、开始重复劳动 | 高 |
| H4 | Generator 和 Evaluator 必须分离，验收要看环境 | 设计反馈环、上线验收、做 eval | 高 |
| H5 | Harness 有适用门槛，重 Harness 不是默认答案 | 新任务选型、POC 是否继续投入 | 中高 |
| H6 | Harness 分数不能脱离模型、任务、工具、预算单独比较 | 看到“提升 X 分”的厂商/论文数字 | 高 |
| H7 | Harness 的默认终态是删除，不是累积 | 模型升级、组件过期、架构重构 | 高 |

## H1. Harness 决定“能不能稳定交付”，模型决定“上限”

**判断**：同一个模型能否稳定完成任务，主要取决于模型之外的状态、工具、约束和验收系统；模型只提供能力上限，Harness 决定这份能力能否被稳定调用。^[raw/articles/harness-engineering-framework.md, raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**机制**：模型产生的是动作候选；Harness 负责把候选变成可执行步骤，并在失败时提供观测、重试、回退和终止。没有这层，模型越强，错误也可能越大，因为行动空间变大了而责任边界没有变大。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**证据**：Terminal Bench 2.0 中底层模型不变，仅改造 Harness，分数从 52.8 升到 66.5；货拉拉的生产复盘把 text2SQL Agent 的上下文爆炸、超时、幻觉和权限问题归因于 Harness，而不是继续优化提示词。^[raw/articles/harness-engineering-framework.md, raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**边界**：短任务、一次性任务、确定性 API 就能完成的任务，不应该先上 Harness；这时 Harness 增加的是延迟和维护成本，不是可靠性。

**动作**：故障归因时先问一句：这是“模型不会”，还是“系统没接住”？只有后者才进入 Harness 建设。

## H2. 模型升级会吃掉能力补偿层，但会加厚治理验证层

**判断**：Harness 组件要分两层看。补模型当前短板的“能力补偿层”——提示词补丁、强制步骤、防错外壳——会随模型升级贬值；补责任、权限、审计、回滚和领域知识的“治理验证层”会保留，甚至变厚。^[raw/articles/claude-code-之父最新访谈编程已经结束harness-将消失claude-code-将只有-100-行代码loop-才是未来.md, raw/articles/cat-wu-anthropic-pm-interview.md, raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md, raw/articles/karpathy-vibe-coding-agentic-engineering-v4.md]

**机制**：能力补偿层的收益来自“模型还不会”；治理验证层的收益来自“错误不可见、不可回滚、不可追责”。前者会随模型能力上升而下降，后者会随行动空间和风险上升而上升。^[raw/articles/claude-code-之父最新访谈编程已经结束harness-将消失claude-code-将只有-100-行代码loop-才是未来.md, raw/articles/cat-wu-anthropic-pm-interview.md, raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md, raw/articles/karpathy-vibe-coding-agentic-engineering-v4.md]

**证据**：Boris Cherny 和 Cat Wu 判断产品外壳会缩小；黄仁勋、Karpathy 和罗福莉则强调企业 Harness、verifier 和框架放大能力。表面冲突，实际是同一个词指了两层。^[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md, raw/articles/karpathy-vibe-coding-agentic-engineering-v4.md, raw/articles/luo-fuli-ai-paradigm-shift.md]

**边界**：这些是访谈中的方向判断，不是同任务跨模型对照实验；“100 行”指产品外壳，不是整个 Agent runtime；企业级命题不能直接套到个人项目。^[raw/articles/claude-code-之父最新访谈编程已经结束harness-将消失claude-code-将只有-100-行代码loop-才是未来.md, raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16.md]

**动作**：每个组件标注“补能力 / 补责任”。补能力的设删除日期；补责任的设 owner、审计口径和回滚路径。换模型时逐项重判，不要整体保留。

## H3. 长程任务的瓶颈是上下文与状态治理，不是 prompt 长度

**判断**：任务一长，失败主要来自关键信息被稀释、状态丢失和错误累积，而不是指令写得不够多。^[raw/articles/harness-engineering-framework.md]

**机制**：上下文被无关内容占满后，信号噪音比恶化，模型决策质量会系统性下降，这就是 Context Rot。有效做法是把上下文分层：常驻身份与约束、按需加载的技能、运行时注入、跨会话记忆、确定性系统输出；同时把进度和中间状态写到模型外部。^[raw/articles/harness-engineering-framework.md, raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**证据**：货拉拉把“管理 I/O 与上下文”“状态与记忆持久化”列为生产四支柱；其 Agent 在上下文爆炸和任务超时后，才开始做分层降维和状态外置。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**边界**：单轮问答和短任务不需要复杂分层；分层本身有维护成本，层数过多会让系统更难调试。

**动作**：先做三件事——限制 working set、把进度写到磁盘、把可复用事实放进外部记忆；再考虑压缩、摘要和 Context Reset。

## H4. Generator 和 Evaluator 必须分离，验收要看环境

**判断**：让生成者评价自己，会系统性高估；生产验收必须由独立 Evaluator 执行环境验证或确定性工具，而不是读 Generator 的自我报告。^[raw/articles/harness-engineering-framework.md]

**机制**：Generator 有为自己输出辩护、放大成功面、淡化失败面的倾向；Evaluator 直接操作页面、跑测试、调编译器，才能把“看起来对”变成“实际可用”。^[raw/articles/harness-engineering-framework.md]

**证据**：Anthropic 实践用 Playwright 实际访问修改后的页面，而不是让模型读代码打分；反馈环稳定还需要三个条件：验证结果可操作、环内信息不腐化、最大循环次数有上限。^[raw/articles/harness-engineering-framework.md]

**边界**：低风险、可逆、低成本任务可以轻量验证；独立 Evaluator 不是无限循环，必须有终止条件和人工接管点。

**动作**：至少配一个确定性 verifier（测试 / lint / 类型 / schema）和一个环境检查；给重试设上限，给失败留可读的错误信息。

## H5. Harness 有适用门槛：先确定性 API，再单次调用，再薄 Agent，最后才是重 Harness

**判断**：不是所有任务都值得上 Harness。一次性脚本、短任务、低失败代价场景，重 Harness 往往是负收益；先找更便宜的确定性替代，再考虑 Agent 和 Harness。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**机制**：Harness 的成本来自维护、延迟、上下文、权限和调试复杂度；只有任务会被重复执行、失败代价高、且模型能力确实不够时，这些成本才摊得平。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**证据**：货拉拉明确列出三个不需要 Harness 的场景：短期少量单轮任务、预算充足直接上最强模型、核心价值直接建立在推理/生成上的小团队。^[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**边界**：高风险、强合规、长程、重复执行的任务仍然需要重 Harness；门槛判断不是“越轻越好”。库内负结果见 [[queries/negative-results-registry|负结果登记簿]]。

**动作**：上 Harness 前四问——当前模型真的还有这个短板吗？有没有更便宜的确定性接口？任务复用频次撑得起维护成本吗？删除条件现在就写了吗？

## H6. Harness 分数不能脱离模型、任务、工具、预算单独比较

**判断**：看到“改造 Harness 提升 X 分”，先问基线是什么；不同模型、任务集、工具权限、token/时间预算和验收器下的数字，不能直接移植。^[raw/articles/harness-engineering-framework.md, raw/articles/harnesseval-evaluation-harness-mirros-jiqizhixin-2026-08-18.md]

**机制**：Harness 改变的是搜索空间和验证信号；这些变量一起变，分数无法归因。一个 Harness 在 A 模型上是补短板，在 B 模型上可能只是增加摩擦。^[raw/articles/harness-engineering-framework.md]

**证据**：Terminal Bench 的 52.8→66.5 是同一基准、同一模型下的改造结果，不能外推成“任何任务都能提升 13.7 分”；HarnessEval 则把评测本身也视为需要 Plan / Route / Decompose / Verify 的 Harness。^[raw/articles/harness-engineering-framework.md, raw/articles/harnesseval-evaluation-harness-mirros-jiqizhixin-2026-08-18.md]

**边界**：方向性结论可以迁移，具体幅度不能迁移；跨厂商 benchmark 还要额外检查任务污染、提示泄漏和评分器差异。

**动作**：做 Harness A/B 时冻结模型、任务集、工具权限、预算和 verifier；报告绝对值、方差和失败类型，不只报一个提升百分比。

## H7. Harness 的默认终态是删除，不是累积

**判断**：每个组件都编码了“当前模型还不行”的假设；模型升级后不删除，Harness 会从能力放大器变成摩擦源。^[raw/articles/harness-engineering-framework.md, raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**机制**：组件会占用上下文、增加权限复杂度、制造新的错误路径和维护负担；能力补偿层过期后，这些成本不再换来可靠性。^[raw/articles/harness-engineering-framework.md]

**证据**：Harness Engineering 框架记录组件会随模型代际衰减，主线实践也反复重构和删除组件；库内把这一现象归入“代际衰减”负结果。^[raw/articles/harness-engineering-framework.md, raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09.md]

**边界**：治理、安全、审计和回滚组件不能因为模型变强就删除，除非责任边界本身消失；删除的是能力补偿，不是责任。

**动作**：每个组件写“删除条件 + 复查日期”；每次模型升级做一次 remove-one test——一次只移除一个组件，观察质量、成本和失败率。

## 最小上线闭环

- [ ] **动作空间**：工具白名单、危险操作确认、权限分级、重试熔断。
- [ ] **上下文**：常驻层 / 按需加载 / 运行时 / 记忆 / 系统输出分层，设置 working set 上限。
- [ ] **状态**：进度、决策和中间结果写到模型外部，支持中断恢复。
- [ ] **验收**：独立 Evaluator + 至少一个确定性 verifier，验收看环境而不是自我报告。
- [ ] **观测**：轨迹、成本、失败类型、重试次数可回放。
- [ ] **退出**：每个组件有 owner、删除条件和复查日期。

这套闭环对应 [[queries/harness-minimum-checklist|Harness 最小检查表]]；后者偏启动与迭代清单，本页偏“什么时候信、什么时候删”的判断。

## 证据强度与边界

- **高置信**：H1、H3、H4、H6、H7 有一手实践或明确机制支撑，但具体数字仍需按任务复核。
- **中高置信**：H2 来自多期访谈的交叉判断，方向清楚，但不是同任务对照实验。
- **中置信**：H5 来自单一生产团队的边界条件，适合作为选型门，不是普适定律。
- **已消解的冲突**：H2 的“变薄 vs 变厚”不是真假冲突，而是“能力补偿层”和“治理验证层”被同一个词覆盖。
- **会改变判断的证据**：跨模型 Harness 迁移实验、独立复现的 benchmark、生产事故复盘、成本结构变化。

## 怎么用

| 场景 | 先看 | 产出 |
|---|---|---|
| 新任务要不要上 Harness | H5 | 一页选型判断：确定性 API / 单次调用 / 薄 Agent / 重 Harness |
| 换模型 | H2 + H7 | 组件删留表：补能力 / 补责任 / 删除日期 |
| 任务开始变长 | H3 | 上下文分层 + 外部状态 + 恢复点 |
| 设计验收 | H4 + H6 | 独立 Evaluator + 固定变量的 A/B |
| 看到厂商数字 | H6 | 基线、任务集、预算、评分器四问 |

## 关联

- [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- [[concepts/agentic-engineering-paradigm|agentic engineering 工程范式]]
- [[concepts/when-not-to-harness-engineering|何时不要 Harness Engineering]]
- [[concepts/evaluation-harness-design|评估 Harness 设计]]
- [[queries/harness-minimum-checklist|Harness 最小检查表]]
- [[queries/podcast-knowledge-deconstruction|播客知识拆解：从单期摘要到跨期判断]]
- [[queries/negative-results-registry|负结果登记簿]]
- [[queries/agent-evaluation-harness-best-practices|Agent Evaluation Harness 的最佳实践]]

## 主要原文

- [[raw/articles/harness-engineering-framework|Harness Engineering 框架原文]]
- [[raw/articles/huolala-harness-engineering-stable-agent-practice-2026-11-09|货拉拉 Harness 生产实践]]
- [[raw/articles/claude-code-之父最新访谈编程已经结束harness-将消失claude-code-将只有-100-行代码loop-才是未来|Boris Cherny：Harness 将消失]]
- [[raw/articles/cat-wu-anthropic-pm-interview|Cat Wu：Claude Code / Cowork 访谈]]
- [[raw/articles/jensen-huang-harrison-chase-harness-future-2026-07-16|黄仁勋 × Harrison Chase：Harness 的未来]]
- [[raw/articles/karpathy-vibe-coding-agentic-engineering-v4|Karpathy：从 Vibe Coding 到 Agentic Engineering]]
- [[raw/articles/luo-fuli-ai-paradigm-shift|罗福莉：AI 范式巨变]]
- [[raw/articles/harnesseval-evaluation-harness-mirros-jiqizhixin-2026-08-18|HarnessEval：评估 Harness 设计]]

