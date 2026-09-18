---
title: "LingBot-Video：全球首个具身专属MoE视频模型"
created: 2026-07-09
updated: 2026-09-19
type: entity
tags: [ai, video-generation, embodied-ai, moe, open-source, model, ant-group, lingbot]
sources: [raw/articles/lingbot-video-moe-embodied-video-2026, raw/articles/刚刚全球首个具身专属的moe视频模型开源了]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LingBot-Video：全球首个具身专属MoE视频模型

> **LingBot-Video** 是由蚂蚁灵波（Ant Group）发布的面向具身智能的 MoE（Mixture of Experts）视频基础模型与视频物理引擎。该模型采用 MoE30B-A3B 架构，总参数量 30B、推理时仅激活 3B，通过全链路设计（架构—数据—训练）专为机器人、人形智能体等具身场景打造其核心关注点从通用视频的"时长、美学、画质"转向"是否符合物理规律"^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

## 架构设计

模型采用稀疏 MoE 架构，在固定计算预算下扩大参数容量。MoE30B-A3B 在 1M Token 长度下对比 Dense6B、Dense 14B、Dense 30B 的速度比分别达到 1.50×、2.59× 和 3.18×，同时保持接近 3B 模型的推理效率。稀疏 MoE 将总参数规模与每个 Token 实际激活的计算量解耦，使视频模型能处理复杂运动轨迹、三维空间一致性、材质纹理等复杂分布。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

## 数据策略

模型引入超过 70,000 小时的 embodiment-oriented footage，覆盖机器人操作 VLA、导航、第一视角视频，包括真实机器人、仿真、开源、第三人称视角以及人形机器人、四足机器人等平台。训练流程采用专门的"少筛选、多保留"策略，防止高价值具身数据被海量普通互联网视频稀释。所有素材经过**五维结构化标注**（物体、材质、动作时间戳、受力交互关系），并采用**课程式五阶段渐进训练**（从低清静态图像到高清长时序视频）。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

## 强化学习与物理约束

与通用视频模型仅用画面美观度、文本匹配度做优化目标不同，LingBot-Video 搭建了一套**分层强化学习奖励体系**，从三个维度约束生成结果：^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

1. **感知维度**：画面清晰度、文字描述匹配度、动态流畅度
2. **物理维度**：物体不穿透、无凭空消失、运动符合重力惯性、材质受力形变合理
3. **执行维度**：机器人肢体结构完整、动作流程可落地、任务目标完整完成

训练采用 **GRPO（Group Relative Policy Optimization）** 方案，搭配负感知微调规避奖励黑客问题。模型原生支持 **Action-to-Video** 动作条件生成——输入机器人动作指令即可输出后续完整视觉变化，可直接对接机器人运动规划模块。此外配备**级联精炼方案**：先生成 480p 基础时序画面保证运动逻辑，再精炼至 1080p 高清画质，平衡推理速度与画面细节。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

## 评测表现

在评测中与 NVIDIA Cosmos3、LongCat-Video、LTX-2.3 等开源模型比较：^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

- **TI2V（Text-Image-to-Video）任务**：在开源竞品中达到 SOTA 水平，general quality 和 embodied domain 两项得分均位居第一
- **T2V（Text-to-Video）任务**：general quality 排名第二，embodied domain 得分超过 Cosmos 等竞争基线
- 已在 **RBench** 上超越业内通用视频生成标杆模型

## 价值分层应用

该模型的价值可以划分为三层：^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

1. **Data Engine** — 为机器人训练提供低成本、可反复试错的物理世界模拟数据
2. **Policy Evaluator** — 在虚拟视觉环境中提前评估策略效果，降低真实测试风险
3. **Action Planner** — 直接对接机器人运动规划模块，输出动作条件对应的视觉变化

→ [[raw/articles/lingbot-video-moe-embodied-video-2026|原文存档]]

## 第 2 来源 — 量子位（2026-07-09）

量子位对 LingBot-Video MoE 的独立报道，重点覆盖了模型的具身专属设计理念和开源意义。^[raw/articles/刚刚全球首个具身专属的moe视频模型开源了.md]

→ [[raw/articles/刚刚全球首个具身专属的moe视频模型开源了|量子位报道原文]]

## 深度分析

### 具身视频与内容视频是「两套评价体系」

「具身专属视频模型」是否成立，起点不在模型能力，而在评价标准的分裂。量子位的论断是：内容视频和具身视频是两套评价体系。通用模型围绕视觉质量、语义对齐、运动连贯训练，观众被打动的是画质与构图；机器人看世界却不同——它不只要看见杯子，还要判断自己伸手后杯子会怎么动、走过去会不会撞到障碍。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

代价也不对称：通用模型偶发的穿模、物体凭空消失、动作违背惯性，对短视频只是瑕疵；同样的视频拿去训练机器人，就等于教它一套错误的世界规律。所以「具身专属」不是营销定位，而是评价函数从「像不像人拍的视频」换成「符不符合物理规律」之后必然推出的产品形态（对照 [[concepts/world-models|世界模型]]）。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

### MoE30B-A3B：把总容量与单次计算量解耦

30B 总参数、单次生成约激活 3B：直接收益是成本下降，更关键的是扩展方式变了。Dense 模型像一个大办公室，每个任务所有人都要一起上场，稳但贵；MoE 像大型专家库，任务来了只叫最相关的一组专家出手，总参数规模与每个 Token 实际激活的计算量由此解耦。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

