---
title: "MiniCPM-Robot：面壁智能开源具身智能 VLA 模型系列"
created: 2026-07-22
updated: 2026-07-27
type: entity
tags: [model, open-source, robot, vla, embodied-ai, chinese-ai, minicpm]
sources: [raw/articles/waic-minicpm-1-5b-model-2026]
confidence: 0.75
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

→ [[raw/articles/waic-minicpm-1-5b-model-2026|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

