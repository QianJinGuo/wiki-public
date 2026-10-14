---
title: "UrbanGround — 真实三维香港城市沙盒的具身导航评测（长程导航记忆崩塌）"
created: 2026-09-08
updated: 2026-10-14
type: entity
tags: [benchmark, embodied, navigation, spatial-ai, spatial-intelligence, multimodal, agent-eval, world-model, vlm]
sources: [raw/articles/urbanground-embodied-navigation-benchmark-2026]
confidence: 0.72
provenance_state: extracted
---

# UrbanGround — 真实三维香港城市沙盒的具身导航评测

## 概述

**UrbanGround** 是上海交通大学、新加坡国立大学、美团、香港中文大学、上海大学与牛津大学联合推出的**具身城市导航评测基准**——用香港真实三维地理数据搭建一座可连续行走的城市沙盒，让大模型以第一人称「走完一条路」而非只答对眼前问题。其核心发现是导航长程失败并非不会认路，而是「**走两步，就忘了路**」：短程导航最高成功率 75.0%，路线拉长后 10 个模型全部跌到 0%–3.8%。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 设计：把评测搬进可重复的城市沙盒

过去的城市空间智能评测多停在单张街景或航拍图，或沿预设节点切换视角，模型并未真正连续行走。UrbanGround 由地理、仿真、智能体三层组成。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

- **地理层**：以香港地政总署公开的 3D Visualisation Map 与 3D Pedestrian Network 为底图，将地理坐标、三维网格、步行网络对齐到同一套沙盒坐标。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]
- **仿真层**：建筑/墙体/坡道/地形起伏影响行动，同一地点可切换昼夜与天气，道路可途中封闭，行人沿步行网络移动，支持反复重跑同一条路线只改天气、封路或人流。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]
- **智能体层**：模型只见当前第一人称画面 + 任务说明，需要时可打开地图，再选择移动/转向/跳跃/操作地图；地图标出当前位置与目标但**不替它规划路线**。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

最终整理出 **810 个经人工检查的任务实例**，覆盖香港多个城区，用同一套动作接口测试 **10 个多模态模型**。论文从空间定位、持续记忆、动态调整三方面测「模型的判断能否一直管用」。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 关键发现 1：空间定位——认得出地标，分不清方向

10 个模型的视觉识别准确率 75.0%–93.8%，主动探索 46.3%–82.5%，但**方向理解只有 23.3%–58.3%**。Gemini-3.1-Pro 视觉识别 82.5%、方向理解却仅 23.3%（甚至低于随机猜测）——能认出地标，却在视角转动后难以判断其在左还是右。还有模型答对线索却走出路网（如 GPT-5.5 正确找出「东亚银行」，探索时却横穿马路离开步行区）。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 关键发现 2：持续记忆——走两步，就忘了路

短程导航 GPT-5.5 与 Claude-Opus-4.6 成功率均 75.0%；长程导航 10 个模型全部跌到 **0%–3.8%**（GPT-5.5 75→0%，Claude-Opus-4.6 75→1.3%）。每次转弯/过街/开关地图模型都要重新判断自己在哪、目标在哪，一个小偏差一路累积难以纠正。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

更关键的是**轨迹而非起点决定失败**：长程任务结束时各模型仍有 52.5%–81.3% 的轨迹比起点更接近目标，但完整成功率接近零——它们找对了大方向、走对了一段，最终仍卡在半路。GPT-5.5 的长程轨迹中 57.5% 比起点更接近目标，34.6% 将距离缩短至少 20%，但任务结束平均剩余距离仍是初始距离的 98.9%，到达率 0%。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 关键发现 3：动态调整——城市已变，模型还在走旧路

在行动过程中加入封路与移动行人：封路任务各路网遵循率 93.0%–96.4%，但**封路遵守率仅 10.0%–46.7%**，最终到达率 0%–3.3%；移动行人任务 90.6%–98.5% 时间仍走在路网上，**行人碰撞率却高达 76.3%–90.0%**。模型大部分时间按路网走，却不擅长因前方封闭及时改道，也常避不开行人——「看见变化还不够，动作必须跟着变」。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 结论与价值

