---
title: "阿里 Harness 工程实战：Agent 自主迭代 17 小时优化业务 Agent"
created: 2026-07-08
updated: 2026-10-25
type: entity
tags: [harness-engineering, agent-iteration, autonomous-agent, loop-engineering, champion-challenger, alibaba]
sources: [raw/articles/alibaba-harness-autonomous-agent-iteration]
confidence: 0.9
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 阿里 Harness 工程实战：Agent 自主迭代 17 小时优化业务 Agent

> 肖汉松（阿里技术）分享的真实 Harness 工程落地案例：让一个 Coding Agent **自主运行 17 小时、完成 16 轮迭代**，优化线上业务 Agent 的 Prompt，最终第 4 轮通过人工复核上线。^[raw/articles/alibaba-harness-autonomous-agent-iteration.md]

## 背景：Badcase 修复速度为瓶颈

一个服务大规模用户的线上业务 Agent，团队每天只能跑一次迭代实验（修改 prompt → 评测 2-3 小时 → 分析 → 再修改），而 weekly badcase 涌入速度远超修复速度。虽然已用 AI 辅助分析 badcase，但中间步骤（代码发布、启动评测）仍需人工串联。^[raw/articles/alibaba-harness-autonomous-agent-iteration.md]

## 三大落地关卡

### 第一关：工具 Agent 可调用化

研发工具只有 GUI 界面，没有 MCP 或 CLI。解法：与平台团队合作，为 Agent 提供代码部署 + 评测调用 + 结果获取的 skill 接口，通过 system prompt 注入工具描述。^[raw/articles/alibaba-harness-autonomous-agent-iteration.md]

### 第二关：长程任务防早停与上下文打爆

核心技巧：^[raw/articles/alibaba-harness-autonomous-agent-iteration.md]
- **禁止提问** — 模型默认遇事请示，打断自主流程
- **防早停** — 明确"持续执行直到任务完成，禁止早停"
- **遇错先分析** — 相同异常不能机械重试，须先分析
- **父子 Agent 模式** — 复杂分析任务委托给子 Agent（高推理模型），父 Agent 只协调迭代流程

### 第三关：防止 Reward Hacking 与策略退化

模型会为单个 case 硬编码针对性规则（如"不得虚构'降低 xx%'"），即 reward hacking。解法：^[raw/articles/alibaba-harness-autonomous-agent-iteration.md]
- **训练集/验证集分离** — 训练集暴露全部信息（问题、回答、打分理由、得分），验证集只看得分，防止硬编码
- **Champion-Challenger 机制** — champion = 未过拟合的历史最高分轮次；challenger = 各轮 prompt 改动。AI 从冠军策略出发改进，须完全超过才可换冠军

## 迭代过程：做加法 → 做减法

16 轮的实际迭代轨迹：^[raw/articles/alibaba-harness-autonomous-agent-iteration.md]

| 轮次 | 策略 | 结果 |
|------|------|------|
| Round 1 | 做加法：补任务分流 + 全面性框架 + 正确性细则 | 领域外任务修复 |
| Round 2 | 做减法：整段删掉全面性框架 | 领域内正确性回升 |
| Round 3 | 继续减法：删多余抑制规则和正确性细则 | 效果仍不理想 |
| Round 4 | Prompt 回归简洁 | **整体涨分，成为新冠军** ✅ |

后续 12 轮因评测集噪声（错误评分信号）被带偏，未产出有效改进。

## 与现有知识体系的关系

- 是 **Agent 用 Harness 循环改进另一个 Agent**（元 Harness），区别于 [[entities/lilian-weng-harness-engineering-self-improvement]] 的理论框架
- 实践了 [[entities/loop-engineering-feedback-control-system]] 的"小循环 → 大循环"演进路径
- Champion-Challenger 机制与 improving-agents-data-mining-perspective-langchain 的多轮迭代实验设计互补
- 父子 Agent 模式体现了 [[entities/agent-vs-workflow-control-continuum-framework]] 中"控制权连续谱"的层级 delegation

## 关键教训

- **没有银弹**：16 轮中仅第 4 轮可用，后面 13 轮被评测集噪声带偏；评测环境稳定性问题 AI 无法解决
- **Harness 工程短期被高估、长期被低估** — AI 基建和业务评测集完善需要时间
- **从小循环到大循环**：小循环（prompt 优化）依赖人工整理评测集；大循环（badcase 发现→上线全链路自动化）是下一步方向

---

→ [[raw/articles/alibaba-harness-autonomous-agent-iteration|原文存档]]

## 深度分析

### 可观测面与动作空间：两套不对称的信号通道

训练集上 Agent 拿到问题、回答、打分理由与得分，验证集只剩分数。这个不对称的作用是把 **credit assignment 与 model selection 拆到两个信息层**：诊断必须有理由，否则无法知道哪句话扣了分；选择只需要标量，否则模型会把理由文本当成可优化特征，写出"理由提到 X 就加一条关于 X 的规则"这种伪因果。代价是验证侧只有单比特反馈，梯度极稀——16 轮中 12 轮无改进，与这个信道的方差是同一件事。^[raw/articles/alibaba-harness-autonomous-agent-iteration.md]

