---
title: "CausalWM：把因果思维链焊进具身世界模型（Aether AI 16B）"
created: 2026-09-20
updated: 2026-09-20
type: entity
tags: [world-model, embodied-ai, causal-reasoning, chain-of-thought, video-generation, reinforcement-learning, aether-ai]
sources: [raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026]
confidence: 0.7
review_recommendation: ingest
---

# CausalWM：把因果思维链焊进具身世界模型（Aether AI 16B）

Aether AI 发布第一版因果世界模型 CausalWM（16B），核心主张是：世界模型不应只回答"未来长什么样"，还应显式写出"未来是怎样一步步发生的"。它把光流、深度、三维几何等物理变量组织成一条有序的 **Causal Chain-of-Thought（因果思维链）**，让模型沿 `Observation → Physical Motion → Geometry → Future Observation` 的链条逐步推演，再生成未来画面。CausalWM 在具身世界模型基准 TriWorldBench 上以 TWB-Score 66.04 登顶，并在 PAI-Bench 位列第一。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

## 问题：隐式物理建模不可验证

主流世界模型路径是"当前画面 + 动作指令 → 直接生成未来视频"。数据规模足够大时这条路径很有效，但运动、接触与空间变化全部被压进模型内部隐表示，外界无法判断模型究竟理解了物理过程，还是从训练数据里找到了 shortcut correlation 直接"猜"出结果。场景一复杂、预测时间拉长，中间一次接触判断失误就会让后续画面越偏越远，误差在看不见的地方层层累积。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

## 机制：Causal Chain-of-Thought

CausalWM 的做法类似"让只写答案的学生补上解题过程"：在生成未来视频前，先把一部分物理变化显式预测出来，并把每一步结果重新放回上下文，作为下一步推理的条件。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

| 推理阶段 | 物理含义 | 使用的变量 |
|---|---|---|
| Physical Motion | 画面中什么在动、往哪动 | Optical Flow |
| Geometry | 运动如何改变三维空间关系（物体位置、臂-物相对距离） | Depth、Pointmap |
| Future Observation | 在此运动与几何变化之后，未来画面应是什么样 | RGB 未来帧 |

- 训练时的典型顺序为 `Flow → Pointmap → Future RGB`。
- 关键区别在于**中间变量不只是辅助监督目标**：传统方法即使预测 Flow/Depth/Pointmap，未来视频仍可走另一条内部路径生成；CausalWM 则强制每一步的输出进入下一步的上下文，让这条链真正参与未来生成。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

**stage-ordered attention mask（阶段有序注意力掩码）** 是防止"偷看答案"的关键设计：给不同推理阶段规定阅读权限，前面的步骤只能看到已经发生的信息，不能提前读取后面的 Pointmap 或未来画面。否则模型表面完成了 Motion、Geometry 中间预测，实际上可能已从"最终答案"反推出前面的变量，因果链失去意义。训练与推理遵循同一方向 `Motion → Geometry → Future`。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

## 数据与三阶段训练

Aether AI 为 CausalWM 构建了约 **3 万小时具身视频混合数据**，覆盖机器人操作、第一视角视频、多视角机器人数据及其他物理交互场景。训练分三步：`Pixel-level Pre-training → Causal CoT Mid-training → Multi-objective RL Post-training`。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

**第一阶段（像素级预训练）**：以 LTX-2.3-22B 为基座，在大规模视频上学习语言条件下的未来预测，先建立足够强的世界生成能力。现实数据缺少机器人动作标注，且不同机器人的关节数量、控制方式、动作空间差异很大；团队引入 **CD-LAM**，从视频中提取统一的 latent action，把画面变化压缩成潜在动作表示，使缺少真实控制指令的视频也能学习"什么动作带来什么变化"，并让来自不同机器人的视频共同用于 action-conditioned dynamics。此阶段还训练了多视角版本，为 TriWorldBench 的三视角评测打基础。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

**第二阶段（Causal CoT 中期训练）**：选取 Optical Flow、Depth、Pointmap 作为中间物理变量——它们均可从原始视频自动提取，无需逐条人工标注，易于扩展到大规模数据。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

**第三阶段（多目标 RL 后训练）**：视频生成具有强开放性，同一初始场景可能有多个合理未来，纯监督学习难以同时照顾物理一致性、时序质量与任务完成情况。因此对同一上下文采样多个未来结果，从物理一致性、视觉质量、任务完成度等维度给奖励，用 group-based optimization 比较和优化候选。优化范围还包括中间的 Causal CoT：能产生合理未来的推理路径被强化，容易把运动与几何关系带偏的路径被抑制。至此 Causal CoT 从固定监督链变成可探索、可优化的物理推理过程。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

## Causal CoT 作为 In-context 控制接口（干预能力）

