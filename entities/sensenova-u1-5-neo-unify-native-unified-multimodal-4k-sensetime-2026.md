---
title: "SenseNova-U1.5：商汤 NEO-unify 原生统一多模态训练法（4K 生成 + 多专家 OPD）"
created: 2026-09-18
updated: 2026-11-15
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

## 深度分析

### 为什么「先专业化、再统一」比直接统一训练更有效

统一中期训练用 0.1 : 1.0 的理解/生成损失权重把优化力度压向生成侧，但这只解决了「谁优先」的问题，没有解决「目标互相打架」的问题：追求画面美感会伤文字，编辑动作过强会破坏原图，信息图同时要文字准确、布局组织与整体质量。这些冲突发生在梯度层面，靠加权无法拆开。U1.5 的做法是把冲突各方物理隔离——四个专家各自用互不干扰的奖励信号充分优化（美学专家穿插文字任务防止美学 RL 伤可读性；OCR 专家用 PaddleOCR 校验加 GRPO-Guard 稳定稀疏奖励；信息图专家先强化文字再 DPO 改整体视觉；编辑专家用「最差维度决定奖励」的 min-reward 设计），训练阶段允许能力专业化，推理阶段再通过在线策略蒸馏汇回一个统一模型。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:125-141]^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:153] 这实质上是把多目标优化从「一个模型的权重空间里做权衡」改写为「多个模型的输出空间里做合并」，绕开了统一目标下单项能力被稀释的老问题——这正是 U 系列从 U1（验证统一可行）到 U1.5（回答统一之后如何扩展）的关键一跃。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:63]

### 空间联合重建：4K 的瓶颈不在 token 粒度而在解码端

U1 已经证明 32×32 像素一个 token 的极紧凑表示可以压低序列长度，但每个 token 在输出端独立还原自己的图像块，块与块之间没有最后一步信息交换。这个缺陷在低分辨率下被隐藏，分辨率一高，接缝、纹理不连续与局部几何偏差被同步放大——瓶颈不在表示压缩率，而在像素恢复路径缺少邻域通信。U1.5 的修复方式相当克制：保留 32×32 token 粒度不动，只重做解码端——先还原成二维特征场，用 3×3 卷积让相邻区域交换信息，再经多级 Pixel Shuffle 恢复 RGB，让颜色、纹理、几何在像素确定之前完成联合建模。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:71-81] 配套地，分辨率本身被纳入去噪条件（噪声条件扩展到 4096×4096），模型同时感知去噪阶段与目标分辨率对应的噪声统计。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:81] 对同样受「紧凑表示 vs 高分辨率」张力约束的架构，这是一个可复用的诊断思路：先定位空间通信断裂发生在编码还是解码，再决定动哪一端。

### 多专家 OPD：把蒸馏监督钉在学生自己的分布上

传统离线蒸馏的根本缺陷是教师轨迹与学生实际生成分布之间的偏差——教师从自己的状态出发给出的「正确答案」，学生可能根本到不了那个状态。U1.5 的多专家在线策略蒸馏（OPD）针对性地改了这一点：学生先沿自己的生成分布产生轨迹，再按任务路由到对应冻结专家，在同一学生状态、同一时间步、同一条件下比较两者的速度场，蒸馏监督因此直接落在学生实际会到达的状态上。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:145-149] 另一个值得注意的调度设计是监督从高噪声状态逐渐移向低噪声：前期盯整体结构与构图，后期才收紧到文字、纹理与细节——这与编辑专家把优化窗口从早期去噪移向后期的做法同构，本质上都是「先定语义结构、再修局部细节」的课程顺序。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:143]^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:151] 代价在于工程复杂度：四个专家各需一套奖励管线与独立训练，且路由正确性直接决定监督质量。

### 生成与理解的双向迁移：CoT 的收益边界在哪里

U1.5 的实验给出了双向迁移的两个方向的实证。正向（理解→生成）：生成数据并未大量依赖结构化模板，但模型面对长指令、复杂组合与高度结构化视觉要求仍泛化良好，研究团队据此判断理解与推理中形成的结构知识和规划能力迁移进了视觉创作。反向（生成→理解）：VBVR-Pro 图文交错 68.2（Nano-Banana-Pro 56.4、GPT-Image-2 50.7）、RealUnify-GEU 56.3（上一代 47.5），生成开始作为中间表征参与后续理解与推理。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:189-201] 但 CoT 的收益有清晰边界：WISE 0.70→0.81、RISEBench 总分 33.6→38.6，时间/因果/逻辑类编辑收益明显，而空间类编辑从 49.0 反降至 42.0——对能直接依据图像空间关系完成的编辑，显式思维链是负资产。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:187] 这提示推理开销应按任务类型门控，而不是全局套用。

## 实践启示

1. 扩高分辨率先查解码端：紧凑视觉 token 的分辨率瓶颈往往不在编码压缩率，而在像素恢复缺少邻域通信。3×3 卷积 + 多级 Pixel Shuffle 的空间联合重建是低成本修复路径，比牺牲 token 粒度更划算。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:71-81]
2. 多目标冲突调不动权重时，改拆专家再合并：specialize-then-unify 把权衡从梯度层面挪到输出层面，四专家 OPD 汇回是可直接套用的模板。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:125-153]
3. 蒸馏务必 on-policy：让监督落在学生实际生成的状态上（同状态、同时间步、同条件比速度场），否则教师轨迹与学生分布的偏差会吃掉蒸馏收益。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:145-149]
4. 编辑奖励用 min-reward 设计：最终奖励由表现最差的维度决定，未编辑区域一旦被破坏就直接压低总分——「某维度再好也抵消不了另一维度的严重失误」，这比加权平均更能约束破坏性行为。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:139-141]
5. CoT 按任务类型门控：时间/因果/逻辑类编辑加显式推理有正收益，空间类编辑直接生成更好；全局无差别套 CoT 会付出负收益与额外推理成本。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:183-188]
6. 4K 需要数据与条件双前提：88.2% 训练量超 1024²、64.4% 超 2048² 的高分辨率数据供给，加上分辨率感知的噪声条件（至 4096×4096），二者缺一——只堆数据不给条件信号，或只调条件不改数据分布，都推不到原生 4K。^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:81]^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:97]^[raw/articles/sensenova-u1-5-neo-unify-native-unified-multimodal-4k-sensetime-2026.md:103]

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
