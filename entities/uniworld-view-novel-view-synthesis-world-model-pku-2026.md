---
title: "UniWorld-View：登顶 WorldScore 的世界模型（单图/视频新视角合成）"
created: 2026-08-27
updated: 2026-09-07
type: entity
tags: [ai, world-model, vision, novel-view-synthesis, 3d, video-generation, pku, open-source]
sources: [raw/articles/uniworld-view-novel-view-synthesis-world-model-pku-2026]
confidence: 0.68
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# UniWorld-View：登顶 WorldScore 的世界模型（单图/视频新视角合成）

> **v×c score**: 56 | stars=4
> **来源**: [[raw/articles/uniworld-view-novel-view-synthesis-world-model-pku-2026|UniWorld-View 原文存档]]

兔展智能联合北京大学、鹏城实验室研发的 **UniWorld-View** 登顶李飞飞团队 WorldScore 世界模型榜单（训练数据来自北大系初创企业元空智能），并完成国产昇腾算力适配与代码、权重开源。^[raw/articles/uniworld-view-novel-view-synthesis-world-model-pku-2026.md]

## 核心能力：统一的新视角合成

UniWorld-View 回答的不是"如何生成更漂亮的图"，而是**如何理解没有被拍到的世界**——给定单图或单段视频，按指定相机轨迹生成新视角画面（推拉摇移、360°环绕），保持较好空间一致性。无论是单图 3D 生成还是视频 4D 生成，都采用同一套模型、同一套代码，实现"Uni（统一）World-View"。^[raw/articles/uniworld-view-novel-view-synthesis-world-model-pku-2026.md]

## 技术难点：大基线新视角合成

新视角合成（Novel View Synthesis）本质是让 AI 根据有限视角推断未拍摄空间并生成新画面，难点在**大基线（large-baseline）**——只给一张正脸，推断侧脸与后脑勺，同时保证空间结构、遮挡关系与物体位置一致。^[raw/articles/uniworld-view-novel-view-synthesis-world-model-pku-2026.md]

- **重建路线**（NeRF、3DGS）：依赖大量多角度数据，单目视频视角覆盖不足，易产生空洞伪影。
- **纯生成路线**：多数方法把相机位姿当抽象条件向量塞进扩散模型，无显式 3D 建模，转大角度会失控。
- **几何+生成混合**（UniWorld-View 采用）：先用几何模型把画面撑成 3D 点云，再把目标视角的点云渲染图喂给视频扩散模型当条件——几何归几何、生成归生成，相机控制变准。同类路线有 ViewCrafter、英伟达 GEN3C。

## 与既有世界模型家族的关系

UniWorld-View 属**几何约束的生成式世界模型**路线，与以下既有 entity 互补：[[entities/feifei-li-masked-visual-actions-world-model-2026|李飞飞 masked-visual-actions 世界模型]]、[[entities/loopwm-looped-world-models|LoopWM 循环世界模型]]、[[entities/amap-abot-earth-0.5-3d-native-world-model|高德 3D 原生城市世界模型]]、[[entities/yann-lecun-jepa-world-model|LeCun JEPA 世界模型]]。它聚焦的是「视频/图像的空间外推」（novel view），而非动作预测或城市级重建，填补了 wiki 在**新视角合成世界模型**子领域的空白。

## 相关方向

- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- 机器人具身 AI
- [[entities/world-model-evaluation-position-paper-nju-2026|世界模型评估立场论文（南大）]]

→ [[raw/articles/uniworld-view-novel-view-synthesis-world-model-pku-2026|原文存档]]
