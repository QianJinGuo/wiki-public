---
title: GenCeption — Video Generation Models are General-Purpose Vision Learners
created: 2026-07-15
updated: 2026-07-27
type: entity
tags: [computer-vision, video-generation, foundation-model, multi-task, eccv-2026, paper]
sources: [raw/articles/genception-video-generation-general-purpose-vision-learner-2026]
confidence: 0.7
---

GenCeption 是一项由 Google DeepMind、MIT、牛津大学等机构合作完成的 ECCV 2026 论文工作，作者包括何恺明、Andrew Zisserman 等视觉领域的代表性研究者。其核心目标是将大规模文生视频模型改造为通用的视频感知模型，用单一架构统一处理深度、分割、姿态和三维几何等多种视觉任务。这一思路试图回答一个根本性问题：视频生成是否就是视觉领域一直在寻找的"下一个 token 预测"？^[raw/articles/genception-video-generation-general-purpose-vision-learner-2026.md]

论文的核心洞察在于，视频生成同时具备理解语义、空间和时间信息的能力，且能随数据、模型规模和算力持续扩展——这与语言模型中"下一个 token 预测"的统一范式高度相似。GenCeption 基于开源文生视频模型 WAN 2.1，通过将时间步固定为 t=0，仅执行一次 DiT 前向计算，将多步生成过程压缩为单步感知推理，从而复用生成阶段积累的丰富视觉先验。^[raw/articles/genception-video-generation-general-purpose-vision-learner-2026.md]

在方法层面，GenCeption 将所有任务统一在共享的预训练 DiT 主干之上。深度、法线、分割等稠密任务编码为三通道图像，相机射线图通过空间分区重新编码为 RGB 表示；稀疏任务（如 3D 关键点）则引入可学习 token 并由 MLP 输出坐标。所有任务采用统一的 L2 损失，仅通过文本提示和目标数据表示加以区分。这种设计使得单套架构即可覆盖深度估计、表面法线、相机位姿、前景分割、指代表达式分割和 3D 人体关键点等多种任务。^[raw/articles/genception-video-generation-general-purpose-vision-learner-2026.md]

实验结果表明，GenCeption 在多项视觉任务上达到或接近专用模型的水平，且在深度估计上呈现出随模型规模和后训练数据增加而提升的趋势。14B 参数版本在单块 v6e TPU 上处理 81 帧 480×832 分辨率视频可达约 8 FPS。加载更多生成预训练层能显著改善收敛，证明性能提升主要来自生成预训练所学的可迁移视觉表示，而非仅仅依赖后训练数据和模型结构。^[raw/articles/genception-video-generation-general-purpose-vision-learner-2026.md]

尽管 GenCeption 目前仍聚焦于感知任务，且 14B 模型的资源需求较高、多任务联合训练在某些任务上仍有退化，但它为视觉领域的统一预训练范式提供了重要启示：文生视频模型积累的视觉先验可以通过后训练高效迁移到多样化的下游任务中，这使其成为通向通用视觉智能的一个有前景的方向。^[raw/articles/genception-video-generation-general-purpose-vision-learner-2026.md]

→ [[raw/articles/genception-video-generation-general-purpose-vision-learner-2026|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

