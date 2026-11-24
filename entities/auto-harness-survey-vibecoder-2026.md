---
title: "AutoHarness 走到哪了：学界业界的能力调研"
created: 2026-07-28
updated: 2026-11-23
type: entity
tags: ['auto-harvested']
sources: [raw/articles/auto-harness-survey-vibecoder-2026]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/auto-harness-survey-vibecoder-2026.md|原文存档]]

- TextArena 145 个游戏中合法动作率达 100% ^[raw/articles/auto-harness-survey-vibecoder-2026.md]

## 摘要

这是一篇关于自动 Harness 演化（Automated Harness Engineering）的学界业界调研，核心论点是 Agent 的下一轮竞争正从模型能力转向 Harness 能力（涵盖 memory、skills、工具描述、middleware、planner、沙箱、权限、发布与回滚）。调研覆盖 DeepMind AutoHarness（把环境规则编译成 Python 代码，TextArena 145 个游戏合法动作率 100%）、Google ReasoningBank（经验记忆，Gemini-2.5-Flash 在 WebArena +8.3pp、SWE-bench Verified +4.6pp）、AHE（搜索 system prompt/memory/skills/tools/middleware，Terminal-Bench 2 pass@1 从 69.7% 升至 77.0%）以及 HARBOR、TTHE、Adaptive Auto-Harness、Meta-Harness 等方案，并给出"两条循环（演化循环/适配循环）+ 三层权限（候选生成器、独立记分器、不可变安全内核）"的生产架构、八周落地计划与四条安全红线。文章同时引用 Anthropic Retro Game Maker（6 小时 $200）与 OpenAI 内部产品（约 5 个月、约 100 万行代码、约 1500 个 PR）作为 Harness 工程进入真实开发流程的证据。^[raw/articles/auto-harness-survey-vibecoder-2026.md]

## 关键要点

- AutoHarness (DeepMind)：环境规则编译成 Python，TextArena 145 个游戏合法动作率 100%，16 个单人游戏 Harness-as-Policy 平均奖励 0.870；能判定动作合法性但无法自动保证任务成功
- ReasoningBank (Google)：演化对象是经验而非 Harness 本身，要求记录来源、适用范围、模型版本、过期时间与反例，避免错误记忆长期污染
- AHE：Terminal-Bench 2 pass@1 69.7%→77.0%；SWE-bench Verified 75.2%→75.6% 但 tokens/trial 下降约 12%——Harness 价值有时体现在更少上下文与更稳定执行；缺口是归因记录不等于因果证明、无原子发布闭环
- 生产架构：演化循环（低频、受限写权限）与适配循环（请求路径、默认只读）分离；候选生成器、独立记分器、不可变安全内核三层权限
- 四条安全红线：不让 evolver 修改身份/权限/密钥/网络出口；不让 proposer 读取 hidden holdout；不让生成候选的模型自行裁定有效；不根据单一 benchmark 直接全量发布
- 真实案例：Anthropic Retro Game Maker 连续迭代 6 小时花费 $200（solo 仅 20 分、$9）；OpenAI 内部产品约 5 个月 100 万行代码 1500 个 PR

## 深度分析

### 演化对象的三条路线：代码、经验与搜索面

把调研里的方案并排看，能清晰地分出三条演化路线，它们各自决定了"能保证什么"和"不能保证什么"。AutoHarness（DeepMind）走的是编译路线：把环境规则编译成 Python 代码，等于把"动作是否合法"从概率问题变成可判定的验证问题——TextArena 145 个游戏合法动作率 100%、16 个单人游戏 Harness-as-Policy 平均奖励 0.870 就是这条路线的收益上限；但调研明确指出它"能判定动作合法性，无法自动保证任务成功"，即编译路线换来的是硬保证，牺牲的是对开放世界目标达成的覆盖。ReasoningBank（Google）走的是经验路线：演化对象不是 Harness 本身而是结构化记忆，Gemini-2.5-Flash 配合后 WebArena +8.3pp、SWE-bench Verified +4.6pp、平均少走约 3 步；它的关键设计是给经验记录来源、适用范围、模型版本、过期时间与反例，本质上是承认经验层必然出错，用元数据把污染变成可审计、可撤销的事件。AHE 走的是搜索路线：把 system prompt、memory、skills、tools、middleware 当作可搜索空间，Terminal-Bench 2 pass@1 从 69.7% 升到 77.0%。三条路线并非竞争关系，而是对应不同可形式化程度：规则能写成代码就用编译，失败模式能总结成经验就用记忆，其余才交给搜索。TTHE 把演化移到测试时、HARBOR 在已有配置开关时做受约束贝叶斯优化、Meta-Harness 在现有组件表达不了新机制时开放代码搜索，可视为搜索路线在不同约束下的变体——Meta-Harness 的消融（只给分数/摘要远弱于完整轨迹）提示：轨迹信息是搜索的原材料，压缩轨迹等于砍掉搜索能力。这一分层视角与 [[entities/harness-engineering-第三代工程范式|Harness Engineering：第三代工程范式]] 中"Automated Harness Engineering"的预测相互印证，也与 [[concepts/agent-self-improvement-loops]] 的自改进回路一致。

### 收益的形状：分数之外是 token、稳定性与路由

