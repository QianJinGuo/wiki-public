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

## 关联

- → [[raw/articles/automating-eval-design-and-hillclimbing-claude-dev|原文存档]]
- [[concepts/verifier-driven-development|Verifier 驱动开发]]
- [[entities/claude-code-skill-writing-guide|Claude Code Skill 编写指南]]
