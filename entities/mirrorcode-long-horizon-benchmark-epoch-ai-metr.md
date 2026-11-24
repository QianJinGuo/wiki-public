---
title: "MirrorCode: AI 从行为重建完整程序的长时程基准（Epoch AI × METR）"
created: 2026-08-05
updated: 2026-11-15
type: entity
tags: [benchmark, long-horizon, coding-agent, evaluation, agent]
sources: [raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# MirrorCode: AI 从行为重建完整程序的长时程基准（Epoch AI × METR）

## 核心定义

MirrorCode 是 Epoch AI 与 METR 联合开发的软件工程基准，用于测试 AI 模型在**长时程编码任务**上的能力：模型被要求**端到端重新实现整个程序**，且无法访问原始源代码——只能根据程序行为推断实现，AI 生成的解决方案必须在端到端测试（含 held-out 测试）上与原始程序输出完全一致。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

25 个目标程序覆盖 Unix 工具、数据序列化与查询工具、生物信息学、解释器、静态分析、密码学与压缩等计算领域。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

## 与现有 SWE 基准的关键差异

大多数软件工程基准（如 SWE-bench）聚焦短任务——修 bug 或实现单个 feature，且推理预算通常限制在 $1–10。MirrorCode 提供**足够大的推理预算**让模型认真尝试：最大的任务单次运行花费 $2,600，AI 无人工干预连续工作 19 天。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

- **任务形态**：从行为重建完整程序（black-box reimplementation），而非补丁式修复
- **预算规模**：单任务可达 $2,600 / 19 天；正式 leaderboard 每任务 100 亿 token + 7 天预算，每任务跑 3 次
- **测试设计**：模型永远看不到端到端测试，防止构建 lookup table 伪造输出
- **沙箱约束**：无网络、无原始代码库、无作弊路径

## 实测结果：Claude Opus 4.7 重建 gotree

Claude Opus 4.7 成功重新实现了 gotree——约 16,000 行 Go 代码、40+ 命令的生物信息学工具包，耗时 14 小时、成本 $251。人类工程师无 AI 辅助完成同一任务预计需要 2–17 周。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

最佳 AI gotree 实现通过了 2000/2001 个测试——仅在一个处理日期标注的边缘命令上失败，被视为近乎完美的重建。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

## 数据污染（memorization）控制

由于任务涉及重建开源程序，模型预训练中很可能见过原始代码库，可能虚增分数。MirrorCode 内置 **memorization screen**：AI 成功重建了通过该筛查的目标程序，而未能重建筛查显示存在记忆证据的程序——表明结果并非主要由记忆驱动，但无法完全排除记忆的贡献。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

## 开放资源与评估协议

- 开放 scaffold + 25 个目标程序中的 22 个（共 132 个任务实例，覆盖 6 种编程语言），其余 3 个目标作为私有测试集保留
- 正式 leaderboard 固定任务集：Medium + Large 桶的 15 个目标程序，每个程序用两种语言（通常 Go + Ada）评估，共 30 个任务
- 论文：arXiv 2606.30182，作者 Tom Adamczewski / David Owen / David Rein 等

## 与 Wiki 现有概念的关系

MirrorCode 填补了 Agent 评估基准 中长时程维度的空白——现有 LLM 基准全景 以短任务为主，而 MirrorCode 把「重建整个程序」变成可评测任务，与 [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 长时程实践]] 互补。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

其推理预算设计（$2,600 / 19 天）与 [[concepts/long-running-agent-architecture|长时程 Agent 架构]] 直接呼应——评测预算必须匹配任务的真实难度，否则短预算只会测出「短时程能力」。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

同时它也是 [[entities/cursor-reward-hacking-coding-benchmarks|编码基准 reward hacking]] 讨论的反面样本——通过 held-out 测试 + 沙箱 + memorization screen 三重设计降低作弊面。^[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr.md]

## 深度分析

### 任务分桶与两套评估协议的张力

