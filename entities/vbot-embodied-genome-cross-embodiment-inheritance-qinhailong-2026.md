---
title: "Vbot 具身基因组：跨本体智能继承（维他动力 秦海龙）"
created: 2026-08-22
updated: 2026-11-08
type: entity
tags: [embodied-ai, robot, cross-embodiment, world-model, multi-embodiment, vla, reinforcement-learning]
sources: [raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Vbot 具身基因组：跨本体智能继承（维他动力 秦海龙）

## 概述

维他动力（Vbot）在 WRC 上首次公开人形机器人 Vbot ATOM 并开启预订，同时提出「Vbot Embodied Genome｜具身基因组」技术体系，核心命题是：机器人换了一副「身体」，能否「继承」已经学会的智能。该框架把「智能跨任务继承、跨本体进化」作为通用机器人的关键，区别于行业常见的「一个模型覆盖更多任务」路线，转而追问「一套已形成的智能系统能否顺畅进入截然不同的物理身体」。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

ATOM 并非从零起步，它背后是已量产交付、走进真实用户生活的四足机器人「大头」及其积累的真实数据与迭代闭环。秦海龙（哈工大、新加坡国立、浦项科大背景，前蔚来自动驾驶高级总监、千里智驾首席科学家）于 2026 年 3 月加入维他动力任研发副总裁，负责世界模型、空间智能、Agent OS 及人形机器人研发。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

## 三大核心模型

维他动力公布三大核心模型，对应机器人智能链条的三个关键环节（理解人、预测未来、适应不同本体），连接成「前台持续交互 → 世界持续推理 → 本体持续进化」的完整闭环：^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

- **Vbot-OmniDuplex** — 多模态流式全双工交互与异步推理，负责「理解人」
- **Vbot-WorldModel** — 动作条件未来预测与策略评估，负责「预测未来」
- **Vbot-EvoMorph** — 共享策略骨干 + 多本体 Real-Sim-Real 进化，负责「适应不同本体」

## 跨本体智能继承的分层

针对「智能继承不能一概而论」，秦海龙提出按模块分层的继承能力：^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

- **交互层** — 可 100% 完全继承。机器人与人交互的理解、成功与失败案例与物理本体形态无关，四足积累的交互数据可直接迁移到人形
- **策略生成与空间理解层** — 高度复用。移动导航任务的空间动静态理解、避障、通行判断在四足与人形间差异不大；操作任务采用无本体（Egocentric / 第一人称视角）基模路线，复用大量公开与合成数据，真正的瓶颈在真机后训练数据
- **底层运控与强化学习层** — 「隐空间共通，末端分叉」。上层隐空间理解、环境感知、控制逻辑相通，下层因关节自由度、拓扑结构、运动学不同而输出分叉，需依赖特定本体的适配层与强化学习（RL）完成控制

## Real-Sim-Real 与数据稀缺

体系依托多本体 Real-Sim-Real 进化：策略共享骨干在仿真与真实世界间交替迭代，让一套智能逐步适配多种本体。文中强调具身智能领域真正稀缺的数据是**真机后训练数据**（real-machine post-training data），而非公开/合成基模数据——这决定了跨本体能力能否落地到具体物理身体。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

## 深度分析

### 三闭环架构：IRE、GEO、RSR

具身基因组不是一组僵化参数，而是一套「跨任务可继承、跨本体可表达、由真实物理反馈持续改写」的同源智能体系，其底层被组织为三条深度耦合的闭环。IRE 闭环（交互—推理—进化）让实时流式交互与异步深度推理并行，并把人机交互本身沉淀为系统进化数据；GEO 闭环（生成—评估—优化）让策略生成候选动作、世界模型推演并评估未来、训练与离线评估持续优化策略；RSR 闭环（真实—仿真—真实）用真实日志重建物理世界，在仿真中展开大规模对抗试错后再回流真机验证。三条闭环共享同一条架构原则：共享高层语义与空间理解，分化特定任务表征，独立底层本体控制——这正是它与「把不同构型机器人强行塞进同一套权重」路线的分野所在。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

### 世界模型的角色迁移：从「生成未来」到「用未来做决策」

文中对世界模型的批判性定位颇有辨识度：多数 Demo 关注生成画面是否逼真、运动是否符合物理规律，但对机器人而言更尖锐的工程问题是——生成的未来究竟能否直接指导下一步物理决策。Vbot-WorldModel 因此被定义为动作条件下的决策推演引擎，在「输入观测—输出动作」的黑盒映射中显式插入未来推演：策略网络先产出多个候选动作序列，世界模型逐一预测各候选的第一视角物理演进，按任务完成度、动作一致性、空间几何与碰撞安全性等维度给出价值评分，优化器据此筛选下发；训练阶段该评价信号直接反哺策略参数迭代。支撑这套能力的语料约为 770K+ 真实导航片段、3000+ 小时视频、约 5000 公里轨迹，覆盖道路、园区、社区、办公、家庭等场景。这一取向与 [[concepts/world-models|世界模型]] 研究「以预测服务决策」的思路同向，可与 [[entities/causalwm-causal-chain-of-thought-embodied-world-model-2026|CausalWM 因果世界模型]] 相互参照。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

### 「隐空间共通，末端分叉」的工程化：Action Token 与 Body Adapter

EvoMorph 把分层继承论落到了具体结构上：实时多模态感知、本体状态、任务指令与形态信息先进入统一时序编码器，构建通用的「环境—身体联合潜在状态」，并输出表达高层抽象策略意图（如靠近目标、减速避障、抬升姿态）的 Action Token；随后这些 Token 被分发至各本体的适配器，分别解算为四足的足端接触力与步态曲线、人形的髋膝踝联动与质心平衡、机械臂的末端轨迹与抓取力度。换言之，「跨本体」并非让不同形态共用底层控制参数，而是新本体完整继承感知理解、推演模型与任务表征，再经轻量适配层演化出属于自己的身体表达。这也解释了 ATOM 为何能在不到半年内从启动研发走到产品化——它不是从零起高楼，而是同源智能进入一副新人形躯体的高效验证；同一「共享骨干 + 分化适配」的思路亦见于 [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld 跨本体世界模型]]。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