UrbanGround 的价值在于用**可重复、可归因**的城市沙盒暴露出导航模型的「前后接不上」问题：前一刻作出的判断管不到后面的动作，路线越长、途中变化越多，这种断裂越明显。对具身 Agent 进入真实城市前的评测，它提供一种低成本、可逐步定位「模型从哪一步走偏、为何没绕回来」的手段。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

论文/项目主页/代码/App 均已公开（arXiv 2608.27456），接入自己的 Agent 即可在真实城市结构中测试其导航能力。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 深度分析

### 能力断层：识别、定位与导航是三种不同的能力

UrbanGround 最锋利的一刀是把「空间智能」拆成了三级台阶，然后逐级测量。视觉识别（认出地标）75.0%–93.8%，主动探索（走几步补线索）46.3%–82.5%，方向理解（判断地标在自己哪个方位）只有 23.3%–58.3%。Gemini-3.1-Pro 的 82.5% 对 23.3% 说明这不是个别模型的短板，而是当前多模态模型的普遍结构：**语义层强、几何层弱**。模型学到的「东亚银行」是一个可命名的视觉概念，而不是一个带方位角的空间坐标。这解释了为什么「会看街景」与「会走路」之间存在一道评测长期没有测到的鸿沟——过去用单张街景图测空间智能，恰好只测了最上面那一级台阶。这与 [[concepts/world-models]] 中「世界模型需要内隐几何而非仅语义表征」的判断互相印证：缺乏几何约束的表征，在需要视角变换的任务上会系统性地崩塌。

### 长程失败的本质：不是「不会走」，而是「状态不持续」

轨迹数据推翻了「模型一开始就走错」的直觉解释：长程任务结束时 52.5%–81.3% 的轨迹比起点更接近目标，但完整成功率 0%–3.8%。GPT-5.5 的数据更极端——57.5% 的轨迹在靠近目标，34.6% 缩短了至少 20% 的距离，可平均剩余距离仍是初始的 98.9%，到达率 0%。走近的轨迹和绕远的轨迹互相抵消，整体几乎没有净进展。这是典型的**逐步贪心、全局不一致**模式：每一步的局部判断大体正确，但没有一个持续的状态表示把这些判断串成一条受约束的路线。每次转弯、过街、开关地图，模型都在从头重估「我在哪、目标在哪」，上一步的置信度无法传递到下一步。从 [[concepts/agent-memory-architecture]] 的视角看，这暴露的正是工作记忆中缺乏「已验证路径 + 误差累计」这类持久空间状态的代价——上下文里没有可靠的自定位锚点，重估就会漂移，漂移就会累积。

### 感知—行动闭环的断裂：看见变化 ≠ 改变计划

动态调整实验给出了本基准最有工程价值的一组数字：封路任务中路网遵循率高达 93.0%–96.4%，封路遵守率却只有 10.0%–46.7%；移动行人任务中路网遵循率 90.6%–98.5%，碰撞率却高达 76.3%–90.0%。两组数字并排读，结论很清楚：**「按路网行走」是一个已经固化的低层策略，「因变化而改道」是另一个模型完全不具备的高层能力**。模型能看见封路（否则遵循率不会掉），但看见之后并不会触发计划更新——观测更新了，规划没有。这在控制论的意义上是开环的：感知通路工作正常，反馈通路断在「观测→动作修正」这一段。GPT-5.5 在中央障碍前反复向前直到第 100 步也不重新规划，就是这种断裂的逐帧标本。

### 排除法定位病因：长程失败不能推给感知退化

