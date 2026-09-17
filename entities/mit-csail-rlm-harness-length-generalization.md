---
title: "MIT CSAIL RLM: Harness-Driven Length Generalization — 64K to 2M Tokens"
created: 2026-07-22
updated: 2026-09-17
type: entity
tags: [harness, generalization, length-extrapolation, RLM, transformer, MIT, agent-harness]
sources: [raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推]
confidence: 0.6
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# MIT CSAIL RLM: Harness-Driven Length Generalization

MIT CSAIL researchers propose using the **[[agent-harness-architecture|harness]]** as an explicit variable for compositional generalization in recursive language models (RLMs). By training only on **64K-token tasks**, the system generalizes to **~2M token evaluations** — a **32× length extrapolation** — outperforming transformer baselines by approximately 10× in long-task evaluation gains. ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md]

## Core Insight: Harness as a Generalization Variable

The key innovation is repositioning the harness from a peripheral engineering framework to a carrier of **high-level inductive biases**. When tasks share similar decomposition structures (even with different domains and text surfaces), the harness can produce approximately isomorphic model trajectories — the root model learns reusable organizational strategies across tasks. This is formalized through the concept of **task equivalence classes**. ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md]

## Locally In-Distribution (LID) Design

The core architectural principle is **Locally In-Distribution** (LID): while the overall long task may be out-of-distribution, each individual model call is kept close to the training input distribution through two mechanisms: ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md]

1. **Context Unloading** — Task data is stored as variables in the external program environment. The root model first encounters the task structure, while domain-specific content remains external.
2. **Programmatic Sub-Agent Calls** — Sub-agents and tools are treated as function calls in a code environment. Results, intermediate calculations, and tool outputs are stored in variables rather than accumulating in the main context.

## Results

| Task | Training Length | Eval Length | Extrapolation |
|---|---|---|---|
| MRCRv2 | 64K | ~2M tokens | ~32× |
| GraphWalks | <128K | >1M tokens | ~8× |

Using a Qwen3-30B-A3B base with RL training (only updating the root language model), the RLM system approaches or exceeds GPT-5.5 reference results on four long-task benchmarks. Transformer baselines (including YaRN-extended) show improved training rewards on short tasks but limited long-task improvement — confirming that without harness-level generalization design, length extrapolation remains fundamentally constrained. ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md]

## 深度分析

### 1. 泛化变量从权重挪到 harness：改的是「谁承担归纳偏置」

传统长度泛化都在模型侧改（位置编码、注意力结构、长序列续训）；这项工作把承载高层归纳偏置的位置从权重搬到 harness——模型只负责"在局部条件下做一次规整调用"，任务怎么切、中间结果放哪、根模型看到什么轨迹由 harness 决定 ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:72-82]。作者用「任务等价类」刻画它：领域与文本表面迥异、底层分解结构相近的两个任务，在同一 harness 下走出近似同构的轨迹，于是学到可跨任务复用的组织策略而非领域表面模式。证据是同底座对照：RLM 与 transformer+YaRN 基线共用 Qwen3-30B-A3B，短任务奖励都在涨，却只有 RLM 把增益带进长任务评测（约 10 倍）^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:174-184]。但这是机制性证据而非分离性证明（同时变的还有动作空间与轨迹形态）：具备程序化分解动作时承载位置可外移，缺少 harness 侧承载则长度外推仍受结构性约束。参见 [[concepts/agent-harness-engineering-paradigm|harness 工程范式]]。

### 2. LID 是可操作的设计约束，不是修辞

LID（Locally In-Distribution）可直读为一条约束：**整条轨迹可以处于分布外，每次调用的输入却应尽量落在训练分布内** ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:49]。反直觉之处：长轨迹崩掉不是因为"长"，而是工具结果与中间状态不断回灌主上下文，形态随步数漂移——不是容量问题，而是输入形态保真问题。两项机制把它落地：**上下文卸载**（任务数据以变量存于外部程序环境）与**程序化子 agent 调用**（子 agent 与工具统一为代码环境中的函数调用，结果存入变量而非堆进上下文）^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:116-126]。作者另有一个否定性论断：**递归不是 LID 的必要条件**，关键在信息隔离与根模型看到的轨迹结构 ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:152]——故 LID 可移植到非递归的单层 agent loop，与 [[concepts/context-management-agent-systems|上下文管理]] 同族。另一半是**长度不变性**：短/长任务须采用大致相同、不随长度变化的分解策略，长度增加只应表现为"更多子调用" ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:211]；否则模型会学到只在短任务有效的取巧策略。

### 3. 32× 如何测出，被什么条件框住

