---
title: "MineExplorer：开放世界多模态评测基准"
created: 2026-08-18
updated: 2026-11-22
type: entity
tags: [ai, agent, benchmark, multimodal, evaluation, world-model, long-horizon, meituan, video-generation]
sources: [raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026]
confidence: 0.75
provenance_state: extracted
score: 72
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# MineExplorer：开放世界多模态评测基准

## 核心洞察

多模态大模型能看懂图像、解析视频、在复杂场景里做静态推理，但一旦被放进一个**实时变化、需要持续探索的开放世界**（如《我的世界》），其真实能力出现系统性断层——这是静态 benchmark（截图问答）掩盖的长程能力短板。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md]

## 评测设计

美团 LongCat 团队构建了 **MineExplorer**——首个在开放世界中做到**分钟级长程任务**的评测基准，系统评测多模态大模型在需要**长程规划**、且包含**隐藏前置条件**（如先砍树才能获得木头、先熔炼才能得到工具）的任务中的真实能力。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md]

- 与静态截图问答的区别：模型必须在环境中持续执行、感知反馈、修正规划
- 任务结构：长时间跨度、多步骤因果链、隐含前置条件
- 目标：暴露"温室"（静态数据集）与"开放世界"（动态执行）之间的能力落差

## 与现有评测体系的关系

MineExplorer 属于 agent/能力评测家族的新分支——面向**执行型长程任务**而非单轮问答。它与 [[entities/meituan-longcat-vitabench-20-long-term-dynamic-agent-benchmark|VitaBench 2.0]]（长期动态智能体基准）、Agent 评测基准、[[entities/从月球漫步到赛博都市wbench测出了世界模型的边界|WBench 世界模型边界]] 同属"更贴近真实执行环境"的评测转向。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md]

## 深度分析

### 隐藏前置条件才是真正的难度来源：hop 分级测的是依赖图深度

MineExplorer 对"难"的定义不在任务字面长度，而在隐藏前置依赖的层数。每个复合任务被形式化为四元组 τ=(q, s₀, Gτ, Mτ)：自然语言指令、初始状态、任务间依赖图（DAG）与规则化里程碑检查器，其中最关键的设计是**指令 q 刻意不枚举依赖图中的全部节点**，智能体必须从环境交互中自行推断那些没有写在任务描述里的前置子任务。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:51-60] 评测结果印证了这一定义的杀伤力：最强模型 Claude-Opus-4.6 从 1-hop 任务的 77 分一路跌至 4-hop 任务的 12 分，每增加一层隐藏前置依赖就掉一个台阶。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:105-107] 由于任务实例每次运行 1800 个环境步、每步 0.1 秒、对应 3 分钟连续交互且世界状态随每个动作实时更新^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:45-47]，隐藏前置的发现必须在动态执行中完成，无法靠离线分析指令穷举——失分点集中在"发现未说明步骤"这一推理环节，而非步骤本身难以执行。

### 知识解耦：把"背 Minecraft Wiki"从"会探索"中剥离出来

MineExplorer 用 LLM 裁判对每个原子任务做归类：判断其表现主要依赖通用世界常识还是 Minecraft 专有机制，并把后者主动过滤掉。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:64-66] 这一设计直接回应了以往游戏类基准的通病——分数可能被训练语料中恰好覆盖的攻略知识刷高，测出来的是"背题"而非"探索"。解耦之后，MineExplorer 度量的是在动态物理世界中自主探索的通用能力，这与 [[entities/gemini-agents-minecraft-long-horizon-collaboration-2026|Gemini Minecraft 长程协作基准]]、[[entities/triworldbench-three-view-embodied-world-model-benchmark-2026|TriWorldBench]] 等新一代评测共同体现的"把领域知识从能力测量中剥离"的方法论转向一致，也与 [[entities/world-model-evaluation-position-paper-nju-2026|世界模型评测立场论文]] 对评测效度与任务真实性的关切相呼应。换句话说，MineExplorer 测的不是"AI 会不会玩 Minecraft"，而是"AI 能不能在一个动态物理世界里自主探索"——这让它的榜单分数可以直接解读为探索能力的基线，而非领域知识熟练度。

