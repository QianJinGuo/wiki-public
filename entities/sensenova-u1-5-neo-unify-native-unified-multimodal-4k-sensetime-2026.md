---
title: "SenseNova-U1.5：商汤 NEO-unify 原生统一多模态训练法（4K 生成 + 多专家 OPD）"
created: 2026-09-18
updated: 2026-09-18
type: entity
tags: [multimodal, native-unified, neo-unify, mixture-of-transformers, flow-matching, 4k-generation, on-policy-distillation, sensenova, sensetime]
sources: [raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026]
confidence: 0.85
provenance_state: extracted
---

# SenseNova-U1.5：商汤 NEO-unify 原生统一多模态训练法（4K 生成 + 多专家 OPD）

商汤日日新 U 系列的 SenseNova-U1.5 是 8B-MoT 原生统一多模态模型，延续 NEO-unify 架构路线：去掉独立视觉编码器与 VAE，让理解、推理与像素生成在同一套模型中学习；原生图像生成最高扩展到 4K，同时覆盖双语文字渲染、信息图、多参考图生成与编辑。论文报告登上 Hugging Face 论文日榜第 2，官方开放主模型权重、SFT 权重与 8-step 蒸馏 LoRA（后者支持 8 个去噪步骤完成文生图推理）。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

## 问题定义：统一之后如何继续扩展

原生统一的难点已从「能力共存」转向「统一之后如何扩展」——高分辨率生成需要保住低层纹理与空间连续性，理解依赖高层语义，文字、美学与编辑由不同奖励信号驱动；分辨率越高，图像块边界与局部几何问题越难隐藏，任务越复杂，共同优化越容易稀释单项能力。U 系列上一代（U1）验证了原生统一架构可行性，U1.5 回答的是扩展路径。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

## 视觉接口重做：空间联合重建撑起 4K

U1 采用极紧凑视觉表示（每个 32×32 像素区域对应一个视觉 token），但在解码端每个 token 独立还原自己的图像块，缺少最后的空间信息交换，分辨率上升后接缝、纹理不连续与局部几何偏差放大。U1.5 保留 32×32 的 token 粒度，重做解码端：视觉 token 先还原成二维特征场，相邻区域通过 3×3 卷积交换信息，再经多级 Pixel Shuffle 逐步恢复 RGB——颜色、纹理与几何关系在像素确定之前完成邻域联合建模。同时把分辨率本身纳入去噪条件，分辨率感知的噪声条件扩展到 4096×4096，让模型在生成过程中同时感知当前去噪阶段与目标分辨率对应的噪声统计。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

## 架构与训练目标：MoT 共享注意力 + 三类监督联合目标

U1.5 延续原生 Mixture-of-Transformers（MoT）设计：干净图像、文本与带噪视觉状态进入统一序列，通过共享自注意力直接交互，而理解与生成各自保留注意力投影、归一化和前馈网络等任务专属参数——共享一套视觉表示，但保留各自所需的计算模块。训练目标上，语言侧用自回归目标学习理解与推理，图像侧直接在 RGB 像素空间做流匹配（Flow Matching），再加入 LPIPS 感知约束校正整体结构与局部纹理，三类监督进入同一个联合目标。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

## 数据与训练流程：四类任务 30/40/20/10

生成数据新增约 5900 万组图文对（来自 78 个来源），高分辨率占比很高：约 88.2% 有效训练量超过 1024²,64.4% 超过 2048²，并加强中英文文字密集图像、复杂版式与稀有视觉概念。训练流程专门留出原生 4K 阶段：生成分支先从 256²—1024² 起步，再扩展到 512²—4096²，随后加入图像编辑与图文交错任务。进入统一中期训练后，理解、文生图、编辑、图文交错四类数据以 30%、40%、20%、10% 比例共同训练，最大序列长度提升到 32768；理解与生成损失权重设为 0.1 : 1.0，在保留预训练理解能力的同时把优化力度更多放到更难的生成目标上，并在随后的统一 SFT 阶段继续保持。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

## 编辑与图文交错：结构化约束 + 多模态轨迹