MirrorCode 将 25 个目标程序按规模分成 Small / Medium / Large 三个桶，但论文协议与 leaderboard 协议并不相同：论文覆盖全部 25 个目标，在 Small 与 Medium 桶上使用全部 6 种 agent 实现语言，除 Large 目标外单次尝试预算为 10 亿 token 且不限时；而 leaderboard 为了跨时间可比性收敛到固定任务集——Medium 与 Large 桶的 15 个目标，每个程序用两种实现语言（通常 Go + Ada，即一门主流语言配一门低资源语言），得到 30 个任务，每任务跑 3 次，预算提高到每尝试 100 亿 token 加 7 天。两套数字不可直接比较，引用分数时必须注明协议版本。值得注意的是，正式榜单上「+Private」计法会把私有测试集纳入统计，且 Go + Ada 的 2L 映射刻意让一半任务落在低资源语言上——主流语言的高分不能掩盖模型在低资源生态上的短板。「论文探索广度、榜单固定深度」的分工本身就是一个可借鉴的基准运维模式。

### 三重防作弊面：行为对齐而非输出模仿

MirrorCode 的评估设计可拆成三层：其一，模型在开发期间永远看不到端到端测试（含 held-out 测试），因此无法构造 lookup table 逐条模仿原始程序的输出；其二，沙箱隔离——无网络、无原始代码库、无作弊路径，模型只能从可观察的行为推断内部实现；其三，memorization screen 针对「预训练见过原代码」这一污染路径单独设卡。三层分别封堵伪装输出、环境作弊与记忆泄题，共同保证分数反映的是「从行为重建程序」的真实能力，而非信息泄露的副产物。

### 数据污染：相关性证据而非确定性结论

污染控制的证据结构值得细看：成功重建的目标程序恰好都是通过 memorization screen 的程序，而筛查显示存在记忆证据的目标，模型均未能重建。这是一个反向一致的相关性——若分数主要由记忆驱动，应出现相反的分布。但作者明确表示无法排除记忆对性能的贡献，只能说期望 MirrorCode 测得的能力能外推到未见过的代码库。这种「给出倾向性证据、同时诚实声明不可排除」的表述，比非黑即白的结论更有参考价值。对使用者的实际含义是：在 MirrorCode 上得到的分数可以理解为「重建能力的下界」，外推到全新代码库时需保留不确定性余量。

### gotree 案例：近乎完美与严格达标的差距

Claude Opus 4.7 重建的 gotree（约 16,000 行 Go、40+ 命令）通过了 2000/2001 个测试，唯一失败项是一个操作日期标注的小众命令的边缘用例。按严格口径任务并未 100% 完成，但覆盖了几乎全部范围内功能，可视为近乎完美的重建。14 小时、$251 对比人类无辅助估计的 2–17 周，效率差距达两个数量级；而 $2,600 / 19 天的最大任务说明天花板尚未触及——MirrorCode 仍未被解决。


## 实践启示

1. **评测预算必须匹配任务的真实难度。** $1–10 的预算只能测出「短时程能力」；给足资源（最高 $2,600 / 19 天）后，AI 已能完成人类需数周的工作。预算上限本身就是被测能力的组成部分。
2. **防作弊要在信息流上设计，而非靠规则声明。** 模型不可见测试 + 沙箱无网络 + memorization screen 三层各堵一条泄露路径；只声明「不许作弊」而不切断信息通路等于没防。
3. **污染控制可以做成可操作的筛查。** memorization screen 与重建结果交叉验证（通过筛查者被重建、未通过者未被重建），为「分数是否由记忆驱动」提供了可复核的相关性证据。
4. **固定任务集 + 多次运行是榜单可信度的基础。** 15 目标 × 2 语言 = 30 任务、每任务 3 次、100 亿 token + 7 天，牺牲广度换取可比性；开源 scaffold 与 22/25 目标保证社区可复现，3 个私有目标保留甄别能力。
5. **「近乎完美」与「严格达标」要分开表述。** 2000/2001 严格说未解决任务；如实区分两种口径，避免误导下游结论。
6. **长时程自主任务已进入可评测区间。** 14 小时无人工干预完成约 16,000 行重建意味着瓶颈正从「能否坚持长任务」转向 scaffold 质量与预算配置；搭建 agent 工作流时，把「跑多久、花多少钱」当作一等设计参数。

## 相关实体

- Agent 评估基准
- LLM 基准全景
- [[entities/claude-opus-47|Claude Opus 4.7]]
- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 长时程实践]]
- [[concepts/long-running-agent-architecture|长时程 Agent 架构]]
- [[entities/cursor-reward-hacking-coding-benchmarks|编码基准 reward hacking]]

→ [[raw/articles/mirrorcode-long-horizon-benchmark-epoch-ai-metr|原文存档]]
