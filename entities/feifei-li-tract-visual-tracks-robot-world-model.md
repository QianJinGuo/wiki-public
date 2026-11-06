---
title: "TrAct：视觉轨迹作为机器人控制与世界模型之间的共同语言（李飞飞团队）"
created: 2026-08-28
updated: 2026-11-07
type: entity
tags: [world-model, robotics, embodied-ai, visual-tracks, fei-fei-li, stanford, video-model]
sources: [raw/articles/feifei-li-tract-visual-tracks-robot-world-model]
confidence: 0.72
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# TrAct：视觉轨迹作为机器人控制与世界模型之间的共同语言（李飞飞团队）

## 核心命题

机器人动作指令适合让机器人执行，却未必适合让世界模型预测未来——动作是低维、与具体本体绑定的控制指令（Franka 和 UR5 即便完成相似操作也可能需要不同指令），且同一动作的视觉结果取决于场景几何/物体状态/接触动态。李飞飞、吴佳俊团队提出把**视觉轨迹（Visual Tracks）**放在策略模型与世界模型之间作为信息接口：策略同时输出动作和对应视觉轨迹，世界模型根据轨迹预演未来，让"先预演再行动"进入任务评估闭环。^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md]

## 架构：VLAT → TWM → VLAC

- **VLAT**（建立在 π0.5 上）：根据当前观测 + 语言指令，联合生成多组动作与视觉轨迹（成对输出，轨迹非由动作转换）。视觉轨迹描述任务相关点随时间在图像空间中的二维运动（夹爪 7 关键点 + 腕部视角 5×5 均匀采样场景点）。^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md]
- **TWM**（轨迹条件世界模型）：以 Stable Video Diffusion 为骨干，ControlNet 把视觉轨迹编码为空间控制条件，生成未来视频。对照的**AWM**是动作条件世界模型，用相同预训练/微调数据。^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md]
- **VLAC**：候选视频生成后按任务指令逐一打分，执行最高分对应动作（每决策步仿真采样 20 组、真机 16 组）。^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md]

## 关键结果

- **仿真**：LIBERO-INTEGRAL 上 π0.5 成功率 27% → VLAT 44% → +AWM 49% → **TrAct 55%**；三随机种子平均 54.7% vs VLAT+AWM 49.0%（95% CI 不重叠）。跨本体 Franka→UR5：π0.5 17% → **TrAct 50%**。^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md]
- **真机**（Franka 400 演示微调）：π0.5 / VLAT+AWM / TrAct 平均成功率 49% / 66% / **76%**；未见背景下差距更大（关柜门 VLAT+AWM 40% vs TrAct 70%）。^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md]
- **视频预测质量**：TWM 在五项视频指标全优于 AWM（仿真外部视角 PSNR 15.12→24.51、FVD 129→38）。^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md]

## 技术价值

TrAct 的贡献不在"换了个模型"，而在**换了一层信息接口**：视觉轨迹是一种跨本体的"共同语言"——一头连机器人控制、一头连世界模型的未来预测。统一轨迹表示还让 DROID 机器人数据与 EgoDex 人类第一视角视频共享同一轨迹预测目标（人手动作与机器人动作无法用同一控制参数表示，只用轨迹监督）。扩大预训练数据后 TrAct+ 在最难 5 个 UR5 任务上 32%→38%。^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md]

## 深度分析

### 动作接口失败的根因：低维指令无法约束稠密视觉未来

论文对"动作为什么不是理想接口"给出了三层拆解：动作是低维且本体绑定的控制指令（Franka 与 UR5 完成相似操作需要不同指令）；同一动作的视觉结果取决于场景几何、物体状态与接触动态；已有研究甚至发现动作条件视频模型会忽略错误动作、依赖视觉先验生成"看似合理"的结果——即世界模型从稀疏本体指令推断稠密视觉未来时，约束过弱导致幻觉式预测。视觉轨迹直接描述任务相关点在图像空间的运动，等于把预测目标从"猜测这段指令会产生什么"变成"外推这条已经在图像空间画出的路径"。 ^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md:45]

### 消融链证明收益来自轨迹条件而非候选筛选

LIBERO-INTEGRAL 的递进消融（π0.5 27% → VLAT 44% → VLAT+AWM 49% → TrAct 55%）设计得很干净：VLAT+AWM 与 TrAct 共用同一套候选生成与筛选流程，唯一变量是世界模型的条件信号（动作 vs 轨迹）。49% → 55% 的增益因此只能归因于轨迹条件；三随机种子重评（54.7% vs 49.0%，95% CI 不重叠）排除了运气。世界模型自身质量同步验证了这一点：TWM 在五项视频指标上全面优于 AWM（PSNR 15.12→24.51、FVD 129→38）。 ^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md:87]

### 轨迹作为跨本体语言解锁了异构数据联合训练

统一轨迹表示的更深层价值在数据侧：EgoDex 人手视频与 DROID 机器人数据无法共享动作参数空间，但可以共享同一轨迹预测目标——人手数据只监督轨迹，机器人数据同时监督轨迹与动作。TrAct+ 扩大预训练后在最难 5 个 UR5 任务上 32%→38%。这把"跨本体泛化"（Franka→UR5 17%→50%）从架构问题部分转化为数据问题，人类视频第一次能以受监督的方式直接喂给机器人策略预训练。 ^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md:121]

### 收益与任务难度、分布偏移正相关

标准 LIBERO 已饱和（π0.5 96.8% / TrAct 98.3%），接口改良无处发力；LIBERO-INTEGRAL（物体替换/位置/视角/初始状态扰动）与真机未见背景下差距才拉开（关柜门 VLAT+AWM 40% vs TrAct 70%）。规律是：分布偏移越大，轨迹的空间约束价值越高——因为视觉先验在 OOD 场景失效，而显式轨迹约束不依赖先验。 ^[raw/articles/feifei-li-tract-visual-tracks-robot-world-model.md:97]

## 实践启示

1. 设计"预演再行动"系统时，优先选视觉轨迹而非动作作为世界模型条件信号——条件信号应携带图像空间中的位置信息，而非本体相关的低维指令。
2. 在已饱和的基准上评估接口改良没有意义；主动构造分布偏移（背景、物体、视角、跨本体）的评估集，收益只在 OOD 处显现。
3. 做消融时隔离条件信号这一单一变量（共用候选生成与筛选流程），否则无法区分"筛选带来的收益"与"预测质量带来的收益"。
4. 具身数据策略上，把人类第一视角视频纳入预训练时用轨迹监督对齐目标，不必强行映射人手动作到机器人控制参数。
5. 轨迹表示的选择（关键点数、采样密度）直接决定约束强度：TrAct 用夹爪 7 关键点 + 腕部 5×5 场景点，为场景与相机运动提供参照，可按任务复杂度调整。

## 相关

- [[entities/feifei-li-masked-visual-actions-world-model-2026|掩码视觉动作（李飞飞团队世界模型）]]
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld 跨本体物理世界模型]]
- [[entities/eccv-2026自驾vla-scaling有戏了北航清华driveteach-vla用图像轨迹打通驾驶场景与基模预训练|DriveTeach-VLA 图像轨迹驾驶]]
- [[entities/amap-abot-earth-0.5-3d-native-world-model|Amap ABOT Earth 3D 原生世界模型]]
- [[entities/egosuite-open100k-embodied-human-video-data-open-source-2026|EgoSuite-Open100K 具身视频数据]]

→ [[raw/articles/feifei-li-tract-visual-tracks-robot-world-model|原文存档]]
