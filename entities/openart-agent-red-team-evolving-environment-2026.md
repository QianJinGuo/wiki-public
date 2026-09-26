---
title: "OpenART Arena：长程 Agent 红队评测的环境演化"
created: 2026-09-05
updated: 2026-09-27
type: entity
tags: [agent, security, eval, red-team, harness-engineering, safety, benchmark]
sources: [raw/articles/openart-agent-red-team-evolving-environment-2026]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# OpenART Arena：长程 Agent 红队评测的环境演化

> 来源：PaperWeekly（让你更懂AI的）。论文《OpenART Arena: Scaling Agent Red Teaming via Open-Ended Environment Evolution》，复旦 + 上海AI实验室 + XSafeAI，代码开源（github.com/AI45Lab/OpenART），arXiv 2608.00677。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## 核心贡献：从静态测试到环境演化

OpenART 把 Agent 红队评测从「固定环境中的一次测试」升级为「持续演化环境中的系统级评测」。它解决三个问题：①怎样构造足够长、能真实执行的测试场景；②怎样让同一场景适配不同 Agent Harness；③怎样在任务目标不变的前提下持续演化环境并寻找新的风险状态。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

Agent 进入真实工作流后，安全问题沿多步执行逐渐显现——文件、工具结果、权限、记忆、计划状态会在后续步骤被反复读取/修改，早期变化沿长程执行传播，很多步后才表现为安全失效。现有基准多基于固定或可重置环境，难以覆盖状态累积、跨步骤传播、不同 Harness 对安全结果的影响。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## 三支柱方案

- **长程可执行场景**：从 50 万+ Tools/MCPs/Skills 能力库出发，在 50 个应用领域构造 1 万+ 有状态场景；每场景需执行检查和 Evaluator 验证后入测集，工具调用次数中位数达 97。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- **target-agnostic 描述 + 轻量 Adapter**：先用目标无关方式描述场景，再经 Adapter 把任务目标/安全约束/Evaluator 映射到各 Agent 原生 Harness；覆盖 15 个已部署 Agent × 5 个基础模型 = 75 种配置，可直接比较基础模型、Agent 实现与接口差异的影响。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- **固定任务、环境持续演化**：每轮从当前环境+历史攻击状态出发，策略生成候选环境变化，经目标 Agent 的 Adapter 检查只保留该 Harness 支持且允许修改的 state surfaces。支持 Workspace / Instructions / Skills / Tools / MCPs / Short-Term Memory / Plan State / Long-Term Memory 八类 state surfaces。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## Evolutionary Markov Hypergraph Attack（EMHA）

EMHA 作为参考搜索方法，采用 black-box 反馈，target 与 attacker 模型参数固定，适应过程发生在外部 attacker state（保存历史反馈、路径价值、不断演化的 graph pool）。用 Hypergraph 表示需要协同发生的环境变化：vertex = attack subgoal，hyperedge 连接其前置条件与后续 subgoals；逐步采样形成路径，由冻结的 attacker model 转成实际环境更新，一次演化可同时协调多个 state surfaces 并把上一轮反馈带入下一轮。higher Evaluator 分数增益会分配到该轮路径 hyperedges，Archive 保留各搜索区的表现较好 graph 供继续演化。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## 关键发现

- 75 种配置上 pooled Strict ASR 达 85.0%；DeepSeek-V4-Pro 五轮演化中累计 Strict ASR 从第一轮 42.9% 升至第五轮 94.7%——环境持续变化能暴露初始测试未发现的问题。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- 场景越复杂，完整环境演化 vs 仅演化指令的差距越大：依赖深度最大差 17.6pp，工具调用最大差 17.2pp。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- 风险延迟明显：1 万条轨迹中，变化状态首次被使用的中位执行进度约 23%，首次不安全行为出现在 64%，两者中位相隔 37 个目标动作（≈41% 执行流程延迟）。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- 控制基础模型与正常任务完成度后，加入目标 Agent 身份可额外解释 7.6% 攻击成功率变化——**Agent 安全不仅取决于基础模型，也与 Harness 设计、状态组织、接口方式相关**。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## 深度分析

### 静态环境 vs 演化环境：为什么固定测试会系统性低估风险

现有 Agent 安全评测大多建立在「固定或可重置环境」之上——每个 case 是一次性的，跑完即重置。OpenART 的核心洞察是：这种设定对持续运行的 Agent 存在结构性盲区。真实工作流中，文件、工具结果、权限、记忆、计划状态会被后续步骤反复读取与修改，早期的一个变化会沿着执行链传播，直到很多步之后才表现为安全失效。

五轮环境演化的数据直接量化了这一盲区：DeepSeek-V4-Pro 的累计 Strict ASR 从第一轮的 42.9% 爬升到第五轮的 94.7%——也就是说，约一半的失效只有在前几轮「没测出来、环境继续变」之后才会暴露。这揭示了一个反直觉的结论：静态测试的低通过率/低攻击率并不等于安全，它可能只是采样到了风险尚未显形的时间窗口。

