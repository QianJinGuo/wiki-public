---
title: "Claude-Shaped Science：Fable 5 端到端求解 S-matrix bootstrap 椭圆费曼积分（第一手案例）"
created: 2026-10-02
updated: 2026-10-03
type: entity
tags: [anthropic, fable-5, agentic-science, s-matrix-bootstrap, feyman-integrals, elliptic-integrals, bootloop, agentic-coding, ai-for-science, first-party]
sources: [raw/articles/claude-shaped-science-anthropic-2026]
confidence: 0.85
provenance_state: extracted
---

# Claude-Shaped Science：Fable 5 端到端求解 S-matrix bootstrap 椭圆费曼积分（第一手案例）

## 核心案例

物理学家第一手叙述：将分散在数学/物理/计算机三个学科、多种语言（Wolfram Language/C++/Python/Julia）、大量"有论文无代码"文献中的半数值 bootstrap（semi-numerical bootstrap）方法，交给 Fable 5 移植到统一框架——模型 20 分钟复现了作者花数周写的论文结果，还主动指出作者所用算法低效并给出更优算法。^[raw/articles/claude-shaped-science-anthropic-2026.md]

## BootLoop：30 个积分端到端求解

要求 Claude 搜索未解振幅后，发现模型把自己限制在最简单的对数函数族；追问能否处理椭圆函数族后，模型把对数情形的全部机制轻松泛化到椭圆积分类，大部分软件由模型自写或从数学（而非物理）借来。最终 30 个积分 BootLooped 端到端完成：15 个复现已知结果 + **15 个此前从未被计算过**（bootstrap 方法下椭圆费曼积分此前仅有极少数被算过）。整个过程只用了几周。^[raw/articles/claude-shaped-science-anthropic-2026.md]

## 为什么 bootstrap 是 agentic AI 的理想试验场

三个结构特征：①所需专长横跨数学/物理/CS，没有任何个人全部掌握（agent 可聚合分布式专长）；②需要大量编码与算法开发；③**可检验**——同样的数值方法让任何人（无论专家与否）跑两个脚本即可把最终答案验证到任意精度（有时 1,000 位数字）：候选解空间足够小时，高精度数值能精确钉住剩余系数。^[raw/articles/claude-shaped-science-anthropic-2026.md]

## 关键洞见

**问题难度不再是人类专长分布的函数**：S-matrix bootstrap 中"足够简单到人能做"的问题大多已被做完——但专长分布不均（好想法锁在多种语言和没有代码的论文里）才是瓶颈。Claude 聚合了跨学科、跨语言、跨 codeless-paper 的分布式专长，攻克了没有任何单一人类能独立解决的问题。这是"AI 聚合人类分散专长"范式在数学物理上的首次可验证展示。^[raw/articles/claude-shaped-science-anthropic-2026.md]

## 深度分析

### 瓶颈是"聚合"而非"未知"

这篇案例最有力的论点在于重新定义了剩余科学难题的性质：对人类"足够简单"的 S-matrix bootstrap 问题大多已被做完，剩下的不是方法上不可解，而是所需专长横跨数学、物理与计算机科学，散落在多种编程语言和大量没有代码的论文中。换言之，问题之所以"未解"，是因为没有任何单一人类掌握全部拼图。Fable 5 在此的角色并非发明新数学，而是把分布式专长聚合到一个统一框架里——它写出了那些论文从未提供的代码，把"有论文无代码"的沉没知识重新激活。这提示我们：在许多领域，AI 最大的近期价值可能不是超越人类前沿，而是把已知但不可及的知识变成可操作的。

这也解释了作者最初的惊讶与不惊讶并存的反应：惊讶于 20 分钟复现数周的代码工作，不惊讶于模型指出原方法低效——因为后者恰恰是聚合视角的必然产物。一个同时见过 Wolfram Language、C++、Python、Julia 四种实现传统的系统，天然处于能比较不同算法路径的位置，而任何单语言深耕的人类作者都不具备这种比较基准。"低效"不是模型的额外能力，而是聚合本身的副产品。

### 可检验性是 agentic 循环的安全网

