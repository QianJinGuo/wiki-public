---
title: "3D 指标超过 Nano Banana Pro：浙大开源 PhyEdit 让 AI 在平面图像里进行立体编辑"
created: 2026-08-15
updated: 2026-09-26
type: entity
tags: [ai, image-editing, 3d-editing, diffusion, dit, acm-mm26, zju-reler, qwen, multimodal]
sources: [raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 浙大 PhyEdit：用显式 3D 几何约束让 AI 在平面图像里做立体编辑

PhyEdit 是浙江大学 ReLER 团队提出并开源的图像编辑方案，论文被 ACM MM 2026 接收。针对"语义指令听懂了、但生成出的三维状态常错得离谱"这一图像编辑痛点，PhyEdit 用**显式 3D 几何 preview 指导 DiT 图像编辑**，让模型更准确地处理物体远近、尺度、遮挡和多物体操作，在 ManipEval 的 3D 指标上超过商业闭源的 Nano Banana Pro。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md]

核心哲学一句话概括：**让几何模块负责"搬"，让生成模型负责"画"**。preview 不需要像完整照片，只要清楚表达"物体应该在哪里、应该多大、应该挡住谁"，生成模型就能从源图恢复纹理和身份，把粗糙几何修成自然结果，从而避开"纯 prompt 盲猜三维空间"与"点云投影直接当最终图像"两个极端。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md]

## 方法：四步管线与像素级深度监督

PhyEdit 核心流程拆成四步：用户给定待操作物体和三维操作指令（移动或 6DOF）；估计场景深度和相机参数，把物体反投影成三维点云；在 3D 空间中移动点云再投影成目标 preview；将源图、preview 和文本一起交给 Qwen-Image-Edit backbone 生成最终图像。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md]

在基础 flow-matching loss 之外，PhyEdit 加入**像素级 SILog depth loss**：先从预测 velocity 恢复编辑图，再比较编辑图和目标图的深度。消融结果直接：不使用深度监督 DIoU 62.37 / Chamfer 24.52；latent-to-depth 64.19 / 20.87；pixel-level 方案 65.33 / 18.93，逐级提升。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md]

## 数据与评测：RealManip-40K 与 ManipEval

团队构建了 **RealManip-40K 数据集**（41154 对真实场景图像，提供深度、物体 mask 和代表性三维坐标）。数据管线利用 3D foundation model 的 camera token 聚类，筛选相机近似静止的视频片段，再完成目标检测、跟踪、分割、深度估计和三维位移筛选，保证图像对变化主要来自物体而非相机；重点覆盖明显远近变化、复杂遮挡、多对象同时操作三类难题。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md]

**ManipEval 基准**包含 200 对图像、约 320 个物体，一半单物体、一半多物体操作，从五个层面考察：2D 落点、深度正确性、重建 3D 点云接近度、身份与画质保留、光照接触遮挡合理性。PhyEdit 主要成绩：DIoU 65.33、Mask IoU 27.20、Chamfer 18.93、RA-DINO 36.91、Phys-VLM 93.72；与 [[entities/nano-banana-2-lite-gemini-omni-flash-google-deepmind-2026|Nano Banana 系列]] 的 Nano Banana Pro 相比，DIoU 提升 5.36、Chamfer 距离降低 6.40、RA-DINO 提升 2.14。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md]

## 能力边界与延伸

PhyEdit 还能把一次编辑变成连续动作：给出三维轨迹后，在轨迹多个位置生成关键状态，再由视频模型插值中间帧；未出现在训练分布中的机械臂也能沿曲线移动物体。普通编辑能力基本保住：外观编辑 + 3D 操作同时完成的综合成功率 87.5%，而 Qwen-Image-Edit 单独普通编辑为 88.8%，仅差 1.3 个百分点。团队还开源了交互式 GUI，可分割选择多个物体、在 3D 点云中调整平移旋转后生成图片。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md]

团队定位克制：PhyEdit 不是完整 physics simulator，不显式计算受力、碰撞和动力学，透明反光物体、极端近景移动、严重深度或分割错误仍可能导致失败。它解决的是更具体的问题——当动作已由用户指定时，如何把一个明确的三维操作渲染成几何合理的视觉状态。这种"显式几何 + 生成渲染"组合，为机器人视觉规划、交互式内容和图像状态预测提供了可复现的开源起点。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md]

## 深度分析

### 为什么"盲猜式"编辑在 3D 场景上必然失败

图像编辑模型在语义层面并不弱——指令能听懂、画质能保证，但一涉及真实三维状态就系统性翻车：物体移近后尺寸不变、遮挡关系算错、原位置留残影。这不是单点 bug，而是架构性的：模型只能从文本 token 与二维像素分布的相关性去"猜"三维，而深度、遮挡、尺度本质上是需要外部几何信号才能确定的量。PhyEdit 的诊断是把问题拆成两半——语义理解交给大模型，几何确定性交给显式 3D preview——从根源上绕开"让大模型完全靠 prompt 猜三维空间"这个极端。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:19-27]

### Preview 粗糙也无妨：几何只需表达"布局"，纹理由源图恢复