### 失败解剖：感知-推理剪刀差与近 60% 的导航失败

18 个来自八大模型家族的顶级模型集体考砸，最好的 Claude-Opus-4.6 整体任务成功率（TSR）也只有 41 分。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:97-103] 几乎所有被测模型都呈现同一梯度：感知分 > 行动分 > 推理分（Claude-Opus-4.6 感知 61.91、推理 54.71）——瓶颈不是"看不见"，而是看懂了却无法把观察串联成有效策略。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:113-115] 基准本身借鉴 ReAct 范式，把开放世界探索拆成感知、推理、行动三大维度共 14 项细粒度能力，空间感知、移动、采集等基础能力出现频率最高^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:80-88]，因此失败模式可以精确定位到具体能力项：失败归因中近 60% 是导航失败，即模型在最基础的"走到目标跟前"这一物理执行层就已大量失分。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:119-121] 这把"长程能力短板"落到了可操作的层面：问题首先是感知与行动的闭环断裂，其次才是抽象规划。

### 资源消融：加步数、加记忆都不是解药，瓶颈在状态对齐

消融实验否定了两种最直觉的补救方案。步数维度：能解的任务模型在早期步数就已解出，解不了的给到 1800 步上限照样解不了；记忆维度：历史画面帧数加到一定程度后性能反而下降，因为过期的观察会干扰对当前局面的判断。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:125-141] 作者据此把核心结论落在"瓶颈不在资源，而在模型没法把已有信息和当前世界状态对齐"上^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:143]——这与 [[concepts/agent-memory-architecture|agent 记忆架构]] 领域"更多上下文不等于更好决策"的工程经验互相印证，也提示改进方向在状态跟踪与上下文管理机制（参见 [[concepts/context-management-agent-systems]]），而非单纯堆推理算力或拉长上下文窗口。

## 实践启示

1. **评测多模态/具身模型时按 hop 数分级报分，不要只看整体均值**：1-hop 高分完全可以伴随 4-hop 崩盘（77→12），隐藏前置条件推理才是真实短板的显影剂。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:105-107]
2. **自建基准时做知识解耦过滤**：用 LLM 裁判剔除依赖领域专有知识的题目，否则榜单测量的是语料覆盖度而非探索能力，评测结论会被"背题效应"污染。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:64-66]
3. **造题瓶颈交给多智能体合成流程**：五个分工 Agent 在群聊中协作、orchestrator 控制发言顺序、经初始化与辩论两阶段修订，把题目有效率拉高约 30 个百分点、质量分提升约 0.5，且在最难的 4-hop 任务上优势最大——813 个人工验证实例证明该流程可复用。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:70-76]
4. **排查长程 agent 失败先查执行闭环，再怀疑推理**：近 60% 的失败源于导航（走不到目标），而加步数与加记忆帧均被消融证明无效——优先修复状态对齐与上下文管理，而不是增加资源预算；这一归因方法（失败类型分布 + 维度分数 + 消融交叉验证）本身也值得复制到任何长程 agent 系统的调试流程中。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:119-143]
5. **评测、造题、训练共用同一套基础设施**：MineExplorer 的 Minecraft 沙盒既是考场也是练兵场，开源包内含 813 个 1-hop 到 4-hop 的人工验证实例、规则化里程碑自动评测框架与多智能体合成任务的完整代码。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:26-32,155]
6. **对具身智能的乐观情绪应该用"规划层"标准重新校准**：感知层基本就绪不等于能行动，受控场景跑通单步指令不足以宣布 AI 能行动；把瓶颈定位清楚比盲目乐观更有价值，MineExplorer 提供的正是一条诚实、可量化的能力基线。^[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026.md:149-162]

## 价值

- 提供了超越静态 VQA 的**动态世界评测范式**，可迁移用于 video-understanding / agentic / world-model 类模型的执行能力评估
- 揭示多模态模型在长程规划与隐含前置条件推理上的真实短板

→ [[raw/articles/mineexplorer-开放世界多模态评测基准-longcat-2026|原文存档]]
