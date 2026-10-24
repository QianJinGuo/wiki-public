---
title: "阿里巴巴 RoboFlywheel 首发：百万级仿真资产重构机器人训练场"
created: 2026-09-30
updated: 2026-10-25
type: entity
tags: [embodied-ai, simulation, world-model, training-data, alibaba]
sources: [raw/articles/roboflywheel-embodied-simulation-assets-2026, raw/articles/roboflywheel-data-loop-embodied-bench-alibaba-2026-09]
confidence: 0.7
---

# 阿里巴巴 RoboFlywheel 首发：百万级仿真资产重构机器人训练场

> **Background**：本文基于机器之心对阿里巴巴 RoboFlywheel 具身智能仿真资产体系的报道整理。核心信息：RoboFlywheel-World 三项研究（RoboFlywheel-Rigid 刚体 / RoboFlywheel-Articulation 铰链物体 / RoboFlywheel-Soft 柔性体），构建从物体资产到任务场景的完整仿真训练数据产线。→ [[raw/articles/roboflywheel-embodied-simulation-assets-2026|原文存档]]

## 深度分析

RoboFlywheel 的核心主张是"先把物体做成可交互的资产，再把资产组织成任务需要的世界"。三项研究覆盖具身仿真的三大物体类别：刚体解决"能不能抓"，铰链物体解决"能不能动"，柔性体解决"既要拿起来也要知道有没有损伤"。每个资产都补齐尺度、质量、惯量、摩擦、关节和材料描述，并通过几何检查、关节运动、稳定性探测和具体操作做交互验证，只把能执行、可追溯的结果交给下游 ^[raw/articles/roboflywheel-embodied-simulation-assets-2026.md]。

**刚体（RoboFlywheel-Rigid）**：处理粒度下沉到部件——恢复语义、分解结构、构建碰撞几何，再补齐物理参数，通过多凸体分解保留可交互结构。更重要的是任务驱动的 Agentic 场景构建：根据具身任务描述检索对象、按机器人可达性规划放置与支撑关系、安排房间布局，经物理和视觉检查校验循环自动生成跨仿真器可用的训练场。物体子集上用六种不同手型做抓取与提起评估 ^[raw/articles/roboflywheel-embodied-simulation-assets-2026.md]。

**铰链物体（RoboFlywheel-Articulation）**：关键创新是"类别级生成器"——以柜子为例，尺寸、门型、抽屉数量可变，板材连接、部件位置与关节装配约束写进程序；智能体反复测试、修复生成器，验证后批量采样，无需为每个新资产重新推理。可验证数据：测试资产运动学通过率 99.93%；200 个资产在 Genesis、PyBullet、MuJoCo 三引擎固定根部被动运行 10 秒联合通过率 99.0%。程序保留的规则与参数支持定向 3D 编辑（只改指定部分，其余不变）——规模化靠可复用的规则而非复制同一个模型 ^[raw/articles/roboflywheel-embodied-simulation-assets-2026.md]。

**柔性体（RoboFlywheel-Soft）**：面向 485 种标准对象类型，将柔性实体、布片、薄壳、服装和袋体分成五类几何家族匹配不同仿真表示。最难补的是材料行为：先用视觉和语义先验提出参数，再由智能体选择仿真探测、诊断失败并有界调整，最后交给未参与调整的独立探测验收 ^[raw/articles/roboflywheel-embodied-simulation-assets-2026.md]。

**与训练数据产线的关联**：RoboFlywheel 补的是具身智能"从真机到世界模型"之间缺失的仿真数据供给层，与 [[concepts/world-models|世界模型]]、[[concepts/embodied-intelligence-frontier|具身智能前沿]] 的模型侧进展互补——数据侧（资产可交互性验证、跨仿真器通过率）是其区别于纯模型论文的工程特色。类别级生成器思路与 [[concepts/agent-harness-engineering-paradigm|Harness Engineering]] 的"规则先行、验证闭环"方法论同构。

## 第二来源：阿里数据侧的数据闭环与 Bench 评测（2026-09-28）

阿里数据官方号从「数据配方」角度补足了同一项目的另一半：具身智能的瓶颈不是采集量而是有效训练量——采什么、留什么、怎么组合、如何训练与验证决定能力上限。项目在云栖大会上由阿里巴巴联合清华大学、上海交通大学发布，定位为具身智能开放数据基础设施，四项基础能力围绕数据生产、处理、验证与复用展开 ^[raw/articles/roboflywheel-data-loop-embodied-bench-alibaba-2026-09.md]。

**RoboFlywheel-Sim 的资产规模首次给出具体量级**：首批覆盖 52 万刚体资产、40 万铰链资产与 10 万柔性资产，构建生成式资产底座。围绕统一资产格式、跨引擎资产编译、自然语言场景构建与声明式评测 Harness，同一套资产、场景和任务可在不同引擎中复现，支撑规模化压力测试、失败诊断与版本回归——仿真的价值被明确定义为「让更多问题在进入物理世界之前被更早发现」，而不替代真机检验 ^[raw/articles/roboflywheel-data-loop-embodied-bench-alibaba-2026-09.md]。

**RoboFlywheel-Bench 把评测对象从模型扩展到具身系统**：首发的 WholeBodyBench 聚焦模型能力，从基础单元、单元组合到高层组合逐级提升任务复杂度，同时考察组合泛化、场景泛化以及成功与安全两维指标；后续评测还将覆盖真机与仿真的数据采集效率、机器人构型能力边界等模型之外决定落地成败的环节 ^[raw/articles/roboflywheel-data-loop-embodied-bench-alibaba-2026-09.md]。

两个来源互补关系清晰：首篇（机器之心转载阿里技术）回答「资产怎么造得可交互」，本篇回答「资产与场景如何被组织成可比较、可复现的数据配方与评测」。后者把 [[concepts/agent-evaluation-benchmark-frameworks|评测框架]] 的「分级难度 + 成功/安全双指标」思路搬进了具身数据基础设施，与 [[concepts/embodied-intelligence-frontier|具身智能前沿]] 中仿真与真机之间的验证鸿沟直接对应 ^[raw/articles/roboflywheel-data-loop-embodied-bench-alibaba-2026-09.md]。

## 相关

- [[concepts/world-models|世界模型]]
- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0 跨本体世界模型]]

→ [[raw/articles/roboflywheel-embodied-simulation-assets-2026|原文存档]]