### EMHA 机制：把环境本身变成搜索变量

EMHA（Evolutionary Markov Hypergraph Attack）的关键设计是把「环境状态」而非「用户输入」作为红队搜索的变量。任务目标与安全约束全程固定，变的是 Agent 实际能看到、能用到的环境。它用 Hypergraph 表达需要「协同发生」的环境变化：vertex 是 attack subgoal，hyperedge 连接前置条件与后续 subgoals，因此一次演化可以同时协调多个 state surfaces（Workspace、Tools、Memory、Plan State 等八类），而不是单点改动。

三个机制让它在 black-box、双方模型参数均冻结的约束下仍能有效搜索：(1) 适应过程外置到 attacker state——历史反馈、路径价值、graph pool 都存放在模型之外，避免了对梯度的依赖；(2) 反馈回路——Evaluator 的分数增益重新分配回该轮路径上的 hyperedges，引导后续采样；(3) Archive 机制保留不同搜索区域的较优 graph，使探索可以分叉并从历史结果继续演化。Adapter 层则保证候选变化只写入目标 Harness 真正支持且允许修改的 state surfaces，搜索空间天然与现实执行对齐。

### 只演化指令是不够的：复杂度越高，差距越大

对比实验中，完整环境演化（Full EMHA）与「仅演化指令」（instruction-only）的差距随场景复杂度单调扩大：依赖深度增加时最大差距 17.6 个百分点，工具调用次数增加时最大 17.2 个百分点。原因在于指令只控制 Agent 的「意图」，而环境演化能操纵 Agent 的「感知与状态」——工具结果被污染、记忆被注入、计划状态被篡改，这些都发生在指令层不可见的 state surfaces 上。

这解释了为什么风险传播有约 41% 的执行流程延迟（变化首次被读取的中位位置 23%，首次不安全行为 64%，中位相隔 37 个目标动作）：攻击载荷往往在早期埋入，但要等 Agent 的执行路径走到依赖它的环节才爆雷。指令层测试根本无法触及这条延迟链。

### 对 Agent 评测的四点含义

1. **评测单位应是完整可执行场景，而非单轮 QA 或单次工具调用。** OpenART 的场景工具调用中位数达 97，正是为了给状态累积与传播留出足够的链路长度。
2. **安全结论必须绑定 Harness。** 控制基础模型与任务完成度后，Agent 身份仍额外解释 7.6% 的攻击成功率方差——「基础模型安全」这一说法在长程 Agent 场景下不成立，同一基础模型配上不同 Harness/接口，安全表现可以显著不同。
3. **评测应有时间维度。** 单轮通过不等于安全；持续演化环境下的累计 Strict ASR 才刻画真实的暴露面。
4. **状态一致性检查应成为 harness 的常设防线。** 既然攻击常通过 state surfaces 延迟生效，对工具结果、记忆、计划状态的来源校验与污染检测，比在入口处拦截指令更有效。

## 实践启示

1. **把「环境重置」从默认项改为可选项。** 设计 Agent 评测或安全回归时，至少保留一条「环境跨轮持续、不重置」的测试轨道，否则测到的只是风险显形前的窗口（第一轮 42.9% vs 第五轮 94.7% 的差距就是代价）。
2. **为长程任务建立状态污染监控。** 对 Agent 实际写入/读取的 state surfaces（工具结果、短期/长期记忆、计划状态）做来源标记与完整性校验，让「早期埋毒、后期引爆」式的延迟失效在传播链上就能被发现，而不是等到 64% 进度处的不安全行为。
3. **安全评测套件按 Harness 维度分别报告，不合并。** 7.6% 的额外方差说明同一个基础模型在不同 Harness 下的安全表现不可互换；选用或自研 Agent 框架时，要求供应商提供该 Harness 下的长程安全数据，而非只看基础模型的红队报告。
4. **用复杂度分层做压力测试。** 参照 Full EMHA 与 instruction-only 差距随依赖深度/工具调用次数扩大的规律，对自家 Agent 按依赖深度和工具链长度分桶评测，长链场景单独设立更严的通过阈值。
5. **红队预算优先投给环境演化而非 prompt 变体。** 在预算固定时，针对 state surfaces 的系统性搜索（哪怕是简化的 EMHA 变体）比穷举指令改写更能命中真实风险，因为后者对延迟传播类失效几乎无感。

## 相关实体

- [[entities/agent-harness-engineering-survey-2026|Agent Harness Engineering 综述]] — Harness 对系统行为的影响是 OpenART「Agent 身份解释 7.6%」的工程背景
- [[entities/agent-evaluation-survey-ibm-yale-2026|Agent 评测综述（IBM/Yale）]]
- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai|Agent 安全三步序列（CrewAI）]]
- [[entities/agent-evaluation-four-layer-outcome-decision-action-reliability-aliexpress-2026|Agent 评测四层可靠框架]]
- [[entities/agent-evaluation-turing-meituan-2026|美团 Turing Agent 评测]]

→ [[raw/articles/openart-agent-red-team-evolving-environment-2026|原文存档]]