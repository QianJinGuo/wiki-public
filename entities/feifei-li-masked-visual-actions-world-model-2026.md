---
title: "掩码视觉动作（Masked Visual Actions）——李飞飞团队世界模型"
created: 2026-07-23
updated: 2026-11-24
type: entity
tags: [world-model, robotics, video-model, stanford, fei-fei-li, vision, embodiment, mask]
sources: [raw/articles/feifei-li-masked-visual-actions-world-model-2026]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 掩码视觉动作（Masked Visual Actions）——李飞飞团队世界模型

> **Background**：本文基于 PaperWeekly 对斯坦福李飞飞团队 Masked Visual Actions 论文的解读整理。论文由斯坦福大学联合马里兰大学、哈佛大学研究者共同完成，提出将机器人动作表示为视频中的像素级运动条件，用预训练视频模型统一世界预测与行为生成。^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

## 核心思路

传统机器人世界模型通常接收关节角、末端位姿或控制向量，这些信号与具体机械结构绑定。Masked Visual Actions（掩码视觉动作）将机器人动作表示为**视频中随时间逐帧变化的时空像素掩码**——保留实体在整段视频中的时空像素区域，其余区域统一置为灰色背景。外形、位置、尺度和接触过程都直接可见。^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

## 技术方案

- **底座模型**：Wan-Fun-Control 2.2 14B，采用秩为 256 的 LoRA 适配
- **训练数据**：约 1000 条 DROID 演示 + 4000 个 RoboCasa 样本，总时长约 **15 小时**机器人视频，同时纳入成功与失败轨迹
- **训练规模**：约 10000 步，使用 8 张 H200，耗时约 4 天^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

### 双重用法

在同一掩码表示下，模型始终在补全未显示的实体轨迹：
1. **前向预测**：机器人轨迹作为输入 → 预测物体和环境的变化
2. **逆向生成**：目标物体轨迹作为输入 → 补全相应的机器人行为

前向预测和逆向生成由**同一个模型**完成，无需分设两个网络。^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

### 跨具身泛化

在训练中未出现的夹爪和双臂机器人上，这套视觉接口的跨具身泛化表现出色。视觉表示天然不绑定特定机械结构，可迁移到不同形态的机器人平台。^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md]

## 与世界模型的关系

该方法可归类为[[concepts/ai-coding-agent-from-helloworld-to-production|基于视频的世界模型]]的新范式——不是用关节角或控制向量表示状态，而是直接用像素级时空掩码统一世界预测和行为生成。与 [[entities/baai-orca-next-state-prediction-world-model|BAAI ORCA 状态预测世界模型]]、[[entities/light-interaction-world-model-inference|轻交互世界模型推理]]、[[entities/loopwm-looped-world-models|循环世界模型]]等方案相比，其核心差异在于**动作表示由数值向量变为视觉掩码**，大幅降低了对具体机械结构的依赖。

## 深度分析

### 动作即像素：为什么这条路线成立

预训练视频模型已在海量视频中积累物体运动、接触与交互先验，瓶颈只在"动作如何喂进来"。关节角与控制向量同具体机械结构绑定，即使把骨架和末端位姿可视化进图像，也难以保留完整外形、接触关系与遮挡信息^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:89]。掩码视觉动作保留实体的时空像素区域、其余统一置灰，外形、位置、尺度和接触过程直接可见^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:94]。关键自由度在于"保留哪类实体轨迹，就决定了模型以什么为条件、补全什么"，由此同一模型可充当世界预测器、策略评估器、规划器与动作提取器^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:134-154]。这与 [[entities/rhoda-scaling-web-video-pretraining-robots|网络视频预训练机器人]] 一脉押注相同：视频模型积累的运动知识可少量数据迁移到机器人任务^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:314]。

### 一体两面：前向预测与逆向生成的统一

以往预测动作后果与根据目标生成机器人行为通常需要分别建模；此处两者都被转化为条件视频补全——机器人轨迹输入则预测物体和环境变化，物体轨迹输入则补全机器人行为，前向预测与逆向生成由同一个模型完成^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:141-165]。模型并未专门接受物体轨迹条件训练，却能零样本补全机器人行为；项目主页还展示了从人类示范中提取物体的掩码视觉动作、再生成匹配机器人运动的输入来源^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:160]。注意生成结果仍是视频，真正执行还要经逆动力学模型转成低层控制指令^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:170]。