MoE30B-A3B 在 1M Token 下对比 Dense6B、Dense 14B、Dense 30B 的速度比为 1.50×、2.59×、3.18×，同时保持接近 3B 模型的推理效率；视频要模拟连续物理世界，需处理复杂运动轨迹、三维一致性与材质纹理等复杂分布，固定计算预算下只能靠稀疏结构扩大参数容量（参见 [[concepts/moe-mixture-of-experts-2025|MoE 混合专家]]）。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

效率在这里是可用性前提：机器人训练、策略评估与动作规划天然需要大量试错，全参数激活会让这个「视频物理引擎」根本用不起来。

### 数据侧走的是「第三条路」

语言模型能起来靠的是互联网天然积累的海量文本，但机器人没有属于自己的互联网；真机数据要靠遥操作与真实场地一点点采集，慢且贵，仿真数据又常撞上 sim-to-real gap。LingBot-Video 选的是第三条路：把通用互联网视频和具身数据结合起来。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

具体是超过 70000 小时的 embodiment-oriented footage，覆盖机器人操作 VLA、导航与第一视角视频（含真实机器人、仿真与第三人称视角）；这些数据不是简单拼接，而是在训练流程的专门阶段对稀缺高价值具身数据刻意「少筛选、多保留」，防止被普通互联网视频稀释。素材还会经过五维结构化标注（物体、材质、动作时间戳、受力交互关系），配以课程式五阶段渐进训练，并对机械操作、精密抓取等长尾场景做分布感知加权——真正的差别是稀释控制与注入时机，而非数据总量。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

### 物理约束如何进入优化目标

传统视频模型只用画面美观度与文本匹配度做优化目标，几乎不约束物理逻辑。LingBot-Video 的分层强化学习奖励体系从三维同步约束生成结果：感知维度保障清晰度与动态流畅；物理维度是核心优化指标，校验物体不穿透、无凭空消失、运动符合重力惯性、形变合理；执行维度校验肢体结构完整、动作流程可落地、任务目标完整完成。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

训练采用 GRPO 搭配负感知微调以规避奖励黑客（对照 [[concepts/grpo-policy-optimization-2026|GRPO]]）；模型原生支持 Action-to-Video 动作条件生成，输入动作指令即输出后续完整视觉变化，可直接对接运动规划模块；另配套级联精炼，先生成 480p 基础时序保证运动逻辑，再精炼至 1080p。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

三维里「执行维度」最容易被忽略，也最关键：它把模型从「物理上说得通」推到「机器人做得出来」，而 Action-to-Video 让输出从观看对象变成了策略接口（同一方向上，[[entities/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026|LingBot-VA 2.0]] 走的是具身原生预训练路线）。

### 榜单成绩证明了什么，没证明什么

评测上，LingBot-Video 被拿来与 NVIDIA Cosmos3、LongCat-Video、LTX-2.3 等开源模型比较：TI2V 达开源 SOTA，general quality 与 embodied domain 双项第一；T2V general quality 第二，但 embodied domain 超过 Cosmos 等基线；并已在 RBench 上超越业内通用视频生成标杆。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

这些分数要打个折：榜单衡量的是「生成片段在指标上更像真实的机器人视频」，而「像」与「可用」之间还隔着下游闭环的验证。报道自列的未决问题包括长时序一致性、柔性物体与液体等复杂物理交互、视频预测向真实机器人闭环的转化，以及具身视频评测标准本身的建设（对照 [[entities/world-model-evaluation-position-paper-nju-2026|世界模型评测立场]]）。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

价值分层同样要限定条件。技术报告把 LingBot-Video 定位为面向机器人社区的三层能力：Data Engine（低成本、可反复试错的物理模拟数据）、Policy Evaluator（在虚拟视觉环境中提前评估策略）、Action Planner（直接对接运动规划，输出动作条件对应的视觉变化）；这条排序是由远及近的阶梯，越往下对物理保真度与因果准确性的要求越高，也越依赖尚未解决的评测与闭环问题。^[raw/articles/lingbot-video-moe-embodied-video-2026.md]

## 实践启示

1. **用物理正确性而非画质做选型主指标**。先看模型是否把不穿透、无凭空消失、符合重力惯性、形变合理写进优化目标与评测维度；只有清晰度分数而无物理约束设计的模型，进到策略训练里可能提供负价值。

2. **看激活参数量，而不是总参数量**。机器人场景需要大量模拟试错，决定可跑性的是一次前向激活多少参数；MoE30B-A3B 在 1M Token 下对 Dense 30B 有 3.18× 速度优势。

3. **数据配比与注入时机比数据总量更重要**。稀缺具身数据需要被刻意保护（少筛选、多保留）并按课程式五阶段渐进注入，长尾场景额外做分布感知加权，否则会被海量互联网视频稀释。

4. **用「感知 / 物理 / 执行」三维模板设计奖励**。感知保障可读性、物理保障规律自洽、执行保障下游可行；执行维度最容易被漏掉，却最决定模型能否从「看着对」走到「做得成」。

5. **把生成视频当候选数据或环境，而非成品资产**。三层价值是逐级加码的赌注，落地从最轻一层开始：先用生成数据扩充训练集，再用真机或仿真验证策略收益，最后才谈接进评测与规划。

6. **把 Action-to-Video 当作进入策略栈的硬门槛**。只能接文本或图像条件的模型上限就是数据引擎；只有原生支持动作条件生成并对接运动规划的模型，才有资格充当评测器或规划器。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