口径：同底座 Qwen3-30B-A3B 上以 RL 分别训练标准 RLM、带分解提示的 RLM 与 transformer+YaRN 基线，**只更新根语言模型**；六项任务分别拉长输入/输出/指令数量，训练只覆盖较短任务，评测扩到训练规模的 8–32 倍 ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:174-179]。MRCRv2 由 64K 外推到约 200 万 token 评测、GraphWalks 由不足 128K 延伸到 100 万 token 以上 ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:189]，同一 harness 下训练的 30B 在四项长任务中接近或超过 GPT-5.5。边界条件多由作者自陈：LID 是设计目标而非可严格保证的性质（任务信息仍可能回流主上下文），跨域迁移依赖任务间确有可复用的共同结构。成本约束更硬：多步执行与等待子调用使训练耗时约为基线 1.5–3 倍，8×H100 上 30B 的长轨迹 ReAct 训练很快难以承受 ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:289-294]。软肋是轨迹相似性只用 token/n-gram 表面指标度量，无法证明两条轨迹采用了真正等价的分解策略 ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:258-263]。

### 4. 与递归式长上下文推理、长程 agent 任务的关系

RLM 属于"把长上下文问题转化为调度问题"的一支：不让单次调用看得更长，而让一次长任务由许多次短调用构成。这同时解释了该路线的收益与同类方案的失败面——凡把中间产物留在主上下文的迭代式方案（[[concepts/lost-in-the-middle|lost-in-the-middle]] 式注意力稀释、朴素 ReAct 的轨迹膨胀）都随步数累积漂移，而把中间产物外置到变量者把漂移从"必然"降为"可控"。对长程 agent 任务，可用判据是：**长度增加时根模型看到的轨迹结构是否保持不变**——若只体现为同构子调用变多，泛化就由 harness 承载；若迫使单次调用变长或中间结果回流主上下文，就退回权重侧的老问题。这给 [[entities/towards-long-horizon-agents-survey-mozi-space|长程 agent 综述]] 的"长任务如何拆解与续跑"补上了可测量的中间变量。

### 5. 效能投在哪：harness 设计 vs 微调，与评测口径的坑

作者立场比外部解读保守：不主张把 MapReduce/动态规划等策略预置进 harness；扩大训练数据仍是能力提升的主要动力，harness 决定训练所得能否迁移到更长任务与新领域，更高层归纳偏置也应端到端 RL 学出 ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:299-304]。故正确读法是分工：**harness 制造局部同分布的调用条件，训练在这些条件下把策略学出来**。评测要守住三个对照：同底座、同短任务奖励水平（基线短任务奖励也在涨，只比绝对值会把"没学会"读成"外推差"）、只比评测增益斜率而非单点分数 ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md:184-189]。证据形态是研究者博客/论文式报告而非严格同行评审，本条目 confidence 记 0.6 亦反映"机制可靠、数字待复现"。参见同源条目 [[entities/mit-rlm-harness-length-extrapolation-paperweekly-2026|同源长任务外推]] 与原始工作 [[entities/language-model-harnesses-compositional-generalizers-alex-zhang-2026|harnesses as compositional generalizers]]。

## 实践启示

1. **把 harness 当泛化变量设计**：先问"每次调用看到的输入是否落在训练形态内"，再问模型大小。检查项：任务数据外置为变量、工具/子 agent 统一为函数调用、中间结果只以句柄回流主上下文。
2. **用长度不变性审查分解策略**：短/长任务用同一套切分方式，掐掉"短任务里把整个问题丢给一个子调用"的捷径。自检：长度翻 10 倍时，轨迹结构是否只多了同构的若干节点。
3. **优先上下文卸载，而不是上下文压缩**：压缩是在主上下文里做减法、形态仍随步数漂移；卸载把领域内容搬到外部程序环境，主上下文只留结构。实现可参考 [[concepts/context-management-agent-systems|上下文管理]] 与 [[concepts/subagent-spawning-pattern|子 agent 生成模式]]。
4. **长任务训练预算按 1.5–3× 起步估**：30B 级长轨迹 ReAct 训练在 8×H100 上很快难以承受。先小规模验证可行性，再纳入常规流水线。
5. **不要硬编码高级算法进 harness**：更高层的归纳偏置应通过端到端 RL 学出，harness 负责让这些能力可迁移——即 [[concepts/agent-harness-engineering-paradigm|harness 工程范式]] 的"harness 提供条件、训练提供策略"分工。
6. **锁死评测对照条件**：同底座、同短任务奖励水平、只比增益斜率；并承认"轨迹同构"目前只有 token/n-gram 表面相似度支持，不能证明分解策略等价。

## Implications for Agent Systems

This work elevates the harness from a deployment concern to a **first-class generalization variable** in agent system design. It suggests that agent architectures capable of decomposing long tasks into locally in-distribution calls — through well-designed harnesses — can achieve length generalization impossible through model architecture alone. This aligns with the broader [[agent-harness-engineering-paradigm|harness engineering]] movement in agent development. ^[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推.md]

→ [[raw/articles/mit团队把泛化写进harness短任务训练解锁32倍长度外推|原文存档]]
