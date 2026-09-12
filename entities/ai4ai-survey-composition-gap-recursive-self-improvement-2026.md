---
title: "第一篇 AI4AI 综述 — Composition Gap 与递归自我改进的证据阶梯"
created: 2026-09-11
updated: 2026-09-11
type: entity
tags: [ai4ai, recursive-self-improvement, survey, composition-gap, harness, evaluation, rsi, automated-research, long-horizon, agent]
sources: [raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026]
confidence: 0.7
provenance_state: extracted
---

# 第一篇 AI4AI 综述 — Composition Gap 与递归自我改进的证据阶梯

> **Background**：本文基于新智元 2026-09-11 对首篇 AI4AI（AI for AI）综述的报道整理。原论文（preprints.org 202608.2108）梳理数百项研究，把 long horizon 智能体、AI4AI 与 recursive self-improvement 放进同一张研究地图，并提出 **composition gap（组合鸿沟）** 作为核心诊断概念。配套资源：Awesome-AI4AI（GitHub Pages）、项目 Blog（simpleagentlab.com/ai4ai）、Harness（github.com/simple-agent-lab/RSIHub）。

## 核心命题：Can AI improve AI?

论文把问题从「AI 能做什么」推进到「Can AI improve AI?」——AI 能否可靠地改进 AI，并让进步持续积累。这意味着 AI 要提出方案、修改代码、运行实验、判断结果，再把有效的改进留下来；最诱人的前景是这一轮变强的系统还能帮助下一轮做得更好。综述的结论是有边界的肯定：**AI 已能在明确的目标和评价规则下承担不少研发工作，但单项能力的提升还不能保证完整研发流程可靠，更不能证明系统能够持续自我改进。**^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

## Composition Gap：每一步都能做，连起来仍可能失败

论文将「会做某一步」与「能把整件事做成」之间的缺口概括为 **composition gap**。一次 AI 研发可能从查资料开始，经过提出假设、修改代码、运行实验，最后形成结论；这些步骤需要共享同一个目标，也需要使用彼此产生的证据，而麻烦往往出在交接处——例如代码已经更新，分析时却用了上一轮的日志；实验指标看似提高，报告中的对照组却采用了另一套配置。单看每项操作都像是在推进工作，把它们连起来，结论却可能失去依据。^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

因此，检索、编程和工具调用的单项成绩不能直接当作完整研发能力的证明。long horizon 的关键在于前一步会影响后一步：一个早期选择可能改变后续能够采取的行动，一个错误可能直到训练结束后才暴露。系统需要记住关键状态、理解反馈来自哪次尝试，再据此修正计划——更有价值的问题不是「运行了多久、调用了多少次工具」，而是「它能否在步骤相互依赖、结果延迟出现时，仍然把目标、版本和证据保持一致」。^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

## 模型侧与 Harness 侧：会执行 ≠ 掌握研究过程

综述观察到 AI 正承担更多规划、编程、实验和修复工作，但人类通常仍在决定研究目标、评价标准以及什么结果可以被接受——让系统按既定规则优化某个指标，与让它独立判断哪个问题值得研究、什么证据足以支持结论，需要的能力并不相同。论文从两侧梳理改进方向：一侧是**模型侧**（学会规划、正确使用工具、从较长行动过程与反馈中学习），另一侧是 **Harness**（组织智能体运行的工具、记忆、执行和检查机制——保存实验记录、核对执行结果、遇到问题时恢复、判断何时停止或请人处理）。^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

这一区分直接影响评测成绩的读法：同一个模型，换一套工具、增加重试次数或允许更多人工帮助，结果都可能变化；比较时必须说明预算、工具、评估方式和人工介入条件，才能知道提升来自哪里。^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

## 四类证据阶梯：一次涨分 ≠ 持续变强

当一个系统报告「改进成功」时，最直观的证据是指标提高了，但综述认为至少还要分开看四件事：