为什么 bootstrap 能在几周内产出 15 个前所未有的结果？关键在于它的验证结构：候选解空间足够小时，把振幅在少数点上计算到 1,000 位数字精度，剩余系数就被这些数字精确钉住，任何人（无论是否专家）跑两个脚本即可把答案验证到任意精度。这意味着 agent 的工作闭环不需要等待同行评审——错误会在高精度数值比对中立即暴露。这与其他"可验证奖励"驱动的 agent 领域（如竞赛数学、代码测试）结构同构，说明"生成便宜、验证便宜"的领域会最先被 agentic AI 攻克，而验证昂贵的领域则滞后。

### 模型自我设限与人类引导的价值

一个容易被忽略的细节：当被要求自主搜索未解振幅时，Claude 把自己限制在最简单的对数函数族——它选择了保守的解空间而非激进的目标。质变来自一次人类的追问："能否对椭圆函数做同样的事？"此后模型才把全部机制泛化到椭圆积分类。这说明当前 agent 在"主动选择更难目标"上仍显著弱于"在给定目标内执行"，人类在 harness 中的价值正从"会算"转向"会问"：识别出模型能力与问题难度之间的错配，并用一句话解锁一个数量级的进展空间。

### 15+15 结构：复现与首创的对半信号

30 个端到端 BootLooped 积分恰好对半分为"复现已知结果"与"从未被计算过"，这个结构本身就是方法论信号。复现一半用于校准工具链的正确性——若新方法在已知答案上出错，首创结果的可信度就无从谈起；首创一半则证明泛化是真实的而非对已知文献的过拟合。这种"先复现、后推进、复现与首创混排验证"的节奏，可提炼为 agentic science 输出的一般评估范式：单纯报告新结果而不附带已知结果复现的 AI 科研声明，应降低置信权重。值得注意的是，复现采用的还是"新方法复现"——用泛化后的统一工具链重新得到旧答案，这比用原代码跑一遍的验证强得多：它同时检验了工具链的正确性与方法的内在一致性，两次独立路径收敛到同一数值，几乎排除了系统性偏差的幸存空间。

## 实践启示

1. **优先选择"可检验"的领域作为 agentic AI 切入点**：验证成本远低于生成成本时（如 bootstrap 的高精度数值钉死系数），agent 才能不依赖专家评审而自主迭代闭环；验证只能靠人类直觉的领域应后置。
2. **把"专长聚合"作为第一个任务，而不是直接攻新问题**：让模型先移植、统一分散在多语言/无代码论文中的已有成果，建立工具链与信任基线，再推进到未解问题——顺序颠倒会同时放大错误率与不信任。
3. **用复现已知结果快速校准**：20 分钟复现人类数周的工作，既验证了模型能力，也让人类作者能立刻检查每一处偏差；随后让模型指出原方法低效之处，形成人机互查，比单向审阅更有效。
4. **人类的价值从"会算"转向"会问"**：模型自主搜索时会保守地自我设限（对数族），一次高质量的目标引导（"试试椭圆函数"）就能解锁数量级的进展——投资于问题品味与目标选择，而非微操执行细节。
5. **"论文有但代码无"是普遍的高价值自动化目标**：大量领域的好想法锁在没有代码的文献里，"论文→可运行代码"的翻译任务既低风险（有原文可对照）又高杠杆（激活沉没知识），适合批量交给 agent。
6. **评估 AI 科研产出时要求复现/首创混排**：只看"新结果"数量会被记忆与过拟合污染；要求同时复现一定比例的已知结果作为校准，才能区分真实泛化能力。

## 关联

- [[concepts/agent-self-improvement-loops|Agent 自我提升循环]]
- [[concepts/agent-harness-engineering-paradigm|Harness Engineering 范式]]
- [[entities/omniscientist-multimodal-ai-scientist-2026|OmniScientist 全模态 AI Scientist]]
- [[entities/harness-engineering-self-improvement-survey-lilian-weng|Lilian Weng Harness 自我提升综述]]
- [[concepts/ai-self-improvement-bootstrapping|AI Self-Improvement Bootstrapping]]

→ [[raw/articles/claude-shaped-science-anthropic-2026|原文存档]]
