---
title: "PhyEdit：显式 3D 几何 Preview 指导 DiT 图像编辑（浙大 ReLER，ACM MM 2026）"
created: 2026-08-13
updated: 2026-09-07
type: entity
tags: [3d-editing, image-editing, diffusion, dit, geometry, acm-mm26, zju-reler, world-model, multimodal]
sources: [raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026]
confidence: 0.65
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# PhyEdit：显式 3D 几何 Preview 指导 DiT 图像编辑

> PhyEdit 由浙江大学 ReLER 团队提出，论文被 ACM MM 2026 接收，代码、模型、数据集与 GUI 全部开源。核心思想：**让几何模块负责"搬"，让生成模型负责"画"**——先用显式 3D 几何 preview 表达物体的位置/尺度/遮挡，再交给生成模型恢复纹理与身份，避开"纯 prompt 盲猜三维空间"与"点云投影直接当结果"两个极端。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]

## 方法：四步管线 + 深度监督

- **四步流程**：用户给定物体 + 3D 操作指令（移动或 6DOF）→ 估计场景深度与相机参数，物体反投影为三维点云 → 在 3D 空间中移动点云并投影成目标 preview → 源图 + preview + 文本送入 Qwen-Image-Edit backbone 生成最终图像。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]
- **preview 定位**：preview 只需清楚表达"物体应在哪里、多大、挡住谁"，不需要像完整照片；生成模型负责从源图恢复纹理和身份。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]
- **像素级 SILog depth loss**：在基础 flow-matching loss 之外，先从预测 velocity 恢复编辑图，再比较编辑图与目标图深度。消融：无深度监督 DIoU 62.37 / Chamfer 24.52 → latent-to-depth 64.19 / 20.87 → pixel-level 65.33 / 18.93，逐级提升。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]

## 数据与评测

- **RealManip-40K 数据集**：41154 对真实场景图像，含深度、物体 mask、代表性 3D 坐标。数据管线用 3D foundation model 的 camera token 聚类筛选相机近似静止的视频片段，再完成检测/跟踪/分割/深度估计/3D 位移筛选；重点覆盖远近变化、复杂遮挡、多对象同时操作三类难题。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]
- **ManipEval 基准**：200 对图像约 320 个物体（一半单物体、一半多物体操作），五维考察：2D 落点、深度正确性、3D 点云接近度、身份/画质保留、光照接触遮挡合理性。PhyEdit 成绩：DIoU 65.33、Mask IoU 27.20、Chamfer 18.93、RA-DINO 36.91、Phys-VLM 93.72。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]
- **vs Nano Banana Pro**：DIoU +5.36、Chamfer −6.40、RA-DINO +2.14。商业模型常见失败模式：物体滞留源位置、多物体指令只操作部分、目标深度错误、遮挡区身份改变。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]

## 能力边界与延伸

- **连续动作**：给定 3D 轨迹后，PhyEdit 在轨迹多位置生成关键状态，由视频模型插值中间帧；机械臂未出现在训练分布中仍能沿曲线移动物体。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]
- **普通编辑不退化**：外观编辑 + 3D 操作同时完成综合成功率 87.5% vs Qwen-Image-Edit 单独普通编辑 88.8%，仅差 1.3 个百分点，几何控制未大幅牺牲基础编辑能力。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]
- **定位克制**：不是完整 physics simulator（不显式计算受力/碰撞/动力学）；透明反光物体、极端近景、严重深度/分割错误仍会失败。"显式几何 + 生成渲染"组合为机器人视觉规划、交互式内容、图像状态预测提供可复现开源起点。^[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026.md]

## 资源

- 论文：https://arxiv.org/abs/2604.07230
- 项目主页：https://nenhang.github.io/PhyEdit
- 代码：https://github.com/nenhang/PhyEdit
- 模型：https://huggingface.co/ruihangxu/PhyEdit
- 数据集：https://huggingface.co/datasets/ruihangxu/RealManip-40K

## 相关

- [[entities/diffusion-model-consistency-framework-2026-survey|Diffusion Model 一致性框架综述]] — PhyEdit 的 flow-matching 训练属于扩散模型家族
- [[entities/nano-banana-2-lite-gemini-omni-flash-google-deepmind-2026|Nano Banana 2]] — ManipEval 对比对象（商业闭源模型）
- 扩散模型架构 — DiT backbone 基础
- [[entities/amap-abot-earth-0.5-3d-native-world-model|3D Native World Model]] — 3D 空间理解相关

→ [[raw/articles/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026|原文存档]]
