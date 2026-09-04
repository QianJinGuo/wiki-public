---
title: 世界模型（World Models）
created: 2026-09-05
updated: 2026-09-05
type: concept
tags: [world-model, embodied-ai, video-generation, generative-models, robotics]
confidence: 0.75
provenance_state: merged
---

# 世界模型（World Models）

> 库内 51 个实体（2026-09-05 概念缺位检测）共享的核心主题，此前没有 concept 页收拢。建立于 [[drafts/wiki-emergent-viewpoints-2026-09-concept-gap|2026-09 概念缺位透镜轮]]。

## 一词两义：库内的世界模型分两个阵营

**阵营 A —— 学习型模拟器**：世界模型是给机器人/自动驾驶提供可训练虚拟环境的**生成式基础设施**。代表是 NVIDIA Cosmos 系：[[entities/nvidia-cosmos-fine-tuning-robot-video-generation|Cosmos Predict 2.5 可用 LoRA/DoRA 针对机器人视频微调]]^[raw/articles/fine-tuning-nvidia-cosmos-predict-25-with-loradora-for-robot-video-generation.md]，把通用物理视频先验适配到特定本体；[[entities/wall-oss-05-pretraining-embodied-ai-x-square-robot|Wall-OSS-0.5 则把预训练范式带到具身大模型本体]]——零样本上真机。这一阵营的验收标准是**下游任务迁移**，不是画面好看。

**阵营 B —— 实时交互式生成环境**：世界模型本身成为可交互产品。[[entities/世界模型的deepseek时刻魔芯flash-world-model降本70跑出50fps实时交互|魔芯 Flash World Model]] 在 NPU 上跑出 50fps 实时交互、成本降 70%，被称为"世界模型的 DeepSeek 时刻"^[raw/articles/moworld-flash-world-model-50fps-npu.md]；[[entities/amap-abot-earth-0.5-3d-native-world-model|高德 ABot-Earth 0.5]] 做 3D 原生城市世界模型，宣称以 1% 成本覆盖地理级场景。这一阵营的验收标准是**交互延迟与一致性**。

两个阵营共享技术底座（视频扩散/自回归 transformer + 动作条件化），但优化目标相反：A 要"物理正确到能训策略"，B 要"反应快到能实时玩"。库内页面常混用"世界模型"一词而不区分，是检索噪音的主要来源。

## 与 JEPA 的路线分歧

[[entities/yann-lecun-jepa-world-model|LeCun 的 JEPA/AMI Labs 路线]]在库内是阵营外最大的单一声音：拒绝生成式像素预测，主张在抽象表征空间做预测——认为像素级生成把容量浪费在不可预测的细节上。这构成一个尚未在库内对账的张力：**2026 年的落地证据（Cosmos/Flash 全是生成式）暂时站在 JEPA 的对立面**，但 JEPA 阵营会回答"它们生成的是视频，不是可规划的世界状态"。相关分歧的机制层讨论见 [[concepts/eval-optimizer-firewall|评测防火墙]]（训练环境与评测环境的隔离问题在(world-model-as-env)场景下同样成立）。

## 具身连接

世界模型是 [[concepts/embodied-intelligence-frontier|具身智能]] 的数据引擎：真机数据贵且慢，世界模型提供可并行的合成经验。库内 [[entities/powering-the-future-of-robotics-in-europe-deepmind-2026|DeepMind Robotics Accelerator]]、各 VLA 本体页（见 `embodied-ai` 标签簇）构成需求侧；Cosmos/Wall-OSS/Flash 构成供给侧。**"世界模型 → sim-to-real → VLA 训练"这条管线在库内散落成十几页，本页是它们的第一个收拢点。**

## 检索入口

- 高分簇页：[[entities/fine-tuning-cosmos|Fine-Tuning Cosmos]] · [[entities/世界模型的deepseek时刻魔芯flash-world-model降本70跑出50fps实时交互|魔芯 Flash]] · [[entities/amap-abot-earth-0.5-3d-native-world-model|ABot-Earth]] · [[entities/yann-lecun-jepa-world-model|JEPA/AMI]]
- 相邻概念：[[concepts/embodied-intelligence-frontier|具身智能前沿]] · [[concepts/video-generation|视频生成]]（待建）· [[concepts/diffusion-models|扩散模型]]（待建）
- 检测数据：[[queries/vault-evolution-dashboard|进化仪表板]] · `metrics/concept-gaps.json`
