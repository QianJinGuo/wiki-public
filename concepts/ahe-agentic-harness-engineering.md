---
title: AHE — Agentic Harness Engineering
created: 2026-05-07
updated: 2026-08-29
type: concept
tags: [harness-engineering, research, agent-evolution, observability, fudan, peking-university, autonomy-spectrum, failure-modes, oversight]
related:
  - [[entities/agent-engineering-principles-architecture-practice|Agent 工程实践]]
  - [[entities/claude-code-core-internals|Claude Code]]
sources: ['raw/articles/fudan-peking-ahe-agentic-harness-engineering']
review_value: 8
review_confidence: 9
review_recommendation: strong
review_stars: 5
confidence: high
---
# AHE — Agentic Harness Engineering

> **AHE = Agentic Harness Engineering**：让进化 Agent 自动迭代优化 Coding Agent 的 Harness 的工程框架，核心洞察是**可观测性是进化循环的瓶颈而非 Agent 能力**。^[raw/articles/fudan-peking-ahe-agentic-harness-engineering.md]

## 概述

AHE（Agentic Harness Engineering）由复旦+北大联合提出，通过三大可观测性支柱让进化 Agent 自动迭代优化 Coding Agent 的 Harness。核心反直觉判断：进化 Agent 稳定优化 Harness 的瓶颈不在 Agent 能力，而在**可观测性**。

