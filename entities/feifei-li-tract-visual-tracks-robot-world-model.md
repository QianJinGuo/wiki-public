---
title: "TrAct：视觉轨迹作为机器人控制与世界模型之间的共同语言（李飞飞团队）"
created: 2026-08-28
updated: 2026-08-28
type: entity
tags: [world-model, robotics, embodied-ai, visual-tracks, fei-fei-li, stanford, video-model]
sources: [raw/articles/feifei-li-tract-visual-tracks-robot-world-model]
confidence: 0.72
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

## 相关

- [[entities/feifei-li-masked-visual-actions-world-model-2026|掩码视觉动作（李飞飞团队世界模型）]]
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld 跨本体物理世界模型]]
- [[entities/eccv-2026自驾vla-scaling有戏了北航清华driveteach-vla用图像轨迹打通驾驶场景与基模预训练|DriveTeach-VLA 图像轨迹驾驶]]
- [[entities/amap-abot-earth-0.5-3d-native-world-model|Amap ABOT Earth 3D 原生世界模型]]
- [[entities/egosuite-open100k-embodied-human-video-data-open-source-2026|EgoSuite-Open100K 具身视频数据]]

→ [[raw/articles/feifei-li-tract-visual-tracks-robot-world-model|原文存档]]
