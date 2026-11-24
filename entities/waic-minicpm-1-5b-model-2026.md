---
title: "MiniCPM-Robot：面壁智能开源具身智能 VLA 模型系列"
created: 2026-07-22
updated: 2026-11-18
type: entity
tags: [model, open-source, robot, vla, embodied-ai, chinese-ai, minicpm]
sources: [raw/articles/waic-minicpm-1-5b-model-2026]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# MiniCPM-Robot：面壁智能开源具身智能 VLA 模型系列

## 深度分析

面壁智能在 2026 年世界人工智能大会（WAIC）上发布了首个具身智能系列模型成果，包含两个核心模型：**MiniCPM-RobotManip**（操作模型）和 **MiniCPM-RobotTrack**（跟踪导航模型），已全部开源。这套模型的技术路线的核心特点是：不走「从单场景到通用」的传统路径，而是从一开始就构建通用具身模型，再落到具体场景。 ^[raw/articles/waic-minicpm-1-5b-model-2026.md]

### MiniCPM-RobotManip：带记忆的操作模型

RobotManip 基于面壁智能 2026 年 5 月发布的 [[entities/minicpm-v-46-13b|MiniCPM-V 4.6]] 多模态大模型训练而成。其核心技术是 **视觉 Token 极致压缩**——将图片转换的 Token 数量大幅降低而不损失关键信息。这一压缩能力使模型能流畅地将历史观测数据纳入推理，让机器人「记住」过去几秒甚至几十秒的画面和动作，而不像大多数 VLA 模型那样只看「当下这一帧」。通过流式计算，带记忆的推理成本被追平了传统单帧反应式模型的水平，参数规模也比同类型模型（如 π0.5）更小。 ^[raw/articles/waic-minicpm-1-5b-model-2026.md]

评测数据显示，MiniCPM-RobotManip 综合性能位列 VLA 模型第一梯队，与 Qwen-VLA、π0.5 等主流模型性能相近甚至更优。在考验上下文记忆的 RMBench 基准上，π0.5 仅得 10 分（接近随机），而 MiniCPM-RobotManip 达到 **53 分**，验证了视觉压缩+记忆融合路线的有效性。模型同时保持了多模态大模型的通用能力，在仿真评测场景上采用通用多任务统一训练，指标达到专家模型水平。 ^[raw/articles/waic-minicpm-1-5b-model-2026.md]

### MiniCPM-RobotTrack：离线跟踪导航模型

RobotTrack 基于面壁 MiniCPM4-0.5B 模型加 MLP 结构训练，参数规模仅 **0.9B**，不需要额外感知模块或外接开发板，仅靠机器狗原装摄像头和本地算力即可运行。在宇树 Go2 机器狗上实现了稳定 **5 帧/秒** 的端到端推理，响应延迟约 180 毫秒，支持单目标跟踪、动态多目标跟踪和模糊指令跟踪，三项成功率分别达到 **89.8、73.4 和 80.4**，为开源方案最优。 ^[raw/articles/waic-minicpm-1-5b-model-2026.md]

纯本地部署带来的优势包括：无网络延迟的即时响应、不上传摄像头画面的隐私保护、以及弱网/无网环境下的正常运作（如电梯场景中信号缺失仍能完成人员跟随）。团队通过一套「自己找茬、自己补课」的数据管线，先在大规模通用场景数据上训练，再通过仿真和真实机器人持续试跑暴露短板，针对性收集薄弱场景数据反复迭代。 ^[raw/articles/waic-minicpm-1-5b-model-2026.md]

### 配套生态与合作落地

与两个模型同步亮相的还有 **PhyAI** 推理框架，由明体科技联合北京邮电大学、北京大学研发，专为具身智能端侧推理和云端大规模 RL 训练设计。在 RTX 5090 上运行 π0、π0.5、GR00T 等模型分别带来 2.9×、1.8× 和 2.3× 推理加速。面壁本次发布的机器人模型在发布当天即完成适配，推理帧率从 10Hz 提升至 33Hz（H20 上可达 37Hz）。 ^[raw/articles/waic-minicpm-1-5b-model-2026.md]

在产业化落地方面，面壁与乐聚机器人合作推出展厅导览和园区巡检方案，机器人在无网络环境下纯本地完成讲解和异常识别。与吉利汽车合作训练的 VLA 模型通过自研 **RoboHarness** 框架（基于 4 月发布的 EmbodiedClaw 框架），将 VLM 多模态大模型、VLA 模型、机器人本体和导航系统整合，落地到吉利生产仓储场景，支持长链条任务规划与执行，并配有可治理的数据闭环实现模型持续进化。 ^[raw/articles/waic-minicpm-1-5b-model-2026.md]

## 实践启示

对想在端侧落地具身能力的团队，这条「面到点」路线给出了四个可复制的决策点：先解决记忆与算力的矛盾，再做部署减法，用数据闭环补长尾，最后靠框架编排把模型推进真实产线。

### 1. 用视觉压缩换记忆，不要用算力硬扛

若你的 VLA 只看「当前一帧」，优先评估视觉 Token 压缩加流式计算这条路径，而不是直接堆卡：RobotManip 证明带记忆的推理成本可以追平单帧反应式模型。记忆是硬差距而非锦上添花——RMBench 上 π0.5 仅 10 分（接近随机），RobotManip 拿到 53 分。评估长程记忆能力时，可参考 [[entities/embodied-memory-bench-emem-zju-2026|EMem-Bench]] 的四类记忆任务划分。^[raw/articles/waic-minicpm-1-5b-model-2026.md]

### 2. 端侧部署先做减法：砍外接模块、砍云端依赖

RobotTrack 用 0.9B 参数、原装摄像头和本地算力做到 5 帧/秒、约 180 毫秒端到端延迟，不外加感知模块或开发板，换来即时响应、隐私保护和弱网可用（电梯场景照常跟随）。选型时先问哪些环节可以被「小模型+系统级链路优化」替代，再谈功能上限。^[raw/articles/waic-minicpm-1-5b-model-2026.md]

### 3. 长尾场景靠「自己找茬、自己补课」的数据闭环

先大规模通用数据打底，再用仿真和真机试跑主动暴露短板（目标横穿、快速转向、遮挡、多人交叉），定向补采薄弱场景数据、逐轮迭代。这比一次性堆数据更可控，也更贴合真实场景的长尾分布——具身数据供给的产业侧格局可参考 [[entities/embodied-ai-data-market-landscape-97-players-44-billion-2026|具身数据产业格局]]。^[raw/articles/waic-minicpm-1-5b-model-2026.md]

### 4. 模型之外，推理框架与任务编排决定落地速度

发布当天完成 PhyAI 适配、帧率 10Hz 提到 33Hz，说明推理生态适配可以是「发布日动作」而非慢变量，端侧优化方法论可参考 [[concepts/inference-optimization|推理系统优化]]。RoboHarness 把 VLM、VLA、本体和导航当工具模块编排，配失败重试与数据闭环，让机器人在干活中持续进化——产线落地的门槛在系统能力，不在单模型跑分，闭环编排思路可对比 [[entities/embodiedskills-closed-loop-agentloop-vla-2026|EmbodiedSkills 闭环 AgentLoop]]。^[raw/articles/waic-minicpm-1-5b-model-2026.md]

→ [[raw/articles/waic-minicpm-1-5b-model-2026|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

