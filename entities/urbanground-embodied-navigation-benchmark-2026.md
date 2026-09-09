---
title: "UrbanGround — 真实三维香港城市沙盒的具身导航评测（长程导航记忆崩塌）"
created: 2026-09-08
updated: 2026-09-08
type: entity
tags: [benchmark, embodied, navigation, spatial-ai, spatial-intelligence, multimodal, agent-eval, world-model, vlm]
sources: [raw/articles/urbanground-embodied-navigation-benchmark-2026]
confidence: 0.72
provenance_state: extracted
---

# UrbanGround — 真实三维香港城市沙盒的具身导航评测

## 概述

**UrbanGround** 是上海交通大学、新加坡国立大学、美团、香港中文大学、上海大学与牛津大学联合推出的**具身城市导航评测基准**——用香港真实三维地理数据搭建一座可连续行走的城市沙盒，让大模型以第一人称「走完一条路」而非只答对眼前问题。其核心发现是导航长程失败并非不会认路，而是「**走两步，就忘了路**」：短程导航最高成功率 75.0%，路线拉长后 10 个模型全部跌到 0%–3.8%。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 设计：把评测搬进可重复的城市沙盒

过去的城市空间智能评测多停在单张街景或航拍图，或沿预设节点切换视角，模型并未真正连续行走。UrbanGround 由地理、仿真、智能体三层组成。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

- **地理层**：以香港地政总署公开的 3D Visualisation Map 与 3D Pedestrian Network 为底图，将地理坐标、三维网格、步行网络对齐到同一套沙盒坐标。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]
- **仿真层**：建筑/墙体/坡道/地形起伏影响行动，同一地点可切换昼夜与天气，道路可途中封闭，行人沿步行网络移动，支持反复重跑同一条路线只改天气、封路或人流。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]
- **智能体层**：模型只见当前第一人称画面 + 任务说明，需要时可打开地图，再选择移动/转向/跳跃/操作地图；地图标出当前位置与目标但**不替它规划路线**。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

最终整理出 **810 个经人工检查的任务实例**，覆盖香港多个城区，用同一套动作接口测试 **10 个多模态模型**。论文从空间定位、持续记忆、动态调整三方面测「模型的判断能否一直管用」。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 关键发现 1：空间定位——认得出地标，分不清方向

10 个模型的视觉识别准确率 75.0%–93.8%，主动探索 46.3%–82.5%，但**方向理解只有 23.3%–58.3%**。Gemini-3.1-Pro 视觉识别 82.5%、方向理解却仅 23.3%（甚至低于随机猜测）——能认出地标，却在视角转动后难以判断其在左还是右。还有模型答对线索却走出路网（如 GPT-5.5 正确找出「东亚银行」，探索时却横穿马路离开步行区）。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 关键发现 2：持续记忆——走两步，就忘了路

短程导航 GPT-5.5 与 Claude-Opus-4.6 成功率均 75.0%；长程导航 10 个模型全部跌到 **0%–3.8%**（GPT-5.5 75→0%，Claude-Opus-4.6 75→1.3%）。每次转弯/过街/开关地图模型都要重新判断自己在哪、目标在哪，一个小偏差一路累积难以纠正。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

更关键的是**轨迹而非起点决定失败**：长程任务结束时各模型仍有 52.5%–81.3% 的轨迹比起点更接近目标，但完整成功率接近零——它们找对了大方向、走对了一段，最终仍卡在半路。GPT-5.5 的长程轨迹中 57.5% 比起点更接近目标，34.6% 将距离缩短至少 20%，但任务结束平均剩余距离仍是初始距离的 98.9%，到达率 0%。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 关键发现 3：动态调整——城市已变，模型还在走旧路

在行动过程中加入封路与移动行人：封路任务各路网遵循率 93.0%–96.4%，但**封路遵守率仅 10.0%–46.7%**，最终到达率 0%–3.3%；移动行人任务 90.6%–98.5% 时间仍走在路网上，**行人碰撞率却高达 76.3%–90.0%**。模型大部分时间按路网走，却不擅长因前方封闭及时改道，也常避不开行人——「看见变化还不够，动作必须跟着变」。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 结论与价值

UrbanGround 的价值在于用**可重复、可归因**的城市沙盒暴露出导航模型的「前后接不上」问题：前一刻作出的判断管不到后面的动作，路线越长、途中变化越多，这种断裂越明显。对具身 Agent 进入真实城市前的评测，它提供一种低成本、可逐步定位「模型从哪一步走偏、为何没绕回来」的手段。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

论文/项目主页/代码/App 均已公开（arXiv 2608.27456），接入自己的 Agent 即可在真实城市结构中测试其导航能力。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

## 相关实体

- [[entities/om-ai-vlx-go-vlm-navigation-0.6b-2026|OM AI VLX-Go VLM 城市导航]]：前者是端上 VLM 导航能力的模型侧实现，UrbanGround 则提供评测这类导航能力的城市级基准。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0]]（跨本体多模态物理世界模型）与 [[entities/amap-abot-earth-0.5-3d-native-world-model|AMAP Abot-Earth 3D 原生世界模型]]：三维城市/物理世界建模侧，UrbanGround 关注这些世界表征在连续导航任务中的实际落地表现。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]
- [[entities/embodied-native-llm-embodied-intelligence-new-stage|具身原生 LLM]]：体现本轮具身智能从「看懂」到「走完一条路」的评测范式演进。^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]

→ [[raw/articles/urbanground-embodied-navigation-benchmark-2026|原文存档]] ^[raw/articles/urbanground-embodied-navigation-benchmark-2026.md]