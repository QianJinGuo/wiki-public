---
title: "Automating eval design and hillclimbing with Claude (claude-api skill build-eval / hillclimb)"
created: 2026-09-30
updated: 2026-09-30
type: entity
tags: [agent, claude, claude-code, evaluation, harness, skill, context-management, overfitting]
sources: [raw/articles/automating-eval-design-and-hillclimbing-claude-dev]
confidence: 0.7
provenance_state: extracted
---

# Automating eval design and hillclimbing with Claude

## 核心内容

Anthropic（Lance Martin，2026-09-28）系统阐述如何为 AI Agent 设计 eval 并用 hillclimbing 迭代改进而不自欺。配套工具是 claude-api skill 的两个子命令：`/claude-api build-eval`（引导式构建 eval）与 `/claude-api hillclimb`（对既有 eval 做迭代优化）。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md]

## 好 eval 的四要素

1. **评估你真正关心的行为**——eval 任务与生产任务分布对齐，而非代理指标。
2. **性能随模型能力/思考强度单调改善**——若不成立，通常是任务歧义或 grader 校准问题。
3. **前沿有"可提升空间"（headroom）**——最强模型最高 effort 应显著低于 100%，否则无法可靠判断改动影响；判别信号是某任务无论多少 replicates 都失败（不可解任务）。
4. **低 run-to-run 方差**——方差来源包括任务歧义、grader 判定不稳定、effort 配置不一致、环境残留状态（上次 trial 留下的文件/git history 直接把答案递给 agent）。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md]

## Adversarial sampling 陷阱

按"今天的模型失败"选 case = 采样到该模型能力面的 valleys，eval 测的是该模型的 failure fingerprint 而非应用内在难点。正确做法：选 case 前先能说出"为什么这题对人类难"；case 来源优先生产流量、bug reports、tickets，但警惕用户流量偏易（用户倾向尝试他们预期可行的东西）。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md]

## build-eval 工作流

Claude 面试用户后在代码库内构建 eval，样例来源优先级：生产 transcripts → bug reports/tickets → 5-10 个人工 case → 代码库合成 case。Grader 按"够用即可"原则选最便宜的：输出空间受限用程序化校验（exact match/固定标签/JSON schema/测试通过）；开放输出用 LLM-as-judge（rubric 写成可检查 claims 而非 1-5 分；有 baseline 时盲选对比而非打分；judge 模型不得是被测模型）。诊断检查：grader 同输出跑两遍验一致性、检查 timeout/API 错误等 plumbing 噪音、baseline ≥95% 时警告 headroom 不足。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md]

## Hillclimbing 设计原则

- **选可迭代的 surface**：文本类（prompts/skills）便宜可回滚；开放改 harness 代码代价大。
- **可归因（attributable）**：eval 分数变化必须能归因到被修改的 surface（如 skill triggering rate ↔ skill description）。
- **well-scoped objective**：eval 饱和时仍可追求成本目标（性能持平下降成本）。
- **overfitting 防御**：train/test split（train 涨 test 平 = 过拟合警报）；hillclimber 永不把失败 transcript 内容粘进 prompt；答案结构性隔离防 reward hacking。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md]

## hillclimb 工作流

每轮只提一个 patch，目标效应须大于 eval 噪音；从根因修（改产生失败的代码段）而非改措辞。train 涨 test 平 → 疑过拟合回滚；回归 → 回滚；双涨 → 保留。停滞 2-3 轮时进入**根因分诊轮**（不编辑、只把全部剩余失败按原因分桶）——可暴露歧义 case、harness bug、方差问题。结束时停在 test set 最优版本，报告带置信区间；增益在噪音内则建议不合并。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md]

## 实测案例

**成本 hillclimb**（客户支持基准 44 tickets，30 search + 14 holdout）：起点 Opus 4.8 高 effort 74.4% / 4.6¢/ticket → 审计 prompt 删掉强制工具调用仪式/scratchpad/矛盾规则 → Opus 5.5 低 effort 87.8% / 1.9¢ → 降档 Sonnet 5 低 effort 88.9% / 1¢ → 加 routing rules + refund-cap 交叉引用 98.9%；holdout 最终 90.5% vs 原始 78.6%，成本约 1/5。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md]

**性能 hillclimb**（claude-api skill 自身，文档派生 eval）：66% → 补 8 个缺失 feature 覆盖 74% → 修 C#/Java 类型表 77% → 停滞分诊发现"内容在但 Claude 写旧 API 形态"（trained priors），加新旧形态映射表（fixed-budget thinking → adaptive thinking、旧 web search/fetch 工具 → 新版）80% → 修两个坏 grader（grader 要 3 层 catch 但任务只要 1 种；grader 指令与文档矛盾实测文档对）~88%。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md]