### 数据稀缺的真问题：专家数据不等于问题数据

秦海龙对「操作数据稀缺」的再定义是全文最有洞察力的判断之一：遥操数据本质上是专家数据，只能告诉机器人在理想状态下应该怎么做；真正稀缺的是真实用户使用过程中产生的问题数据——动作为什么失败、环境为什么变化、用户为什么介入。这类反馈无法在实验室或仿真中凭空构造，只能靠终端进入真实场景持续运行来形成数据闭环。文中披露的四足「大头」运营数据（累计 126 万次大模型交互、23700 公里用户伴随里程、13.15 万小时陪伴、两周留存 86%、三个月内 4 次 OTA）被定位为数据飞轮运转的证据：交付到用户手中的终端既是持续服务的物理产品，更是捕捉物理世界真实缺陷的动态感知探针。这一判断与 [[entities/embodied-intelligence-sim-to-real-active-inference-behavior-tree-intrinsic-motivation-chenzhiyan-2026-06-17|Sim-to-Real 与主动推理]] 的讨论互为补充。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

### 行业坐标：「具身智能实验室」与「具身智能终端公司」的分化

文章结尾把行业分化为两种组织范式：实验室范式在受控环境下探索模型能力上限，回答「技术能不能做到」；终端公司范式必须直面真实物理世界的不确定性，回答「技术能否规模化量产、进入复杂生活空间并长期稳定服务」。维他动力自定位为后者，其壁垒论也由此展开：模型只是开始，终极比拼的是把技术、本体、产品和商业「打穿」的软硬件综合能力——终端必须能从 1 台、100 台滚动到 1000+ 台，才能沉淀出有价值的真机反馈。文章还提出一个新视角：大模型的 Scaling Law 围绕数据、参数、算力展开，具身智能正在加入一个新变量——身体；若每增添一种本体就重建一遍数据、模型与学习过程，行业便难成规模效应，而「上一副身体的经验成为下一副身体的起点」才是通用的可能路径。这一坐标划分对理解 [[concepts/embodied-intelligence-frontier|具身智能前沿]] 的格局颇有帮助。^[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026.md]

## 实践启示

1. **评估跨本体方案先做「继承分层」核对**：交互层可整体复用、策略生成与空间理解层高度复用、底层运控必须按本体分叉——面对任何「一套模型跑多种机器人」的宣称，先按这三层拆开验证，避免被整体性叙事误导。
2. **把真机问题数据当作一等资产**：遥操与演示数据只覆盖理想状态；应优先设计让真实失败（碰撞、打滑、用户介入）被日志捕获并回流训练的机制，而非单纯扩大遥操采集规模。
3. **以「参与决策」为世界模型的验收标准**：评估世界模型时不看生成画质，而看其价值评分能否实际改变动作选择并提升任务成功率——「生成—评估—优化」是否形成闭环是可操作的检验点。
4. **新本体开发走「共享骨干 + 轻量适配层」路线**：为每种构型从零重写运控的成本呈指数增长；可复用的是高层策略意图表征，需按本体重写的只有 Body Adapter 与 RL 适配，架构设计时应把这两部分显式隔离。
5. **把 Sim-to-Real Gap 从部署障碍转为迭代入口**：既然仿真与现实鸿沟恒在，可反向从真实日志做系统辨识并重建高保真仿真，在仿真内做大规模域随机化对抗，再回流真机暴露新边缘案例，进入下一轮迭代。
6. **商业化规模与数据闭环互为前提**：终端量产规模（1→100→1000+ 台）决定真机反馈的分布宽度；缺乏量产交付能力的团队难以构建具身智能的核心壁垒——评估合作方或公司时，应把软硬件交付能力纳入技术评估维度。

## 相关实体

- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- 机器人具身 AI
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld 跨本体世界模型]]
- [[entities/embodied-native-llm-embodied-intelligence-new-stage|具身原生 LLM]]
- [[entities/amap-abot-earth-0.5-3d-native-world-model|Amap Abot 3D 世界模型]]
- [[entities/baai-orca-next-state-prediction-world-model|BAAI Orca 世界模型]]

→ [[raw/articles/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026|原文存档]]