这条管线里最反直觉的设计是 preview 的宽容度：它不需要看起来像一张完整照片，只要说清楚"物体应该在哪里、应该多大、应该挡住谁"就够了。生成模型从源图恢复纹理与身份，把粗糙几何修成自然结果。这实质上是一种关注点分离——几何模块承担全部空间决策，生成模型承担全部外观合成，两者各取所长。它同时避开了另一个极端（把点云投影直接当最终图像），使方法对深度估计和分割的精度不必苛求。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:48-50]

### "看着像"不等于"深度对"：像素级深度监督的消融证据

图像编辑评测中常见的画质与文本一致性指标，恰恰掩盖了三维正确性错误——画面观感相似但深度错位。PhyEdit 在 flow-matching loss 之外加入像素级 SILog depth loss（从预测 velocity 恢复编辑图后直接比较深度），消融阶梯非常清晰：无深度监督 DIoU 62.37 / Chamfer 24.52，latent 层间接监督 64.19 / 20.87，像素级监督 65.33 / 18.93，逐级提升。这说明监督信号的"位置"本身是关键——越接近最终输出像素，几何误差越难在生成过程中被稀释。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:52-58]

### 数据管线的真正难点不是规模，而是"把相机静止下来"

RealManip-40K 用 41154 对真实场景图像训练，但它的工程价值不在数量，而在如何保证图像对之间的变化只来自物体而非相机运动。管线用 3D foundation model 的 camera token 聚类筛出相机近似静止的视频片段，再串联检测、跟踪、分割、深度估计与三维位移筛选，并刻意覆盖远近变化、复杂遮挡、多对象操作三类难题。这个设计直接回应了评测暴露的商业模型失败模式：物体留在原位、只完成多物体指令的一部分、目标放到错误深度、遮挡区改变物体身份——每一项都是训练数据里缺乏纯物体位移样本的后果。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:60-68]

### 定位克制的"几何渲染器"：不做物理引擎，反而更接近可复现的起点

团队对边界的界定很清醒：PhyEdit 不是完整 physics simulator，不显式计算受力、碰撞与动力学；透明反光物体、极端近景移动、严重深度或分割错误仍会失败。它解决的是一个定义狭窄但可验证的问题——当动作已由用户指定时，把明确的三维操作渲染成几何合理的视觉状态。普通编辑能力的保持（综合成功率 87.5%，与 Qwen-Image-Edit 单独普通编辑的 88.8% 仅差 1.3 个百分点）证明加入几何控制没有让基础能力退化。这种"窄而深"的定位，恰是它能为机器人视觉规划、交互式内容与图像状态预测提供可复现开源起点的原因。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:109-127]

## 实践启示

1. **给生成模型外挂显式几何信号，而不是在 prompt 里描述几何。** 当编辑任务涉及空间关系（移动、遮挡、尺度）时，与其让模型从文本猜三维，不如先构造粗粒度的 preview（位置、大小、遮挡关系），再把源图、preview 和文本一起交给模型。PhyEdit 的四步管线即此范式。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:33-41]

2. **评测 3D 类生成任务时，必须设计显式几何指标。** 只看画质与文本一致性的 benchmark 会系统性漏判深度错误。ManipEval 从 2D 落点、深度正确性、重建点云接近度、身份画质保留、光照遮挡合理性五个层面出题的做法，可直接移植到自建的图像/视频编辑评测中。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:70-84]

3. **深度监督要加在像素级，而非 latent 层。** 消融显示 pixel-level SILog depth loss 相比 latent-to-depth 再带来 DIoU +1.14、Chamfer -1.94 的增益。凡是输出与几何一致性相关的生成任务，监督信号应尽可能靠近最终输出空间。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:54-58]

4. **构建"物体位移"训练数据时，先用相机静止过滤换取因果纯净。** 借助 3D foundation model 的 camera token 聚类筛选相机近似静止的片段，能保证图像对变化主要来自物体而非相机——这是用数据工程解决监督歧义的典型手段，比事后清洗便宜得多。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:62-68]

5. **开源完整交互链路（模型 + 数据集 + GUI）会显著放大研究影响力。** PhyEdit 同步开源了代码、RealManip-40K 数据集、模型权重和可分割多物体、调整 3D 点云的交互式 GUI，使他人能直接复现与扩展；做开源项目时应把"可上手"视为交付物的一部分。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:117-119]

6. **把单次编辑扩展成连续动作时，用"关键帧生成 + 视频插值"组合而非端到端生成。** 给定三维轨迹后先生成多个关键状态，再由视频模型插值中间帧——这条路径甚至能让未出现在训练分布中的机械臂沿曲线移动物体，是低成本衔接图像编辑与视频生成的实用模式。^[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26.md:99-107]

## 相关

- [[entities/phyedit-explicit-3d-geometry-preview-image-editing-acm-mm26-2026|PhyEdit 英文条目]] — 本实体对应的英文条目
- [[entities/nano-banana-2-lite-gemini-omni-flash-google-deepmind-2026|Nano Banana 系列]] — ManipEval 对比对象
- 视频生成模型 — 连续动作编辑依赖视频模型插值

→ [[raw/articles/3d指标超过nano-banana-pro浙大开源方案让ai在平面图像里进行立体编辑-acm-mm26|原文存档]]
