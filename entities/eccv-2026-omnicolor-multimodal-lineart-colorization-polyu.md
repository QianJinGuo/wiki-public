---
title: "ECCV 2026｜OmniColor：统一多模态线稿上色框架"
created: 2026-08-28
updated: 2026-08-28
type: entity
tags: [eccv-2026, omnicolor, image-generation, video-generation, lineart-colorization, computer-vision, multimodal, hk-polyu]
sources: [raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu]
confidence: 0.7
---

# ECCV 2026｜OmniColor：统一多模态线稿上色框架

## 核心命题

动画/美术生产中，线稿上色的难点不是"涂颜色"，而是**多条件输入的协同控制**——现实创作中控制信号总是混合出现（文字描述、稀疏颜色提示、角色参考图、近期/远期历史帧），且彼此可能冲突（参考图想保持角色风格、局部色块涂鸦却要求某区域改色）。现有方法大多围绕单一或少量控制信号设计，缺少统一机制理解多条件输入下的控制对齐。ECCV 2026 接收的 OmniColor 提出把复杂控制信号拆成两类（空间对齐条件 vs 语义参考条件），据此设计针对性编码/训练/冗余消除/自适应门控，支持**任意组合**的多模态线稿上色。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]

## 方法：条件二分 + 差异化处理

- **条件二分**：空间对齐条件（输入线稿、颜色提示、近期历史帧）与目标图像存在强空间对应；语义参考条件（文本描述、角色身份参考图、长期历史帧）提供抽象语义约束、不要求逐像素对齐。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]
- **双编码器（空间对齐条件）**：VAE 编码器保留颜色边界/高频细节 + VLM 编码器提取局部语义，让模型既关注线条/色块位置又理解局部语义归属（头发/衣服/背景）。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]
- **仅 VLM 编码（语义参考条件）**：压缩后的语义 token 已够表达关键属性，减少 token 数提升推理效率。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]

## 三个关键模块

1. **Dense Feature Alignment（DFA）**：用 DINOv3 提取预测/真实图像稠密特征做 MSE 对齐，强化空间约束；因早期生成噪声大，监督限制在较晚去噪阶段。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]
2. **Temporal Redundancy Elimination（TRE）**：相邻帧大量重复静止内容。把参考帧分 patch、按颜色直方图相似度过滤冗余区域，连通域分析取最小包围框，只编码"发生变动的局部区域"。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]
3. **Adaptive Spatial-Semantic Gating（AS-Gate）**：根据空间分支信息密度动态调节语义分支影响（空间约束强则降低语义干扰），集成进 MMDiT 注意力结构，用零初始化 LoRA 保证训练初期接近恒等映射。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]

## 实验与结果

基于 25 部高质量动画构建训练数据（12 万对样本）；测试集 6 部未见动画（305 视频片段 + 900 图像样本）。与三类方法比较在相同输入下表现更高，且支持控制信号自由组合。身份参考在旋转/镜头推拉/视角变化下仍能稳定保持角色面部/发色/服饰一致。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]

## 技术定位

OmniColor 属于**受控图像/视频生成**子领域——以线稿为结构底、多模态信号为控制输入的生成任务。与一般文生图不同，核心难点在"结构与语义控制的解耦 + 条件冲突消解"，对需要强一致性约束的生成式视觉应用有方法论参考价值。论文/代码均开源（arxiv 2603.27531 / github zhangxulu1996/OmniColor）。^[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu.md]

## 相关

- [[entities/cft-consistent-feature-transport-image-relighting-eccv-2026-meitu|CFT 图像重打光]]
- [[entities/eccv26-oral人物不能变姿势要对齐风格还得一致dyref突破多参考约束下的图像生成难题|DyRef 多参考图像生成]]
- [[entities/omnishow-unified-multimodal-video-generation-icml-2026|OmniShow 统一多模态视频生成]]
- [[entities/alaya-evoke-hour-level-video-world-model|Alaya 小时级视频世界模型]]

→ [[raw/articles/eccv-2026-omnicolor-multimodal-lineart-colorization-polyu|原文存档]]