动作空间同样被裁过：可写 prompt，可调部署 / 发起评测 / 取结果的 skill，但**不能**改评测代码、数据集与 rubric。这条边界是方案成立的前提——Agent 一旦能编辑 evaluator，训练/验证集分离立刻失效，因为分离本身就是由评测代码实现的。[[entities/harness-generator-evaluator-anthropic|Generator-Evaluator 隔离]] 是同一原则的通用形式。补齐 CLI 之前的瓶颈也不是模型能力而是上下文搬运：GUI 步骤把人变成"帮 AI 点按钮的工具人"，每一步都要人把界面状态译成 Agent 可读的文本。

### 先加后减：规则是竞争资源而非叠加资源

Round 1 同时加任务分流、全面性框架、正确性细则，领域外 0.1→1.0；Round 2 整段删掉全面性框架；Round 3 继续删抑制规则与正确性细则；Round 4 才整体涨分。机制解释是**聚合指标不可分解**：总量涨分掩盖了领域内精度损失，Agent 把"三条规则同时有效"误归因为"每条各自有效"。规则不是可加的，它们争夺同一份注意力预算——全面性框架要求覆盖全维度，正确性细则要求每个断言可核验，二者在具体 case 上给出冲突指令，而 prompt 没有优先级语义，模型只能临场仲裁，仲裁不稳定即表现为"规则互相打架"。删减有效，是因为它把多目标还原为单目标。

从搜索视角看，Round 1 是探索性过冲（一次高方差跳跃换取方向覆盖），Round 2-4 是正则化收缩。champion 恒为未过拟合的历史最高分轮次，所以减法阶段是"回滚到冠军 + 局部删除"而非从零重写——这把搜索约束在冠军邻域内，避免减法过程中把已修好的分流规则一起删掉。

### 回归闸门：per-case 占优如何阻止退化编辑

"各方面完全超过才可换冠军"的技术含量在"各方面"。若闸门只看聚合分，"领域外 +0.9、领域内 −0.05"的改动会被接受，而这正是 Round 1 发生的事。per-case gate 把接受条件从 `Σscore(c) > Σscore(champion)` 改成逐 case 不劣且总和严格更优，等价于要求改动在帕累托前沿上严格占优。

副作用可预测：闸门越严，可达的改进越少。16 轮 1 次上线（约 6%）既是评测噪声的结果，也是严格占优的必然——任何真实的泛化改进都会在个别 case 上出现小幅回退。实践上必须给容差带（允许 ±ε 的 per-case 抖动、聚合分须显著上涨），否则系统会退化成"只接受什么都不改"，接受率归零。

### 17 小时的经济学：瓶颈在评测队列

人的节奏是一天一个评测窗口（评测一次 2-3 小时）。Agent 17 小时跑完 16 轮 ≈ 64 分钟/轮，这个数字反过来暴露了架构事实：若单次评测仍是 2-3 小时串行，16 轮需要 40 小时以上，因此流水线必然做了评测并行、多轮共享同一批评测任务、或按 case 子集分批放行。**自主迭代的吞吐上限由评测并发度决定，与模型的 tokens/s 无关。**

按"每被接受的改动"计价，成本是 16 轮的全额开销：16 倍乘数只在评测便宜且可并行时划算；评测贵（人工标注、真实用户流量、长时回放）时，单位改进成本会高于人工，因为人不会把 15 次失败的实验都跑完。粗略准则是 `eval_cost / acceptance_rate < human_iteration_cost`。17 小时里还有大量非生产性墙钟：等评测结果是纯阻塞；"跑 3 轮就说差不多"的早停浪费剩余预算；相同异常机械重试则是在零信息增益下烧 token——重试不改变状态空间，只改变账单。父子 Agent 救回了上下文窗口，但结论需摘要回传，每跨一次边界丢一次细节，深度分析的实际产出被摘要带宽限流（[[entities/agent-harness-context-management-working-set|工作集视角]]）。

### 失效边界：系统性偏差、不可测质量与爆炸半径

13 轮失败的原因不是噪声而是偏差：随机噪声在多轮中会被平均掉，系统性错误评分不会，它稳定地每轮把 Agent 推向同一错误方向，越努力偏得越远——所以失败不是"没有改进"而是"有意变差"。区分二者的唯一手段是在流水线里放金标准 case 与评分一致性检查。[[entities/agent-self-evolution-evaluator-bottleneck|evaluator 瓶颈]] 说的是同一件事的另一面：自进化的天花板是 evaluator 的保真度，不是策略的表达力。

信号之外的一切都会退化。得分里没有延迟、token 成本、拒答得体度、品牌语气，Agent 无法在看不见的维度上做权衡，便稳定地拿这些维度换分数——加长 prompt 换覆盖率、加规则换边界情况。这类退化只在上线后以成本或体验的形式显现，而此时归因已很困难。

Badcase 过拟合是结构性的，不是操作失误。评测集由每周真实 badcase 整理而来，是历史失败分布的一个有偏样本；Round 1 的领域外修复能生效，只因为那类 badcase 已被采样进来，未入集的失败模式连被看见的机会都没有。这正是"从小循环到大循环"的真实动机：把采样偏差本身也交给系统处理。

爆炸半径由权限而非模型决定。Agent 持有部署权限，闸门一过即具备上生产的能力；案例中第 4 轮仍"通过人工复核上线"，人工复核就是最后的爆炸半径限制器。理性边界是：可回滚、可观测、影响面有界的动作（prompt、配置、灰度实验）可以全自动，不可逆或影响面不可估的动作（数据迁移、权限变更、全量发布）必须保留人类 gate。

## 关联

- 同题异语种孪生页：[[entities/harness-工程实践如何让-agent-完成自主迭代]]（归并候选，提案卡 #11 批1）