编辑侧数据约 3800 万组，覆盖通用编辑、信息图编辑、参考图驱动编辑与空间控制编辑；多参考样本最高可含 10 张参考图，区域控制进一步加入边界框与视觉标记。对多目标多约束指令，训练数据显式拆出编辑对象、目标区域、属性变化、空间与语义约束以及必须保持不变的内容，并加入结构化提示增强与显式推理样本以降低复杂指令歧义。图文交错数据把教程、生活场景、信息图、视频序列与推理样本组织成文字与图像持续交替的多模态轨迹，其中约 44% 来自生活与教程场景、29% 为信息图、19% 来自视频序列、约 8% 带显式中间推理——图片反复出现在同一连续任务中。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

## 后训练核心升级：先专业化、再统一（specialize-then-unify）

任务越接近真实创作，目标冲突越难忽略（只追求美感文字可能受损，编辑动作过强会破坏原图，信息图同时依赖文字准确、布局组织与整体视觉质量）。U1.5 把后训练改成先专业化再统一：美学、OCR、信息图和编辑分别训练四个专家模型，再把各自能力通过在线策略蒸馏（OPD）汇回统一模型，报告将其列为 U1.5 的主要后训练升级。四个专家优化目标不同、训练方式分开——美学专家围绕视觉偏好与图文对齐并穿插文字任务避免损伤可读性；OCR 专家借助 PaddleOCR 检查生成文字，配合 Precise 采样与 GRPO-Guard 在稀疏 OCR 奖励下稳定训练。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

## 实验结果：生成、编辑与双向迁移

| 评测 | 结果 |
|---|---|
| 4K 文字渲染综合分 | 0.948（LongText-Bench 中/英 0.988 / 0.989） |
| IGenBench 信息图 | 不加 PE 即报告内开源最佳；加 PE 后 Q-ACC 0.62 → 0.76 |
| ImgEdit 通用编辑 | 综合 4.59，表中参评模型最高 |
| MMLU-Pro / C-Eval / IFEval | 86.67 / 90.41 / 93.35 |
| WISE（直接生成 → +CoT） | 0.70 → 0.81 |
| RISEBench（+CoT） | 33.6 → 38.6（时间/因果/逻辑类收益明显；空间类 49.0 → 42.0 反降） |
| VBVR-Pro 图文交错 | 68.2（Nano-Banana-Pro 56.4、GPT-Image-2 50.7），域外平均 68.9 |
| RealUnify-GEU | 56.3（上一代 U1-SFT 47.5） |

^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

关键结论是能力双向作用开始显现：理解与推理中形成的结构知识和规划能力可以进入视觉生成（长指令、复杂组合、高度结构化视觉要求下的泛化）；反过来生成也开始作为中间表征参与后续理解与推理。CoT 的收益具有任务依赖性——对时间、因果、逻辑类编辑有效，对可直接依据空间关系完成的编辑反而负收益。研究团队表示将进一步开放监督微调、强化学习与在线策略蒸馏的训练代码。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md]

## 与已有覆盖的关系

- 架构脉络：MoT 共享注意力 + 任务专属参数的多模态骨架与 [[concepts/moe-mixture-of-experts-2025|MoE 混合专家]] 属于同一族设计，但 U1.5 的专家化发生在后训练阶段（OPD 汇回）而非前向路由。
- 生成范式：RGB 像素空间流匹配与 LPIPS 约束的可比对象是 [[entities/flux-3-multimodal-flow-model-black-forest-labs-2026|FLUX-3 多模态流模型]]，U1.5 的增量在「同一套视觉表征同时做理解与生成」。
- 统一多模态生成：与 [[entities/omnishow-unified-multimodal-video-generation-icml-2026|OmniShow 统一多模态视频生成]] 共享「统一表征」命题，差别在图像 4K 高分辨率与编辑侧多参考约束。
- 数据侧：图文交错轨迹（教程/生活/信息图/视频/推理混合）与 [[entities/qwen-image-agent-bridging-the-context-gap-in-real-world-image-generation|Qwen-Image-Agent]] 的上下文补全思路互补。

## 开放资源

- 技术报告：arXiv 2609.11929
- 模型集合：huggingface.co/collections/sensenova/sensenova-u15
- 代码：github.com/OpenSenseNova/SenseNova-U1
- NEO-unify 架构博客：huggingface.co/blog/sensenova/neo-unify
- Demo：unify.light-ai.top

→ [[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026|原文存档]]
