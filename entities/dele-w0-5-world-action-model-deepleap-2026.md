---
title: "DELE-w0.5 — 结果导向世界动作模型（移除视频生成，未来世界表征仅作训练监督）"
created: 2026-09-08
updated: 2026-09-08
type: entity
tags: [world-action-model, wam, embodied, robot, manipulation, flow-matching, world-model, vla, robotics, deepleap]
sources: [raw/articles/dele-w0-5-world-action-model-deepleap-2026]
confidence: 0.75
provenance_state: extracted
---

# DELE-w0.5 — 结果导向世界动作模型

## 概述

**DELE-w0.5** 是深度跃迁（DeepLeap）提出的具身**世界动作模型（World Action Model, WAM）**，核心主张是**机器人控制器无需逐帧重建未来视觉，只预测动作完成后的紧凑世界表征即可**——以「行动结果为中心」取代「视觉过程为中心」。其在 Astribot S1 双臂机器人的 640 次真机实验中取得 62.5% 完整任务成功率，约为最强基线（XR0, 30.0%）的 2.1 倍。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

## 核心问题：视频生成是控制负担

当前机器人基础模型大致分两条路线。**VLA（视觉-语言-动作）**直接由语言指令 + 多视角观测 + 本体状态输出动作，把环境变化隐含编码在参数里；**视频式 WAM** 同时预测动作与完整未来视觉轨迹，通过未来画面学习物体运动、空间变化与接触关系。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

DELE-w0.5 指出视频式 WAM 的问题：对机器人操作而言，世界模型的目标不是复现每一中间时刻的视觉，而是预测动作执行后达到的物理结果。多帧视频 latent 远大于动作序列，增加序列长度、显存与训练负担，推理时还需对高维视频+动作 latent 多轮去噪，控制循环受视频生成速度限制。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

由此三类模型形成不同预测目标：VLA 预测当前条件下的动作，视频式 WAM 联合预测动作 + 完整未来视觉轨迹，DELE-w0.5 **联合预测动作 + 动作结束后的未来世界表征**——模型仍学习未来，但未来信息更紧凑、更贴近控制目标。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

## 架构：双流 Transformer + 训练唯监督的未来世界表征

DELE-w0.5 采用**双流 Transformer**。条件流接收语言指令（Qwen3 编码）、机器人多视角图像与本体状态（视觉 DINO-v3 编码，含一头 + 两腕视角）；噪声流处理加入噪声的动作序列（一次预测 60 步动作，Astribot S1 每步 14 维，覆盖双臂位置/姿态/夹爪）+ 未来世界表征，通过 **Flow Matching** 同时学习动作去噪与未来状态去噪。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

**未来世界表征不是生成的图片**，而是 DINO-v3 对未来观察的编码（物体位置、机械臂状态、空间关系），模型只在 latent 空间学习这种状态变化。其 token 可读取语言、当前观察与动作，需回答「执行这组动作后环境进入什么状态」。它通过共享参数参与训练，动作生成路径不读取该表征，但仍受动作后果学习影响。**推理阶段未来世界表征 token 全部移除**，控制链路显著缩短，无需等待未来视频去噪。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

模型还设计了**非对称注意力**：动作 token 可读取语言、当前观察与动作序列内部信息，但**无法读取未来状态目标**，防止动作预测提前看到答案。DELE-w0.5 核心基座从底层自研，未二次封装任何开源机器人基础模型/VLA/视频生成权重。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

## 真机实验：640 次长程任务

在 Astribot S1 双臂机器人上设置四类长程任务（开门、冰箱取可乐、杯中加冰、爆米花进微波炉启动加热），对比 GWP0.5、XR0、π0.5、LingBot-VLA2、HY-VLA、Spirit-v1.5、GR00T-N1.7 七种基线，同数据同协议等量微调，每方法每任务 20 次正式测试，共 **640 次真机实验**。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

- **完整任务成功率**：DELE-w0.5 62.5%（80 次中成功 50），XR0 最强基线 30.0%，高出 32.5pp（≈2.1×）。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]
- **平均有序阶段进度**：DELE-w0.5 81.3%，最佳基线 GWP0.5 61.3%，领先 20.1pp（有序进度要求前置步骤完成才计入，避免一次成功抓取掩盖后续问题）。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]
- **分任务**：开门 80%、冰箱取可乐 65%、杯中加冰 45%、微波炉爆米花 60%，全部四任务最高。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

差距主要出现在**任务后半程**——长程操作难点常在技能连接处：抓住物体后需保持已有状态、切换执行手臂与交互对象、推进到最终条件。DELE-w0.5 更频繁到达任务后半程，并将阶段性进展转化为完整成功。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

此外 DELE-w0.5 具备 **zero-shot 跨背景泛化**：在桌面材质/空间布局/光照/视觉背景显著变化的场景（杂乱工位到会议室）仍能完成微波炉与杯中加冰任务，不依赖固定场景也不记忆特定背景。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

## 性能与效率

DELE-w0.5 在 RTX 4090 上推理延迟中位数为 **87.5ms**，快于 GWP0.5、π0.5、LingBot-VLA2、HY-VLA，与 Spirit-v1.5 接近，同时取得更高完整任务成功率。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

## 与前序 WAM 的辨析

DELE-w0.5 与已收录的 [[entities/4d-wam-world-action-model-3d-trajectory-alignment-2026|4D-WAM]] 属同一 WAM 家族但角度互补：4D-WAM 聚焦**训练阶段加 3D 轨迹场辅助监督以缓解 2D/3D 鸿沟**；DELE-w0.5 则主张**移除视频生成、以动作结束后的紧凑世界表征为训练目标本身**，两者都坚持「训练监督、推理零开销」。DELE-w0.5 也区别于 [[entities/noe-0-world-action-model-no-teleop-noematrix-2026|NOE-0]]（无本体数据 WAM）与主流 [[entities/lingbot-vla-2-60000h-open-source-vla|LingBot-VLA2]]（VLA 路线）。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

## 结论视角

DELE-w0.5 的意义不在于终结路线之争，而在于为世界模型提供新思路：最有效的未来信息未必是可观看的视频，而是不追求完整视觉复现、却足以融合真实物理空间并约束行动的世界表征。对具身智能设计，它示范了「以行动结果为中心」的控制器建模路径。^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]

→ [[raw/articles/dele-w0-5-world-action-model-deepleap-2026|原文存档]] ^[raw/articles/dele-w0-5-world-action-model-deepleap-2026.md]