论文：[arXiv:2604.25850](https://arxiv.org/pdf/2604.25850)

## 核心洞察

> 进化 Agent 稳定优化 Harness 瓶颈，不是因为 Agent 不够聪明，而是因为整个进化循环缺乏可观测性。

传统 Harness 设计是"手工作坊模式"：开发者阅读海量轨迹日志，识别失败模式，手动修改 Prompt 或工具。随着 Base Model 快速迭代，这种手动循环已无法跟上模型进化速度。

## AHE vs 古典 Harness：根本区别

| 维度 | 古典 Harness | AHE（Agentic Harness Engineering） |
|------|-------------|-------------------------------------|
| **迭代方式** | 人工手动：人读日志→人找问题→人改配置 | 自动进化：进化 Agent 读证据→自声明预测→系统验证 verdict |
| **可观测性** | 无结构：海量轨迹是"噪音海洋" | 三支柱：组件/经验/决策三层可观测性 |
| **编辑约束** | 软约束：依赖人自觉不作弊 | 硬约束：运行目录/验证器/LLM 配置只读，种子 System Prompt 不可删除 |
| **失败追溯** | 人工事后分析 | 自声明 Manifest + 交集验证，可证伪 |
| **组件粒度** | Prompt 大一统 | 7 种正交组件类型（文件级解耦），每个独立版本控制 |
| **跨模型迁移** | Prompt 往往脆弱 | 编码通用协调模式，5 个基座全部正向增益 |

古典 Harness 的核心假设是**人担任控制器**：人负责感知环境变化、人负责决策、人负责执行调整。AHE 的核心假设是**可观测性担任控制器**：系统提供结构化的观测信号，进化 Agent 在约束内自动搜索更好的配置。

[[entities/agent-engineering-principles-architecture-practice|Agent 工程实践]] 指出"Harness 比模型更关键"——AHE 在此基础上进一步指出，在自动进化场景下，**可观测性比 Harness 本身更关键**。

## 自主性光谱

AHE 的设计隐含了一条从"全控制"到"全自主"的光谱：

```
全控制                                          全自主
  |                                              |
  v                                              v
人工 Harness → 参数化 Harness → AHE 自动进化 → 完全自主 Agent
   人在回路     人配置参数      弱监督进化         无需人
```

### 光谱位置：AHE 处于"弱监督进化"区间

AHE 既不是完全人工的古典 Harness，也不是完全自主的 Agent 自动调参。它的自主性体现在：

- **进化 Agent 自主决定编辑哪些组件**：进化 Agent 根据轨迹证据自行决定修改哪个 Harness 文件
- **但编辑范围受硬约束限制**：运行目录、验证器、LLM 配置均只读；种子 System Prompt 不可删除
- **每轮编辑附带自声明预测**：进化 Agent 预测自己的改动会修复哪些任务、引发哪些回归
- **系统给出 verdict 而非人给出**：预测集合与真实 delta 做交集，由数据而非人判断编辑有效性

### 与光谱上其他位置的对比

- **人工 Harness**（全控制）：Claude Code 的默认模式——[[entities/claude-code-core-internals|Claude Code]] 通过  和  让工程师深度参与 Harness 迭代，人是最终决策者
- **参数化 Harness**：OpenAI Codex 的 Harness 设计——大量参数可配置，但参数空间由人定义，模型不自主修改 Harness
- **AHE 自动进化**：进化 Agent 在约束内自主搜索组件空间，每个编辑附带可证伪的预测
- **完全自主 Agent**：端到端 Agent（如 AutoGPT 类系统），Harness 仅提供工具集，不约束编辑边界

[[entities/tencent-ai-team-knowledge-mgmt-harness-moat|腾讯 AI Team 的知识沉淀实践]] 揭示了另一条光谱：**知识是 Harness 的真正护城河**。在 AHE 的框架下，知识管理不是附属能力，而是持续进化的燃料——Long-term Memory 组件贡献 +5.6 pp，是所有组件中单独贡献最大者。

## 三大可观测性支柱

### 1. 组件可观测性：文件级解耦 Harness

基于 NexAU 框架，将 Harness 显式解耦为 **7 种正交组件类型**（每个都是独立文件）：

| 组件类型 | 说明 |
|----------|------|
| System Prompt | 塑造工作风格与推理策略 |
| Tool Description & Implementation | 工具接口和实现 |
| Middleware | 上下文控制、执行编排、故障恢复 |
| Skill | 可复用工作流 |
| Sub-agent Configuration | 任务委托配置 |
| Long-term Memory | 跨会话持久经验 |

关键设计：**种子 Harness 刻意极简化**（只有一个 shell 执行工具，无中间件、无技能），迫使后续每个组件都必须靠实测数据"挣"到自己的位置。每个逻辑编辑对应一次 git commit，天然支持文件级 diff 和回滚。

[[entities/claude-code-core-internals|Claude Code]] 的  机制与 AHE 的组件可观测性形成鲜明对照：Claude Code 的 System Prompt 是运行时动态组装的（六层优先级），而 AHE 的 System Prompt 是进化 Agent 可编辑的组件文件。前者是框架设计，后者是进化目标。

### 2. 经验可观测性：分层蒸馏轨迹证据

原始轨迹是数百万 token 的"噪音海洋"。AHE 引入 Agent Debugger 框架，输出两层报告：
- **Per-task Analysis**：每个任务的根因分析（成功/失败模式）
- **Benchmark-level Overview**：聚合所有任务的全局概览

这种**渐进式披露（Progressive Disclosure）** 既省 token，又保证决策有据可依。

这与 [[entities/agent-engineering-principles-architecture-practice|Agent 工程实践]] 中提到的"三层可观测性"（完整 Trace 同步落盘 + 人工抽样标注校准 + LLM 全量评估）形成呼应，但 AHE 的经验可观测性更侧重于**机器可读的证据结构**，而非人工审查流程。

### 3. 决策可观测性：可证伪的编辑契约

进化 Agent 每轮决定增删改哪些组件，AHE 对编辑施加两道约束：

**可控性**：进化 Agent 只能在 Harness 工作区内写入，运行目录/验证器/LLM 配置均为**只读**，种子 System Prompt **不可删除**——防止走捷径（禁用验证器、换更强模型）。

**自声明预测**：每个编辑附带 Manifest 记录：
- 失败证据推断的根因
- 目标修复方案
- 预测影响（预期修复哪些任务 + 可能引发哪些回归）

下一轮 rollout 后，系统将预测集合与真实任务级 delta 做交集，给出每个编辑的 verdict（确认有效 / 回滚）。

## Agentic 场景独有 failure modes

AHE 的核心发现之一是**进化 Agent 能可靠地知道自己要修什么，但无法预见自己的改动会搞坏什么**。这揭示了 Agentic Harness 与传统 Harness 截然不同的 failure 模式：

### 1. 回归预见失败（Regression Blindness）

| 指标 | AHE | 随机基线 |
|------|-----|----------|
| Fix Precision | 33.7% | 6.5% |
| Fix Recall | 51.4% | 10.6% |
| Regression Precision | 11.8% | 5.6% |
| Regression Recall | 11.1% | 5.4% |

进化 Agent 定位修复目标的精度约 **5 倍优于随机**，但回归预测仅略高于随机（2 倍）。这意味着在 Agentic 进化场景下，**副作用是最难预测的 failure mode**——编辑往往在修复任务 A 的同时悄悄破坏了任务 B，而进化 Agent 事前无法察觉。

### 2. 组件间非加性交互（Non-additive Component Interaction）

三个正项组件（Memory +5.6 pp、Tools +3.3 pp、Middleware +2.2 pp）相加共 +11.1 pp，但完整 AHE 只提升 +7.3 pp。堆叠后在 Hard 任务上造成**冗余重试**，消耗长程预算。

这是 Agentic 设置独有的 failure mode：在人工 Harness 中，人可以感知到"我已经加了三个东西，它们可能有冲突"；进化 Agent 则可能简单地堆叠所有正向组件，导致 1+1+1 < 3 的效果。

### 3. 跨任务噪声迁移（Cross-task Noise Transfer）

ACE 和 TF-GRPO 的 Prompt 级注入在跨任务时变成"昂贵噪音"——在一个任务上有效的 Prompt 调整，迁移到另一个任务时不仅无效，还消耗 token 预算。这种 failure mode 存在于 Agentic 设置中，而在单任务人工 Harness 中不存在。

### 4. 进化循环崩溃（Evolution Loop Collapse）

当可观测性不足时，进化 Agent 会在噪音海洋中过拟合：基于噪声轨迹做出错误根因判断，编辑后产生更差的 Harness，形成恶性循环。AHE 的三大支柱正是为了解决这个问题——但即使有三大支柱，Regression Blindness 仍然存在。

### 5. 任务饱和效应（Task Saturation Effect）

AHE 的实验显示：离饱和越远的模型增益越大（DeepSeek-v4-flash +10.1 pp vs GPT-5.4 medium +2.3 pp）。当模型能力接近饱和时，Harness 优化的边际收益急剧下降——这对于在人工设置下工作良好的"继续调优"策略是反直觉的。

## 连续监督模式

AHE 的连续监督不是"人盯着 Agent 工作"，而是**结构化的验证循环**：

### 编辑契约模式

```
进化 Agent 编辑 Harness
        ↓
生成自声明 Manifest（根因 + 修复方案 + 预测影响）
        ↓
系统执行 rollout
        ↓
预测集合 ∩ 真实 delta → Verdict
        ↓
有效 → 保留；无效/回归 → 回滚
```

这个模式的关键是**可证伪性**：进化 Agent 的每个编辑都必须产生可被数据否决的预测，而非开放式优化。在 [[entities/claude-code-core-internals|Claude Code]] 的语境下，这类似于 ——在执行前设置检查点，但 AHE 的检查点是预测-验证闭环，而非人工审批。

### 分层渐进披露模式

连续监督的另一层是**渐进式披露**：

1. **Benchmark-level Overview**：全局概览，告诉进化 Agent"整体趋势是上升/下降"
2. **Per-task Analysis**：任务级根因，告诉进化 Agent"哪些任务在变好/变差"
3. **Raw Trajectory**（按需）：仅在两层报告不足以定位问题时，才深入原始轨迹

这与 [[entities/agent-engineering-principles-architecture-practice|Agent 工程实践]] 中"评测系统先修再改 Agent"的原则一致——AHE 的渐进式披露本身就是一种监督效率优化，避免进化 Agent 在噪音中迷失。

### 约束优先于监督模式

AHE 的监督不是"让进化 Agent 少犯错"，而是**通过硬约束让某些错误物理上不可能发生**：
- 运行目录只读 → 进化 Agent 不可能通过修改被测代码来作弊
- 验证器只读 → 进化 Agent 不可能通过禁用断言来作弊
- LLM 配置只读 → 进化 Agent 不可能通过换更强模型来作弊
- 种子 System Prompt 不可删除 → 进化 Agent 不可能通过删掉核心约束来作弊

这与 [[entities/claude-code-core-internals|Claude Code]] 的  设计哲学一致——通过系统层面的约束（而非模型层面的期望）来保证安全。 ^[raw/articles/claude-code-source-deep-dive-warrior.md]

[[entities/tencent-ai-team-knowledge-mgmt-harness-moat|腾讯知识管理实践]] 补充了另一维度的连续监督：**知识成熟度衰减机制**。proven 条目 12 个月未引用自动降级，verified 6 个月未引用自动降级——这本质上是时间维度的连续监督，防止知识库腐化。类似地，AHE 的 Verdict 机制也是一种即时性的连续监督，在每一轮编辑后立即给出判断。

## 自主性 vs 可靠性权衡

AHE 揭示了一个根本性的权衡：**进化 Agent 的自主性越强（能改更多东西），可靠性风险越高（预测不到副作用）**。

| 自主等级 | 可靠性风险 | 适用场景 |
|----------|-----------|----------|
| 人工 Harness（0% 自主） | 最低（人在回路） | 高风险任务、首次部署 |
| 参数化 Harness（10-30% 自主） | 低（参数空间受限） | 稳定任务、微调场景 |
| AHE 进化（30-60% 自主） | 中（Verdict 兜底） | 自动迭代、中等规模优化 |
| 完全自主 Agent（100% 自主） | 高（无约束） | 探索性研究、低风险场景 |

AHE 选择 30-60% 自主区间的原因：**完全自主的搜索空间太大，副作用不可控；完全人工的迭代速度太慢，跟不上模型进化**。Verdict 机制和硬约束是这个权衡的工程实现。

## 实验结果

### Terminal-Bench 2（10 轮迭代，约 32 小时）

| 方法 | pass@1 |
|------|--------|
| **AHE** | **77.0%** |
| Codex-CLI（人类设计） | 71.9% |
| TF-GRPO | 72.3% |
| ACE | 68.9% |
| 种子 Harness | 69.7% |

### 跨基准迁移（SWE-bench-verified）

AHE 在 SWE-bench-verified 上取得**最高整体成功率**，且比种子少用 **12% token**。ACE 和 TF-GRPO 的 Prompt 级注入在跨任务时变成"昂贵噪音"。

### 跨模型迁移

将进化后的 AHE Harness 直接套用到 5 个不同基座上，全部取得**正向增益**：

| 模型 | 增益 |
|------|------|
| GPT-5.4 medium | +2.3 pp |
| GPT-5.4 high | +7.3 pp |
| GPT-5.4 xhigh | +2.3 pp |
| Gemini-3.1-flash-lite | +5.1 pp |
| DeepSeek-v4-flash | +10.1 pp |
| Qwen-3.6-plus | +6.3 pp |

规律：**离饱和越远的模型，增益越大**。说明 AHE 编码的是通用协调模式，而非特定模型的"提示词玄学"。

## 组件消融分析

| 组件 | 单独贡献 |
|------|----------|
| Long-term Memory | **+5.6 pp** |
| Tools | **+3.3 pp** |
| Middleware | **+2.2 pp** |
| System Prompt | **-2.3 pp**（回归） |

三个正项相加 +11.1 pp，但完整 AHE 只提升 +7.3 pp——原因是组件间非加性交互：堆叠后在 Hard 任务上造成冗余重试，消耗长程预算。

## 自归因可靠性

| 指标 | AHE | 随机基线 |
|------|-----|----------|
| Fix Precision | 33.7% | 6.5% |
| Fix Recall | 51.4% | 10.6% |
| Regression Precision | 11.8% | 5.6% |
| Regression Recall | 11.1% | 5.4% |

进化 Agent **能可靠地知道自己要修什么**（Fix targeting 约 5 倍优于随机），但**预见不到自己的改动会搞坏什么**（Regression 预测仅略高于随机）。这是 AHE 当前最大的局限。

## 关键结论

1. **可观测性是瓶颈**：不是 Agent 不够聪明，而是进化循环缺乏结构化上下文和清晰动作空间
2. **多组件联合优化优于单组件**：Memory/Tools/Middleware 联合优化的增益远超单独编辑 Prompt
3. **通用协调模式可迁移**：AHE 编码的是跨模型通用的行为协调模式，不是模型-specific 的提示词
4. **回归预见是未来方向**：当前最大局限是无法预见编辑的副作用
5. **自主性光谱存在最优区间**：30-60% 自主性在迭代速度和可靠性之间取得平衡

## 深度分析

### 可观测性驱动的范式转移：从「模型中心」到「系统中心」

AHE 最深刻的洞察是揭示了 Coding Agent 性能提升的主要矛盾已经转移。在模型能力快速迭代的当下，Harness 作为「中介层」成为决定性能天花板的更关键因素——而 Harness 的瓶颈不在于设计不够精巧，而在于**缺乏让进化 Agent 有效工作的可观测性基础设施**。

传统观点认为要提升 Agent 性能，应该：
- 使用更强大的模型
- 精心设计更复杂的 Prompt
- 不断增加工具数量

AHE 用实验数据反驳了这套逻辑：System Prompt 在消融实验中产生 -2.3 pp 回归，单独增加工具或中间件的效果远小于联合优化。这说明 Agent 性能是由**整个系统的协同效率**决定的，而非某个组件的精巧程度。

### 进化 Agent 的能力边界：知道修什么 vs 知道改坏什么

论文最反直觉的发现是进化 Agent 的「自归因可靠性」高度不对称：

- **Fix Precision 33.7%** vs 随机 6.5%（5 倍优势）→ 进化 Agent 能较准确地定位需要修复的目标
- **Regression Precision 11.8%** vs 随机 5.6%（2 倍优势）→ 但预测副作用的能力仅略高于随机

这意味着进化 Agent 在 AHE 框架下是一个**「精准定位问题、但对副作用盲目」**的优化器。其根本原因在于：修复目标来自对历史轨迹的观察（可以穷举分析），但副作用需要在所有未发生的执行路径上验证（不可能穷举）。

这直接解释了为什么 AHE 选择「约束优先于监督」的设计哲学：与其让进化 Agent 学着预测副作用（几乎不可能），不如用硬约束物理上禁止某些类别的副作用（例如禁止修改验证器、禁止删除核心约束）。

### 组件非加性交互：系统复杂性的隐蔽成本

三个正向组件（Memory +5.6 pp、Tools +3.3 pp、Middleware +2.2 pp）相加 +11.1 pp，但完整 AHE 只 +7.3 pp，缺口 -3.8 pp。这种**组件间非加性交互**揭示了 Agent 系统与传统软件系统的根本区别：

传统软件工程中，组件的边际收益大致可叠加（Module A 提升 X%，Module B 提升 Y%，两者联合大致提升 X+Y%）。但在 Agent 系统中，组件之间通过 LLM 的上下文机制产生**非线性交互**：

- Memory 提供历史经验 → LLM 基于记忆上下文生成策略
- Tools 改变执行能力 → LLM 的工具调用模式影响 Memory 的使用频率
- Middleware 控制执行流程 → 影响上下文结构，进而影响 Memory 的检索效果

三者叠加后，LLM 的上下文压力增加，在 Hard 任务上产生冗余重试，消耗本该用于成功执行的「长程预算」。

这与 Claude Code 的「四级记忆系统」设计形成对照：Claude Code 的记忆系统是分层递进的（项目级 → 仓库级 → 会话级 → 操作系统级），每一级都有明确的用途和容量边界；而 AHE 的 Long-term Memory 是进化后习得的组件，缺乏这种先验的结构性约束。

### 任务饱和效应：模型进化对 Harness 工程的深远影响

实验数据显示 DeepSeek-v4-flash（+10.1 pp）增益最大，而 GPT-5.4 medium（+2.3 pp）增益最小。规律是：**离饱和越远的模型，Harness 优化的边际收益越大**。

这意味着：
1. 当模型能力快速增长时，Harness 优化是「追模型」的手段
2. 当模型能力接近饱和时，Harness 优化的边际收益急剧下降
3. 未来的模型如果持续进化，Harness Engineering 的重要性可能随之变化

对于从业者的实际意义：当团队选型新模型时，应该关注「模型当前处于饱和曲线的哪个位置」，而非单纯比较绝对性能。如果一个模型已经接近饱和，继续投入 Harness 优化的回报有限；如果模型还有大幅提升空间，Harness 工程的价值更高。

### 编码通用协调模式 vs 模型特定提示词

AHE 进化后的 Harness 能零样本迁移到 5 个异构模型家族并全部取得正向增益，这有力地证明了 AHE 编码的是**跨模型通用的行为协调模式**，而非模型特定的「提示词玄学」。

这与 Claude Code 的设计哲学一致：Claude Code 的 System Prompt 不是在「教模型怎么做」，而是在「给模型提供一个结构化的执行框架」。真正的智能来自模型本身，Harness 的作用是将模型能力引导到正确的方向上。

## 实践启示

### 1. 构建可观测性基础设施是 Agent 工程的第一步

AHE 最重要的实践建议不是「如何设计更好的 Prompt」，而是**「如何让系统可观测」**。在着手优化 Harness 之前，团队应该优先建设：

- **轨迹日志基础设施**：完整记录每一次 Agent 执行的全流程，包括工具调用、中间状态、上下文大小等
- **分层证据蒸馏**：不要让开发者直接阅读原始轨迹，而是通过 Agent Debugger 框架输出「Per-task Analysis」和「Benchmark-level Overview」
- **组件级追踪**：能够将失败模式追溯到具体的 Harness 组件（是 Prompt 问题？还是工具描述问题？还是中间件逻辑问题？）

### 2. 从极简种子开始，让数据决定组件必要性

AHE 的种子 Harness 刻意极简化（只有一个 shell 执行工具），这背后的工程哲学是**「不要预设组件的价值，让实战数据证明」**。

实践建议：
- 新项目不要一开始就设计「完美」的 Harness，包含所有可能的中间件和技能
- 从最小可用配置开始，通过实际任务失败数据来决定添加什么组件
- 每个新组件的引入都应该有明确的「预期收益」和「可衡量的验证方式」

### 3. 用硬约束替代软规范，防止进化过程走捷径

AHE 的硬约束设计（运行目录只读、验证器只读、LLM 配置只读、种子 System Prompt 不可删除）揭示了一个重要的工程原则：**「期望靠不住的地方，要用系统约束替代」**。

实践建议：
- 定义清晰的「禁区」：Agent 不应该修改的配置文件、不应该访问的 API、不应该删除的日志
- 约束要足够具体：「不要作弊」是软约束；「验证器目录只读」是硬约束
- 硬约束要覆盖所有「走捷径」的路径，包括那些看起来「合理」的路径

### 4. 建立「预测-验证」闭环，让每一轮编辑都可证伪

AHE 的编辑契约模式（自声明 Manifest → rollout → 交集验证 → verdict）是一个高度可复用的工程模式，适用于任何需要迭代优化的 Agent 系统。

实践建议：
- 每次 Harness 修改都要记录「预期修复哪些任务」和「预期引发哪些回归」
- 通过客观指标（任务成功率、token 消耗）而非主观评估来判断编辑有效性
- 建立回滚机制：预测失败的编辑应该被自动或半自动回滚

### 5. 组件设计要正交，避免非加性交互

组件间非加性交互的发现提醒我们：Harness 组件应该是**正交的**——每个组件解决不同维度的问题，且组件之间不应该通过共享状态产生耦合。

实践建议：
- 设计组件时，明确每个组件的「职责边界」
- 如果两个组件经常需要「联合调参」才能工作，这通常是耦合的信号
- 优先让每个组件独立验证其有效性，再验证联合效果

### 6. 建立跨模型泛化的评测集

AHE 能证明「通用协调模式」而非「模型特定提示词」的关键，是跨模型评测。实践建议：
- 不要只在单个模型上评测 Harness
- 建立包含至少 3-5 个不同模型家族的评测集
- 如果一个优化只在特定模型上有效，需要谨慎判断其泛化能力

### 7. 关注任务饱和度，指导优化资源分配

任务饱和效应意味着：对于已经接近性能饱和的模型，继续投入 Harness 优化的回报有限。

实践建议：
- 定期评估「模型当前性能 vs 理论上限」，判断还有多少优化空间
- 如果模型接近饱和，将资源转向其他方向（数据质量、任务设计、领域适配）
- 如果模型还有较大提升空间，Harness 工程是性价比高的优化手段

## 相关概念与实体

- [[concepts/harness-engineering-framework|Harness Engineering 框架]] — Harness 的系统化设计思路，与 AHE 构成研究对照
- [[entities/agent-self-improvement-six-mechanisms|Agent 自我改进六条路]] — AHE 属于其中的"进化搜索"机制
- [[entities/langchain-anatomy-agent-harness|LangChain Agent Harness]] — Harness 在 LangChain 中的具体实现
- [[entities/agent-memory-architecture|Agent Memory 架构]] — AHE 中 Long-term Memory 是最大贡献组件（+5.6 pp）
- [[raw/articles/harness-engineering-systematic-explainer|Harness Engineering 系统梳理]] — 复旦+北大的 AHE 与此构成研究对照
- [[entities/claude-code-core-internals|Claude Code 源码核心机制]] — AHE 组件可观测性设计 vs Claude Code 动态 System Prompt 的对照
- [[entities/agent-engineering-principles-architecture-practice|Agent 原理架构与工程实践]] — "Harness 比模型更关键"原则与 AHE 可观测性驱动进化的呼应
- [[entities/tencent-ai-team-knowledge-mgmt-harness-moat|腾讯 AI Team 知识沉淀实践]] — 知识作为 Harness 护城河与 AHE Long-term Memory 组件的直接关联
- [[entities/agent-harness-architecture|Agent Harness 架构]] — Harness 在 Agent 系统中的架构定位
- [[entities/harness-production-agent-engineering-deficit|Harness 如何支撑 Agent 在生产环境稳定运行]] — 生产环境 Harness vs AHE 研究框架的对照

## 背景

论文标题：*Agentic Harness Engineering: Observability-Driven Automatic Evolution of Coding-Agent Harnesses*（arXiv:2604.25850）

## 新增关联实体
- [[entities/thought-aligner-shanghai-fudan-icml-2026]]
- Trump Media S Q1 Loss Widens To Usd406 Million On Bitcoin Cro Markdowns

## 关联实体

**上游依赖**:
- [[entities/agent-engineering-principles-architecture-practice]] — 提供基础理论/方法
- [[entities/claude-code-core-internals]] — 提供基础理论/方法
-  — 提供基础理论/方法

**下游应用**:
-  — 具体应用场景
- [[entities/agent-engineering-principles-architecture-practice]] — 具体应用场景
- [[entities/claude-code-core-internals]] — 具体应用场景

**平行协作**:
- [[entities/langchain-anatomy-agent-harness]] — 替代/补充方案
- [[entities/agent-memory-architecture]] — 替代/补充方案
- [[entities/claude-code-core-internals]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