### 跨具身泛化的量化证据

在 DROID 训练分布内，掩码视觉动作与骨架、末端执行器位姿条件表现相近^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:190]，差距出现在未见具身上：换到训练未见的真实夹爪，掩码条件 PSNR 22.79 领先骨架条件的 21.02 与位姿条件的 20.32；在 BEHAVIOR 双臂机器人上为 22.90 对 19.58 与 19.23^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:207-212]。反衬证据是：使用稀疏条件时，未见夹爪上的生成更容易改变机器人形态、甚至凭空增加机械臂^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:195]。这印证了 [[entities/feifei-li-tract-visual-tracks-robot-world-model|视觉轨道世界模型]] 一脉的判断——统一的是世界模型接收动作的视觉表示，不要求不同机器人共享动作空间或低层控制接口^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:185]。多种机器人任务由此可围绕同一视觉接口展开，而非为每种形态重新定义世界模型的输入^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:309]。

### 训练配方与失败边界

配方是 LoRA（秩 256）适配 Wan-Fun-Control 2.2 14B，约 10000 步、8 张 H200、约 4 天；数据仅约 15 小时机器人视频（1000 条 DROID 演示 + 4000 个 RoboCasa 样本），且同时纳入成功与失败轨迹以学习不同动作的结果；机器人区域既可从真实视频中分割，也可依据状态、URDF 和相机参数渲染^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:108-129]。失败边界同样清晰：精细接触仍可能失真，生成中会出现虚假接触、物体瞬移或夹爪离开后物体继续运动；参考帧完全不可见的区域可能出现伪影；多候选生成成本较高；基于 URDF 渲染动作条件还需机器人模型与相机标定^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:287-292]。

## 实践启示

1. **小数据适配是现实的**：约 15 小时机器人视频、8 张 H200、4 天即可让 14B 视频模型学会新动作条件——LoRA 而非全参微调是这个预算的核心假设；复现时可先在 RoboCasa 仿真打通全流程，再迁移真机。
2. **失败轨迹必须入训**：数据同时纳入成功与失败轨迹，世界模型的价值在于预测"坏动作"的后果；只喂成功演示的世界模型学不会拒绝危险方案。
3. **Best-of-N 规划是低垂果实**：Diffusion Policy 采样多条候选、世界模型逐条生成未来视频、Gemini 3.1 Pro 按任务完成度/接触真实性/物理合理性评分选优，六项 RoboCasa 任务提升 7 至 26 个百分点，且无需改动策略本身^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:222-227]。
4. **视频评估只可作相对比较**：RoboCasa 中视频预测与真实成功率相关系数达 0.982，适合比较不同策略的相对表现；但真实机器人上视频评估结果整体偏高，不能当绝对指标用（真实实验覆盖橙子入袋、积木堆叠、毛巾入碗和关闭抽屉，每项 20 条演示^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:241-252]）。
5. **视频不是动作**：生成视频到可执行控制之间隔着逆动力学模型；在逆动力学模型与三个策略基线各用 100 条示范训练的对照下，CoffeeServeMug 上该路线 20 次测试成功率达 90%，高于 Diffusion Policy 的 50%、ACT 的 80% 与 SmolVLA 的 85%^[raw/articles/feifei-li-masked-visual-actions-world-model-2026.md:268-273]。
6. **上生产前先算伪影账**：虚假接触、物体瞬移、不可见区域伪影与多候选生成成本是当前落地的主要风险；选择渲染式条件还要预留 URDF 与相机标定基础设施。

## 论文资源

- **论文**：https://arxiv.org/abs/2607.19343
- **GitHub**：https://github.com/HadiZayer/masked-visual-actions
- **项目主页**：https://masked-visual-actions.github.io/

→ [[raw/articles/feifei-li-masked-visual-actions-world-model-2026|原文存档]]