训练中模型持续接收光流、深度、Pointmap 等视觉变量并据此生成下一阶段结果，久而久之学会一种更通用的操作：**只要新条件能表示成视觉信息，就有机会被放进上下文参与未来生成**。这为控制留出入口：把仿真器规划好的机械臂关节轨迹，利用 URDF 结构信息与正向运动学渲染成画面中的动作轨迹、编码为视觉 token 送入模型，经少量微调后模型即可生成与指定轨迹一致的未来视频；人工绘制的物体路径、几何约束也可沿同一接口输入。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

这把问题从"按已有画面接下来大概率出现什么"，推进到"如果让机械臂这样运动，杯子会怎样移动"这类带干预意味的问题。文章同时指出，这距离严格意义的反事实推理仍有距离——按 Judea Pearl 的因果之梯（关联 / 干预 / 反事实）划分，主流模型大多停留在第一层"关联"。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

## 评测结果

| 基准 | 侧重 | 结果 |
|---|---|---|
| TriWorldBench | 三视角（头部 / 左腕 / 右腕）一致性、任务对齐、运动质量、透视合理性、图像质量 | TWB-Score **66.04，排名第一**；并未包揽所有单项第一，总分领先说明同时处理多视角一致性、任务理解、物理交互与生成质量 |
| PAI-Bench | 生成未来是否符合物理规律（案例来自行车记录仪、工业相机等真实采集，横跨自动驾驶/机器人/工业/人类活动/物理常识）；引入 Domain Score 让多模态大模型当裁判逐条追问物理细节 | 最新排名**位列第一** |

单一榜单的榜首可能有偶然因素，但同一套方法在语言条件、动作条件、单视角与多视角等多种设置下都取得较好结果，提供了一个方法层面的信号：相比让模型从 Observation 直接跳到 Future，把中间的 Motion 与 Geometry 显式建模出来，可能确实有助于更稳定地预测物理世界的变化。^[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026.md]

## 可迁移到 Agent / Harness 设计的几点

- **显式中间状态 vs 隐式隐表示**：把决策链的中间变量显式化并强制其进入下一步上下文，与 Agent 系统中"先产出可检查的中间产物（计划 / 草稿 / 证据链）再生产最终结果"同构；可验证性来自中间表示可读，而非最终输出好看。
- **stage-ordered attention mask 对信息泄漏的防御**：训练期给每个推理阶段限定"阅读权限"，禁止读取未来阶段的信息——这与 Agent harness 中"防作弊 / 防答案泄漏"的评测设计（禁止 agent 读取最终答案或后续步骤上下文）是同一类约束。
- **多目标 RL 优化整条推理链**：奖励不仅给最终结果，也用于强化/抑制中间的推理路径，对应"过程奖励 / 轨迹级优化"而非仅结果奖励。
- **中间表示作为控制旋钮（干预接口）**：把中间变量从"解释用"变为"可注入条件"，用 in-context 方式注入 intervention，是 world model / agent 从"预测"迈向"可控"的具体工程入口（URDF + 正运动学 → 视觉 token 的渲染管线是这篇文章给出的一个可复用 trick）。
- **latent action 统一异构动作空间（CD-LAM）**：用从视频提取的统一潜在动作表示替代缺失/异构的控制标注，解决了"多机器人数据无法合训"的数据工程问题。

## 相关

- 世界模型概念框架：[[concepts/world-models|世界模型]]
- 具身智能前沿：[[concepts/embodied-intelligence-frontier|具身智能前沿]]
- 同类世界模型实体：[[entities/motus-2-self-evolving-world-model-dexterous-manipulation-2026|Motus 2 自进化世界模型]]、[[entities/jepa-anything-cross-domain-latent-world-model-phai-2026|JEPA-Anything]]、[[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0]]、[[entities/baai-orca-next-state-prediction-world-model|ORCA 下一状态预测]]、[[entities/feifei-li-tract-visual-tracks-robot-world-model|TRAct 视觉轨迹]]、[[entities/genesis-ai-gene-25-embodied-foundation-model|Gene 2.5 具身基座]]
- 同团队前作 RSIAgent（数字环境中的 RSI 自探索，同属 Aether AI "因果智能"路线）在本篇被作为数字侧对照提及

## 来源与资源

- 论文：CausalWM: Causal Chain-of-Thought Reasoning for Embodied World Model（OpenReview `3pf4d0EEqm`）
- 代码：`https://github.com/AetherLabsAI/CausalWM` ｜ 权重：`https://huggingface.co/AetherLabs-AI/CausalWM` ｜ 项目页：`https://aetherlabsai.github.io/CausalWM/`
- 原文存档：[[raw/articles/causalwm-causal-chain-of-thought-embodied-world-model-2026|原文存档]]