实验设计中有一个容易被忽略的对照：同一任务在晴天、夜晚、阴天下的短程成功率分别为 75.0%、75.0%、76.3%——视觉条件剧变，短程导航纹丝不动。这个阴性结果的价值在于排除了一个方便的解释：长程失败不是「天黑看不清」造成的感知退化，而是纯粹的记忆与规划问题。换言之，即便把感知做到完美，按当前架构外推，长程曲线大概率依然是零。这为后续研究划定了靶心：改进方向应该在**跨步状态持续性**（记忆机制、显式地图标注、子目标分解），而不是继续堆感知数据。

### 可重复沙盒是具身评测的方法论升级

真实城市里的机器人实验代价高，且无法让两个模型遇到完全相同的路况——这使具身评测长期停留在「跑几次、报个均值」的水平。UrbanGround 把变量控制引入城市尺度：同一条路线可反复重跑，只改天气、封路或人流；每一步画面与动作按同一套城市几何对齐，走偏可逐步归因。这与 [[concepts/agent-evaluation-benchmark-frameworks]] 中「评测需要可复现、可归因」的原则同构，也延续了 [[concepts/embodied-intelligence-frontier]] 从静态识别走向连续行动的范式迁移：[[entities/embodied-native-llm-embodied-intelligence-new-stage]] 代表模型侧的原生具身支持，UrbanGround 补上评测侧基础设施，与 [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model]]、[[entities/amap-abot-earth-0.5-3d-native-world-model]] 形成表征—评测的配对。

## 实践启示

1. **评测具身 Agent 时，用「成功率—路线长度」曲线替代单点成绩。** 短程 75% 而长程 0% 的模型，单看短程指标会被严重高估。跑长程基准时重点看成功率随交互距离的衰减斜率，以及终点与目标的平均剩余距离——后者能区分「走错了」和「走对了大半却卡在半路」。

2. **给导航/具身 Agent 显式的外部空间记忆，不要指望上下文窗口兜底。** UrbanGround 的失败模式表明，每步重判会让小偏差无界累积。可行做法：维护一份「已验证路点 + 已排除岔路 + 当前自定位置信度」的结构化状态，每步先校准再决策，而不是把历史截图原样堆进上下文。

3. **把「观测到变化」和「触发重规划」做成两个显式环节。** 模型能看见封路却不改道，说明反馈不会自动转化为计划更新。应在 Agent 循环里加入偏差检测器（如连续 N 步未缩短与目标的距离）和强制重规划触发器——GPT-5.5 对着中央障碍直走到第 100 步，就是没有这个触发器的后果。

4. **上线前在可重复沙盒里做回归测试，而不是直接上街。** 真实城市实验不可复现、归因困难。先用沙盒重跑同一组路线、逐项切换天气/封路/人流，定位 Agent 最容易走偏的环节（是定位、记忆还是改道），修复后再进真实环境，能大幅降低试错成本。

5. **过程指标与结果指标必须成对报告。** 路网遵循率高不代表会走路（碰撞率 76.3%–90.0%），轨迹接近目标不代表能到达（剩余距离 98.9%）。具身能力评估应同时采集到达率与过程指标（约束遵守率、碰撞率、轨迹改善比例），单一指标会产生误导。

## 相关实体

- [[entities/om-ai-vlx-go-vlm-navigation-0.6b-2026|OM AI VLX-Go VLM 城市导航]]：前者是端上 VLM 导航能力的模型侧实现，UrbanGround 则提供评测这类导航能力的城市级基准。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0]]（跨本体多模态物理世界模型）与 [[entities/amap-abot-earth-0.5-3d-native-world-model|AMAP Abot-Earth 3D 原生世界模型]]：三维城市/物理世界建模侧，UrbanGround 关注这些世界表征在连续导航任务中的实际落地表现。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]
- [[entities/embodied-native-llm-embodied-intelligence-new-stage|具身原生 LLM]]：体现本轮具身智能从「看懂」到「走完一条路」的评测范式演进。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

→ [[raw/articles/urbanground-embodied-navigation-benchmark-2026|原文存档]] ^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]