调研最有信息量的数据之一是 AHE 的"反直觉"结果：SWE-bench Verified 从 75.2% 到 75.6% 只是小幅提升，但 tokens/trial 下降约 12%。这说明 Harness 演化的价值不能只用 benchmark 分数量化——更少上下文、更稳执行、更低成本同样是收益，且往往更常见。调研归纳的四类公开证据（规则场景的合法性、特定切片的得分提升、token/上下文/步骤下降、长期任务流中的专用 Harness 积累与路由）中，后两类恰恰是纯分数视角会漏掉的。与之配套的是膨胀治理问题：Adaptive Auto-Harness 观察到 prompt 从 2KB 膨胀到 68KB、skills 从 12 涨到 34，解法是用 Git Harness tree 保存不同任务族专用版本并在 solve-time 路由——这实际上把 Harness 当成了需要版本管理与分发的软件资产。对评测体系而言，这意味着验收维度必须同时覆盖分数、成本与稳定性，否则容易滑向调研反复警告的 Goodhart 问题；[[concepts/agent-evaluation-benchmark-frameworks]] 的评测面设计与 [[concepts/eval-surface-rotation]] 的轮换思路正对应这一需求。

### 发布工程：归因记录不等于因果证明

AHE 暴露的工程缺口是整篇调研最"生产向"的部分：归因记录不等于因果证明；Best-of-N 在所有 challenger 都更差时仍可能选出变体；缺少原子发布与密封验收的完整闭环。调研给出的生产架构是系统性回应：演化循环（低频、受限写权限）与适配循环（请求路径、默认只读）物理隔离，三层权限（候选生成器只管提案、独立记分器管回归/隐藏集/安全与成本验收、不可变安全内核管身份/密钥/沙箱/网络出口与最终 validator）把"提出变更的模型"和"裁定变更的机制"彻底分开。八周落地计划则把验收集划分成 development/regression/hidden holdout/metamorphic/security 五类、要求 proposer 看不到隐藏集，发布走 offline replay → shadow → 1% canary 的渐进路径并在安全/SLO/成本/回归超阈值时自动回滚——这与 [[concepts/eval-optimizer-firewall]] 的"验证门"思路同构。安全侧的证据同样指向结构而非提示：Claude Code 复盘显示 reasoning effort、idle thinking 清理、system prompt 变化都能引发回归；AgentSecBench 则强调文字提醒与结构隔离的区别，provenance projection、capability restriction、output validation 才是可执行边界，呼应 [[concepts/agent-security-architecture]] 的分层安全设计。

### 学界与业界的分工与剩余差距

调研的引用结构本身就反映了分工：学界贡献方法与基准（TextArena、Terminal-Bench 2、WebArena、SWE-bench Verified、AgentSecBench 等），业界贡献规模证据——Anthropic Retro Game Maker 以 planner/generator/evaluator 连续迭代 6 小时花费 $200（solo 任务仅 20 分、$9），OpenAI 内部产品约 5 个月、约 100 万行代码、约 1500 个 PR，说明 Harness 工程已进入真实开发流程而非停留在论文里。但两边之间的差距同样被明确列出：代理指标 Goodhart、归因与因果混淆、同批适配、选择器误判、评测成本、供应链风险、模型升级失配、公开复现不完整。其中"模型升级失配"值得单独注意——ReasoningBank 要求记录模型版本与过期时间，正是因为经验与特定模型绑定，模型一换记忆可能集体失效。调研给出的最现实目标因此是保守的："Agent 自动提出变更 → 系统验证大部分证据 → 受限流量试运行 → 最终发布权留给外部控制与人"，即自动化覆盖生成与验证，人类保留最终裁定权。这一立场与 [[entities/agentic-harness-engineering-ahe|AHE：Agentic Harness Engineering]] 的演进框架及 [[concepts/ahe-agentic-harness-engineering]] 的描述一致，也与 [[entities/spec-kit-openspec-superpowers-hybrid-harness|Spec Kit 三层 Harness]] 的分层组织形成互补：后者解决 Harness 如何分层，前者解决 Harness 如何自我进化。

## 实践启示

1. **按可形式化程度选型，而不是追新**：规则可形式化（SQL schema、API 参数、状态机、部署命令）用 AutoHarness 式编译 verifier/policy；历史经验复用用 ReasoningBank 式记忆；已有配置开关用 HARBOR 式约束贝叶斯优化；失败能定位到组件用 AHE 式小变更包；只有现有组件表达不了新机制时才动用 Meta-Harness 式代码搜索。
2. **把"提案者"和"裁判"彻底分开**：演化循环低频、只持受限写权限，适配循环在请求路径上默认只读；生成候选的模型永远不自行裁定有效性——回归、隐藏集、安全与成本验收交给独立记分器，身份/密钥/沙箱/网络出口放进不可变安全内核。
3. **验收维度加上成本与稳定性**：只盯 benchmark 分数会漏掉真实收益——AHE 的 SWE-bench 仅 +0.4pp 但 tokens/trial 下降约 12%，这类变更往往是更好的发布对象；同时用密封 holdout（演化结束后只打开一次）对冲代理指标 Goodhart。
4. **给记忆设计过期与反例机制**：经验条目必须记录来源、适用范围、模型版本、过期时间与反例，否则错误记忆会长期污染行为；经验层的价值正在于易审阅、易撤销，模型版本元数据还能缓解升级失配。
5. **主动治理 Harness 膨胀**：prompt 2KB→68KB、skills 12→34 的膨胀不会自己停下来；按任务族在 Git 里维护专用 Harness 版本并在 solve-time 路由，比单一大而全的 Harness 更可维护。
6. **发布权留给人与渐进流量**：单一 benchmark 和一次短测不足以决定发布；按 offline replay → shadow → 1% canary 渐进放量，安全/SLO/成本/回归任一超阈值自动回滚，最终发布权留在人类与外部控制手里。

## 来源

- 原文: [[raw/articles/auto-harness-survey-vibecoder-2026.md|AutoHarness 走到哪了：学界业界的能力调研]]
- 原始链接: : "https://mp.weixin.qq.com/s/J9FKvVCUjfgj_N3uE35GwQ
