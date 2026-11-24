---
title: "Vals AI Multi-Agent Vibe Code Bench — 多智能体 vs 单智能体全栈编码基准"
created: 2026-10-13
updated: 2026-11-21
type: entity
tags: [multi-agent, evals, benchmark, coding-agent, vals-ai]
sources: [raw/articles/vals-multi-agent-vibe-code-bench-2026]
confidence: 0.7
---

# Vals AI Multi-Agent Vibe Code Bench

Vals AI 发布受控基准 + 统计显著性检验，比较多智能体团队与单智能体在全栈 web app 编码任务上的表现。核心数字：团队成本是单 agent 的 1.8-5.1 倍，而四个模型×强度对比中仅一个分数差异统计显著——Sol 的 medium-effort 团队比其单 agent 高 7.3 分（p=0.005），其余三个差异（−0.3 到 +3.4 分）均不显著^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md]。

成本-时间权衡出现反直觉形态：对 Sol 而言提高单 agent effort 比组团队收益更大——max-effort 单 agent 得分提升更多，成本只有 medium-effort 团队的 1/1.6（$4.82 vs $3.07/app），但耗时 3 倍；对 Opus 而言提高 effort 与组团队均无显著差异。团队只在 Sol 上更快完成，Opus 的团队耗时最高达单 agent 的 2.3 倍^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md]。

编排风格的质性差异：Sol 的 lead 在最初几分钟把 app 拆给并行 subagent 并亲自测试结果（max effort 下运行时从 38 分钟降到 28 分钟）；Opus 的 lead 通常是先写一份"app 各部分如何拼合"的共享契约，再按 builder/tester 波次串行委派。作者明确提示结论来自单一全栈 web app 基准，不一定迁移到其他工作类型^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md]。

对 wiki 的价值：这是"多智能体 ≠ 更好"命题的少数学术级量化证据之一（成本 1.8-5.1x、显著性 1/4），与 vals 自家 terminal-bench 作弊率研究同属其 agent 评测产品线。

## 深度分析

### 统计显著性把"谁更强"从演示变成了实验

多数多智能体 vs 单智能体的结论来自单次跑分或个案例子，Vals 这份基准的差异在于把比较做成了受控实验：四个模型 × 单/团队 × effort 强度，并给出 p 值^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md:10-11]。结果是发人深省的：团队成本是单 agent 的 1.8-5.1 倍，但四个显著性对比里只有一个成立——Sol 的 medium-effort 团队比其单 agent 高 7.3 分（p=0.005），其余三个差异（−0.3 到 +3.4 分）全部落在噪声区间^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md:10-11]。这意味着在大多数模型上，为组队支付的成本溢价换不来可证明的分数收益；"多智能体 ≠ 更好"第一次有了学术级的量化背书。对评测工作本身的启示是：任何架构对比若没有重复次数与显著性检验，其结论默认不可信——一次跑分的 ±3.4 分波动完全可能是随机性，而行业叙事往往就建立在这种波动上。这也解释了为什么受控基准 + 显著性检验的组合在 agent 评测里仍然稀缺：它需要一个可重复执行、成本可计量的任务环境，Vals 用其评测产品线恰好补齐了这个前提，与 wiki 中 [[entities/ai-evals-methodology|AI 评测方法论]]一贯主张的受控对比、指标闭环思路一致。值得记录的还有样本结构：四个模型中只有一个从团队获益，说明团队路线的收益是模型相关的偶发事件而非普适规律；把组队当作默认架构选项，等于押注自己恰好抽中那一个。

### 成本、得分与时间构成三角权衡，effort 是更便宜的旋钮

对 Sol 而言，提高单 agent effort 的收益大于组团队：max-effort 单 agent 得分提升更多，成本只有 medium-effort 团队的 1/1.6（$4.82 vs $3.07/app），代价是耗时 3 倍^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md:12-13]。对 Opus 则提高 effort 与组团队均无显著差异^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md:12]。时间维度同样反直觉：团队只在 Sol 上更快完成，Opus 的团队耗时最高达单 agent 的 2.3 倍^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md:14]。把三个维度（分数、成本、时长）放在一起看，结论比"团队 vs 单 agent 谁赢"丰富得多：组队不是免费的加速器，它同时改变三个变量，且每个模型上的变化方向都不同——Sol 的团队更快但更贵，Opus 的团队更慢且不显著。这组数字给出一个可操作的决策序：在预算敏感的场景先拧 effort/推理预算这个旋钮，把它推到边际收益递减之后再考虑组队；这与 [[entities/noam-brown-ai-evaluation-reasoning-budget-performance-cost-curve|推理预算-性能-成本曲线的观察]] 互相印证——算力预算花在哪里（更深推理 vs 更多 agent 实例）本质上是同一类资源分配问题，而这份基准给了该问题第一组同任务环境下的实测数据点。对时效敏感的工作流，时间维度的权重还要再抬高：Sol 的团队快而 Opus 的团队慢，说明"并行必然提速"在编排协调开销面前并不成立，由哪个模型担任 lead 决定了团队模式的速度收益是否存在。