- **Measured Gain** — 在声明的指标上，是否测到了实际增益？
- **Retention** — 继续迭代之后，收益是否仍然存在？
- **Human Comparison** — 在相近时间和计算预算下，与人类相比表现如何？
- **Held-out Transfer** — 离开优化时使用的任务、模型或领域，收益是否仍然成立？

这四类证据不能相互替代：一套系统可以在某个测试上提高，却没有证明它能够保留收益或迁移到新任务；研究没有报告某项结果，也不能直接理解为已经测试失败。多做几轮不一定越来越好——系统可能忘记早期约束逐渐偏离目标，也可能越来越善于迎合评分方式却没有解决真实问题，某个版本一度表现最好、后续修改也可能把收益丢掉。要支持更强的 recursive self-improvement 主张，需要跟踪后续版本，在明确且可比较的条件下持续验证。^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

## 概念三层区分与评估三维度

综述严格区分了三个常被混用的概念：**AI4AI**（AI 优化一个独立模型/工具/数据/评测方法）⊂ **自我改进**（AI 修改的是自身组件）⊂ **recursive self-improvement**（「构建下一代系统的过程」本身可被改进，且收益能传递到后续版本）。阅读每项工作时，综述追问五件事：改进对象是什么、是否改到自己、谁控制目标与各个环节、凭什么判断成功、改进能否被保留和迁移。^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

由此把三个容易混淆的问题分开：**任务有多难、AI 有多少决定权、改进证据有多充分**——运行时间长不一定代表任务难，自动执行了所有步骤也不一定代表目标和判断标准由 AI 决定。^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

## 与既有实体的关系

本文是 AI4AI/递归自我改进簇的最新综述层，与 wiki 既有实体互补而非重复：

- [[entities/llm-self-improvement-system-survey-zesearch-nlp-2026|Zesearch 自我提升综述]] 从**模型自身能力**出发给出四阶段闭环（数据获取→筛选→优化→推理细化）；本综述则从**研发流程完整性**出发，用 composition gap 解释「阶段能力齐备但端到端失败」。
- [[entities/recursive-automated-ai-research-first-steps-2026|Recursive 自动化研究系统]] 给出工程侧 SOTA 实证（NanoChat/NanoGPT/SOL-ExecBench）；本综述提供判读这些结果是否构成「真进步」的四类证据阶梯。
- [[entities/ai4ai-cognitive-exoskeleton-meta-learning|AI4AI 认知外骨骼]]、[[entities/agent-self-improvement-six-mechanisms|Agent 自我改进六机制]]、[[entities/harness-engineering-self-improvement-survey-lilian-weng|Harness 自改进综述]] 分别覆盖机制/工程层；本综述补上 Harness 作为「研发现场配套设施」的定位。
- [[concepts/ai-self-improvement-bootstrapping|自改进自举]] 与 [[concepts/agent-self-improvement-loops|Agent 自改进循环]] 是本实体在概念层的落点；[[entities/lossy-self-improvement|有损自改进]]、[[entities/ai-recursive-self-improvement-nanogpt-prime-intellect|nanoGPT RSI]]、[[entities/science-discovery-tree-search-rsi-research-code-2026|科研发现树搜索]] 是被本框架统一解释的邻居证据。

## 实践启示

对智能体开发团队，综述给出一套定位问题的思路：先判断「是模型不会做、信息传递出错、评价机制不可信，还是有效改进没有被保留下来」——不同原因需要不同解决办法。对研究团队，一个有价值的方向是让 AI 接手目标清楚、结果可检验的研发环节，同时保留可追溯的实验记录，便于判断哪些工作可靠、哪些仍需人类参与。更远的前景是让系统把经过验证的经验和收益交给下一代，但按综述梳理的证据，可靠的研究判断、持久且可迁移的收益、跨代累积的改进仍有待更充分的验证。^[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026.md]

→ [[raw/articles/ai4ai-survey-composition-gap-recursive-self-improvement-2026|原文存档]]