## 关键洞察

- **坏 grader 是 eval 配置错误的最常见来源**——相信 evaluator 前必须人工读样本 scored transcripts。
- **任务永不涨分 ≠ 内容缺口**，可能是 example 或 grader 本身有缺陷（文中两例实证）。
- Hillclimber 的"分诊轮"体现反思式 agent 设计：编辑轮与归因轮分离，避免在测量精度以下浪费轮次。
- 与 verifier 驱动开发相关的 eval 工程：分数必须大于噪音才有行动意义。

## 深度分析

**Adversarial sampling 的本质是把模型能力面的 jaggedness 换成了应用无关的噪音**。模型能力在任务间不平滑，若以"今天的模型失败"为选例标准，采样到的会是该模型的 valleys——eval 测出的是该模型的 failure fingerprint 而非应用内在难点。文中给出的反直觉推论是：连用户流量都不能盲信，因为用户倾向尝试他们预期可行的东西，严格取自用户流量的任务分布会系统性偏易。可操作的判据是"加入前先能说出这题对人类为何难"。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:73-77]

**Grader 校准错误比任务设计错误更隐蔽、更高频**。两个实测案例都指向这一点：一个 grader 要求三重 catch 而任务只需一种，另一个 grader 指令与官方文档矛盾而实测文档正确——两者都让"内容已修复"的任务永远不涨分。这与 build-eval 内置的诊断逻辑一致：同一输出跑两遍验一致性、排除 timeout/API 错误等 plumbing 噪音、baseline ≥95% 时警告 headroom 不足。结论：相信任何分数之前，人工读一批 scored transcripts。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:99-112,185-187]

**停滞分诊轮是编辑与归因的行为分离**。每轮只提一个 patch 的常规循环在测量精度以下会浪费轮次；分诊轮不编辑、只把全部剩余失败按根因分桶，因此能暴露三类正常循环发现不了的问题：歧义 case、harness bug、run-to-run 方差。性能 hillclimb 的关键跃迁（66%→~88%）正是发生在分诊轮：发现"内容在但 Claude 写旧 API 形态"（trained priors 覆盖了 skill 内容），解法是新旧形态映射表而非补内容。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:149-160,185-187]

**Overfitting 防御是结构性的，不是提示词层面的**。eval 泄漏进 harness 的形式可以很具体（为需要 OCR 的 eval 任务给应用加上生产中无用的 OCR 工具），三条对策各自切断一条泄漏路径：train/test split 监测"train 涨 test 平"；hillclimber 读失败 transcript 但永不把失败内容粘进 prompt；答案与模型的结构性隔离防 reward hacking。所有这些机制的前提都是同一个门槛——目标效应必须大于 eval 噪音，否则整条反馈回路无意义。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:123-134,148-158]

## 实践启示

- **建 eval 前先做"人类难度"自检**：每个 case 加入前写出一句"为什么这题对人类难"；说不出来的 case 换掉，宁可从生产 transcripts、bug reports、tickets 里补。同时警惕用户流量偏易——刻意补充用户"预期可行所以没试"的区域。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:73-77,85-89]
- **Grader 选最便宜的够用方案，并做三项诊断再信它**：输出空间受限用程序化校验；开放输出用 LLM-as-judge 时把 rubric 写成可检查 claims（非 1-5 分）、有 baseline 就盲选对比、judge 模型不得是被测模型；上线前跑一致性双检 + plumbing 噪音排查 + headroom 检查。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:93-112]
- **Hillclimb 只选便宜、可回滚、可归因的 surface**：文本类（prompts/skills/tool descriptions）优先；每轮一个 patch、从根因修而非改措辞；分数变化必须能归因到被修改的 surface（如 skill triggering rate ↔ skill description）。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:113-118,149-153]
- **把"性能持平降成本"当作 eval 饱和时的默认目标**：baseline ≥95% 时 quality 维度已无 headroom，cost/latency 是仍然可行动的 objective——实测案例中 prompt 审计 + 模型降档把成本压到 1/5 且 holdout 从 78.6% 升到 90.5%。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:111-122,162-176]
- **卡住 2-3 轮就进分诊轮，并接受"问题可能在 eval 自己"**：不编辑、只按根因给剩余失败分桶；某任务无论多少 replicates 都不涨分时，优先怀疑 example 或 grader 有缺陷，而不是继续堆内容。^[raw/articles/automating-eval-design-and-hillclimbing-claude-dev.md:156-160,185-187]

## 关联

- → [[raw/articles/automating-eval-design-and-hillclimbing-claude-dev|原文存档]]
- [[concepts/verifier-driven-development|Verifier 驱动开发]]
- [[entities/claude-code-skill-writing-guide|Claude Code Skill 编写指南]]