### 两种 lead 编排风格：并行拆测 vs 契约先行

质性观察揭示了 lead 角色的两种范式。Sol 的 lead 在最初几分钟把 app 拆给并行 subagent 并亲自测试结果，max effort 下运行时从 38 分钟降到 28 分钟^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md:15]——这是典型的 [[concepts/orchestrator-worker-architecture|orchestrator-worker 架构]]：任务可分解、lead 保留验证权、worker 之间几乎不通信，协调开销集中在拆分与验收两个环节。Opus 的 lead 则先写一份"app 各部分如何拼合"的共享契约，再按 builder/tester 波次串行委派^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md:15]——契约先行牺牲了并行度，换取强耦合任务下的接口确定性，本质是在协调成本与并行收益之间选了不同的取舍点；串行波次也解释了其团队耗时反超单 agent 的部分原因。两种风格没有绝对优劣，取决于任务的耦合结构：可分解的全栈页面拆分吃并行的红利，紧耦合的系统集成则先需要对齐协议再动手。值得注意的是 lead 亲自测试这一细节——验证权上收给编排者，是两种风格中共同起作用的质量护栏，与 wiki 中 [[concepts/multi-agent-team-coordination|多智能体团队协调模式]] 关于角色分工与验收环节的分析可以互相补证。从工程落地角度，两种风格对前提条件的要求也不同：并行拆测要求拆分边界清晰、每个 subagent 可独立验收；契约先行要求团队对接口有共同语言且契约本身可审查——选哪种，先看自己的任务在哪一份前提上更扎实。

### 外部效度边界：单基准结论不做平台外推

作者明确提示结论来自单一全栈 web app 基准，不一定迁移到其他工作类型^[raw/articles/vals-multi-agent-vibe-code-bench-2026.md:16]。这层边界值得在引用时显式携带：全栈 web app 天然可按页面/模块分解，是并行编排最有利的任务形态；对串行依赖强、验证成本高或上下文高度共享的工作（如长链路数据任务、安全攻防、需要单一连贯叙事的写作），团队路线的相对劣势可能进一步放大，而契约先行的相对优势可能更明显。另一层价值在评测设计本身：与 Vals 自家的 [[entities/vals-ai-cheating-on-the-rise-terminal-bench|Terminal-Bench 作弊率研究]] 同属一条产品线，两者共同指向同一命题——agent 评测的可信度来自受控条件与反作弊设计，而不是榜单分数本身；把显著性检验引入架构对比，是把这套可信度标准从"模型能力评测"延伸到了"架构选型评测"。[[entities/agent-eval-counterintuitive-insights-langfuse|Langfuse 对反直觉评测结论的复盘]] 也观察到同类现象：直觉性的架构判断在受控评测面前频繁翻车，这份基准是同一教训在多智能体问题上的又一次复现。对 wiki 读者的引用建议：把"1.8-5.1x 成本、1/4 显著"当作多智能体叙事的默认反例数字记住，同时注明其 web app 基准边界，避免被反向过度引用为"团队永远不值"。

## 实践启示

1. 组队决策前先做带显著性检验的对比实验：四个对比只有一个是真的，单次跑分差异默认按噪声处理，成本溢价（1.8-5.1x）只有在统计显著时才值得付。
2. effort 旋钮优先于团队旋钮：Sol 的数据表明同样预算花在加深单 agent 推理上比花在组队上性价比更高（$4.82 得分更高 vs $3.07），先榨干单 agent 再考虑扩编。
3. 并行拆分的前提是任务可分解且 lead 保留验证权：Sol 的 lead 亲自测试 subagent 产出并因此缩短运行时（38→28 分钟），只派工不验收的编排在质量上没有护栏。
4. 强耦合任务先写共享契约再委派：Opus 的 builder/tester 波次模式说明接口契约是团队路线的协调原语，宁可牺牲并行度也要先对齐拼合方式。
5. 把团队耗时当风险项而非收益项：Opus 团队耗时高达单 agent 2.3 倍，引入多智能体前先测量该任务形态下并行的真实加速比，而不是默认"多人更快"。
6. 引用单基准结论时显式携带其边界：本基准的结论仅在可分解的全栈 web app 上成立，迁移到串行依赖强或验证成本高的工作类型前，需要重新做受控对比。

## 关联

- [[entities/vals-ai-room-temperature-luttinger-magnet-agent-design|Vals AI Luttinger 磁体 agent 设计]]
- [[entities/vals-ai-cheating-on-the-rise-terminal-bench|Vals AI Terminal-Bench 作弊率研究]]
- [[entities/we-ran-250-ai-agent-evals-to-find-out-if-skills-beat-docs-th|250 次 Agent Evals 技能 vs 文档]]
- [[entities/ai-evals-methodology|AI Evals 方法论]]
- [[concepts/multi-agent-collaboration-patterns|多智能体协作模式]]

→ [[raw/articles/vals-multi-agent-vibe-code-bench-2026|原文存档]]
