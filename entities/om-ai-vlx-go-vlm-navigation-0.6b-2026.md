---
title: "Om AI VLX-Go: 0.6B 导航 VLM — VLX 系列收官"
created: 2026-07-06
updated: 2026-09-07
type: entity
tags: [vlm, multimodal, vision, navigation, om-ai, model-architecture, embodied-ai]
sources: [raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026, raw/articles/om-ai-vlx-go-paperweekly-2026]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Om AI VLX-Go: 0.6B 导航 VLM — VLX 系列收官

VLX-Go 是 Om AI（联汇）VLX 端侧流式多模态模型系列的第三弹，定位为**行动决策层**，解决 VLM 从「看懂」到「行动」的跨越——即模型不仅能描述环境、定位目标，还能输出可执行的导航指令。^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

## 核心定位：从感知到行动

前两代 VLX（Flow 流式视频理解、Seek 细粒度感知）解决了 VLM「看得懂」的问题，VLX-Go 补上了「行动决策」层：接收连续视觉流输入，输出机器人下一时刻的运动目标——往哪里走、何时修正方向、如何在动态画面中避开障碍物。^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

## 技术特点

- **0.6B 参数在端侧跑通导航**：极小模型即可驱动物理世界导航任务，不依赖云端推理
- **从目标定位到行动**：VLX-Go 补上 VLX 系列的行动决策层，与 Flow（流式理解）、Seek（精准定位）形成「理解→定位→行动」完整链路^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]
- **避开突发障碍物**：依靠实时视觉推理进行动态避障，非预编程路径规划
- **短时航点预测机制**：模型每次只预测未来一小段轨迹，执行后再根据新画面更新下一段，避免长路径累积误差^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

## VLX 系列对比

| 型号 | 定位 | 能力 |
|------|------|------|
| VLX-Flow | 流式视频理解 | 边看边理解连续视频流 |
| VLX-Seek | 细粒度感知 | 区域检索 + 精确定位 |
| **VLX-Go** | **行动决策** | **端侧导航 + 动态避障** |

## 与现有实体关联

- [[entities/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026|VLX-Flow]] — 系列开篇：流式视频理解 VLM
- [[entities/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026|VLX-Seek]] — 系列第二弹：3B 细粒度感知 VLM
- [[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026|原文存档]]

## 第 2 来源 — PaperWeekly 解读

PaperWeekly 对 VLX-Go 进行了补充报道，重点介绍了 VLX-Go 在真实机器人上的局部导航演示：跟随目标行走、动态避障、地面机器人平台上的短时航点预测流程。^[raw/articles/om-ai-vlx-go-paperweekly-2026.md]

报道补充了 VLX-Go 的训练模式细节：基于**离线轨迹数据学习**，系统先缓存视频帧的视觉特征减少训练开销，规划器读取视觉 token、历史帧信息后预测短时航点序列。^[raw/articles/om-ai-vlx-go-paperweekly-2026.md]

## 深度分析

### VLX-Go 的技术架构设计范式

VLX-Go 的核心设计选择是**将 VLM 的输出形式从文本答案推进到短时行动轨迹**。传统 VLM 输出停留在文字层面（"向左前方移动"），但真实机器人需要的是可执行的航点坐标——这要求模型的输出层与底层控制接口对齐。^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

VLX-Go 明确自身定位为**感知和控制之间的局部策略层**，而非替代底层控制器。模型负责将视觉、语言和历史帧接入行动预测，输出短时航点；速度命令、平台动力学和安全约束仍交给下游控制器与安全层。这种职责拆分使得模型体积可以压缩到 0.6B，同时保持与不同硬件平台的兼容性。^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

### 离线轨迹学习 + 在线强化学习的混合训练策略

VLX-Go 采用两阶段训练：第一阶段基于离线轨迹数据学习，系统预先缓存视频帧的视觉特征以减少训练开销，规划器学习预测短时航点序列。第二阶段引入在线强化学习优化，在仿真器或闭环环境中，模型预测航点后由控制器执行，环境返回新视觉观测和反馈信号（碰撞、障碍物距离、目标保持、进度奖励等），帮助模型学到更安全、更平滑的局部策略。^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

这种混合策略的关键创新在于**仿真器贯穿数据生成、在线优化和闭环评测三个环节**，使得离线数据难以覆盖的动态避障场景（障碍物布局变化、目标遮挡、执行误差累积）可以通过交互式反馈得到有效治理。^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

### EVT-Bench STT 评测表现

在 EVT-Bench 的 STT 任务上，VLX-Go 0.6B 规划器取得 **SR（成功率）85.42%**、**TR（跟踪率）94.08%**、**CR（碰撞率）6.55%**。评测结果说明：在目标跟随任务中，0.6B 级别的模型已经具备实用竞争力；碰撞率的优化仍可通过仿真环境、奖励设计和安全约束继续收窄。^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

### 与行业同类方案的定位差异

与主流端到端导航方案不同，VLX-Go 不接管整套机器人控制栈，而是专注于将视觉语言状态转换为面向控制链路的航点输出。模型越轻，推理成本和部署压力越低，越接近真实运行约束。对端侧具身设备来说，导航决策随画面变化不断刷新，0.6B 级规划器在部署成本、调用开销和端到端延迟方面具有显著优势。^[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026.md]

## 实践启示

1. **职责拆分是模型瘦身的关键**：VLX-Go 不替代底层控制器，而是作为视觉驱动的局部策略模型。这种"模型负责视觉语言规划 + 控制器负责执行 + 安全层提供约束"的职责拆分，是将其压缩至 0.6B 仍能实用的前提。在构建端侧 AI 系统时，应清晰界定模型的能力边界。

2. **混合训练策略解决数据覆盖不足**：纯离线轨迹学习难以覆盖所有障碍物布局和动态干扰，VLX-Go 的离线+在线强化学习混合策略证明了：仿真环境中的闭环反馈对动态避障类任务至关重要。仿真器应贯穿数据生成、策略优化和闭环评测全流程。

3. **短时航点预测减少累积误差**：每次只预测一小段轨迹，执行后再根据新画面更新，这种"预测-执行-观测"的循环机制避免了长路径累积误差，也使得在真实设备上的安全检查更容易落地。这对长时延场景下的机器人部署具有普适参考价值。

4. **VLX 三件套的协同效应**：Flow（持续感知）→ Seek（精准定位）→ Go（行动决策），三层能力形成完整闭环。在构建端侧系统时，应优先考虑能力链路的完整性而非单一模块的性能巅峰。

5. **0.6B 参数量的实际意义**：在具身系统里航点预测会随着新画面不断刷新，模型越小，部署成本和端到端延迟越可控。VLX-Go 证明 VLM 可以同时做到"小参数"和"实用性能"——前提是明确的任务边界和恰当的训练策略。

→ [[raw/articles/om-ai-vlx-go-vlm-navigation-0.6b-2026|原文存档]]
→ [[raw/articles/om-ai-vlx-go-paperweekly-2026|PaperWeekly 报道原文]]
