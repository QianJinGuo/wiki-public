---
title: "MiDashengLM-Spatial — 小米开源的空间音频理解大模型"
created: 2026-10-25
updated: 2026-10-25
type: entity
tags: [multimodal, audio, spatial-intelligence, xiaomi, open-source, llm]
sources: [raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026]
confidence: 0.7
---

# MiDashengLM-Spatial — 小米开源的空间音频理解大模型

> **Background**：2026 年 10 月 15 日，小米以 Apache 2.0 开源 MiDashengLM-Spatial——以 MiDashengLM-7B 为基座、给音频大模型补上「一双耳朵」的端到端空间音频理解模型。它要回答的不只是「什么声音」，还有「声音在哪、怎么动、与我是怎样的空间关系」。

## 空间听觉：从「什么声音」到「声音在哪」

嘈杂街道上有人喊自己的名字，不必回头就能判断方向与远近。听觉本就是持续的三维定位：耳廓、头部与躯干构成不对称的接收系统，声波抵达两耳时已被物理环境改写。内容是「什么声音」，空间是「来自哪里」。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

主流多模态大模型恰在这一层残缺：多声道音频被简单平均、压成单声道再编码，IPD 与 ILD 在预处理阶段即被抹平，只剩混合响度包络——听得出「有狗叫」，答不出「狗在左前方」。所谓「一只耳朵」不是麦克风数量，而是信息通路。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

小米的动机在生态：空间智能是推理三维物理世界的能力，也是「人车家全生态」的支撑；设备分散各处，只知「有人说话」而不知「人在哪」，系统只能被动等指令。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

## 物理线索与机器约束（IPD/ILD、麦克风阵列）

相位差（IPD）是同一波前抵达两耳的时间差，低频段可靠；响度差（ILD）来自头部遮挡，高频段明显。此外头部、肩部与耳廓的反射衍射会在频谱上留下与入射角相关的起伏，人脑正靠这套「频谱指纹」分辨前后与仰角。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

机器侧条件不同：人的双耳间距由头骨固定，设备却受麦克风数量、布局与形态制约，耳机两麦、音箱环形阵列、车舱分布式麦克风对同一位置给出完全不同的通道信号。空间听觉因此难以靠「统一格式 + 扩规模」推进：不存在跨设备通用的输入分布，多声道数据又远比单声道稀缺。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

## MiDashengLM-Spatial 的定位与开源

解法是双分支编码器：语义分支继承 MiDashengLM 原有编码器（Dasheng-Huge，0.6B），空间分支用轻量化的 Dasheng-Base（86M）。结构同源、参数差一个数量级——空间能力不缺容量，缺的是正确输入与连接方式。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

连接采用跨层语义到空间特征注入，语义分支各层特征逐层注入空间分支并融合，方向单向。空间分支借到语义先验，能分辨「谁是说话人、谁是背景声」，从而做感知、定位与分离；语义分支信息流不动，通用能力被结构性地保护。用约 1 万小时双耳数据，在超过 100 万小时音频的基座上长出新增能力，靠的是旁路扩展而非重训主干。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

数据几乎全部合成：以头相关传递函数（HRTF）与虚源法（ISM）模拟室内声学，合成约 1 分钟的双耳场景，含多个静态或动态声源；语义继承原单声道标注，位置、轨迹与时序由参数化过程精确记录，再由 LLM 生成场景级空间描述，这正是普通音频 caption 给不出的监督信号。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

评测方法学最值得留意：把音频换成高斯噪声甚至屏蔽，模型仍可能凭语言先验得分，单声道模型在空间任务上也明显高于随机猜测。为此引入声道轮换——交换左右后重问，真在用 IPD/ILD 的模型应把左侧声源判为右侧；再统计前后均答对的「全正确率」，把「答对」与「用对了线索」分开。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

结果空间侧领先、通用侧不降：空间编码器在真实场景定位数据集上超过多种专业定位模型，音频问答与识别与基座持平。权重与推理代码以 Apache 2.0 公开，并附技术报告与评估代码——对需硬件验证的能力，开放评估代码不亚于开放权重。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

## 落地场景与生态（30+ 场景、人车家全生态）

「鸡尾酒会」最有说服力：多人同时说话时，「谁在左、谁在右」成为物理先验，辅助分离与跟踪，提升说话人日志与复杂环境识别率，且不需新硬件——两个以上麦克风即可。家庭场景更进一步：结合方向与距离判断用户位置，调度附近终端就近响应，小爱同学从被动接令走向主动感知。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

车舱里摄像头有视场角，A 柱与后视镜是盲区，而鸣笛、警笛、后方来车本就以声音出现，用方向补盲区是低成本的冗余感知。机器人场景更彻底：具身智能长期偏重视觉，空间听觉补上的是「听见身后有人靠近」。这些并非从零起跑——Dasheng 与 MiDashengLM-7B 已在智能家居、汽车座舱等实现超过 30 项落地。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

## 局限与开放问题

局限首先来自合成数据：HRTF 与 ISM 依赖房间几何、吸声与头模假设，合成混响同真实世界有差距，约 1 万小时也难覆盖真实设备的声学条件，模型可能学到「合成器特有的声学指纹」；跨房间与跨设备的泛化结论尚未给出。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

其次形态错位：模型吃双耳音频，真实设备多为几何各异的多麦阵列，迁移需额外通道映射；整体仍是双分支编码器加 7B 语言模型，端侧实时性未披露。评测上，「全正确率」只覆盖左右判断，连续方位角精度、距离误差与多人重叠的分离上限不在射程内。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

更深一层是表征统一：空间听觉输出以观察者为中心的局部方位，[[concepts/world-models]] 追求环境级、可规划的三维表征，不共享坐标系就难以进入决策回路。[[concepts/tool-use-reasoning]] 关注把感知转为动作——「声音在右后方」要变成「转向并避开桌角」，中间需要可调用的空间查询接口。[[concepts/embodied-intelligence-frontier]] 则提示：补齐感知通道是具身的前提，但不等于空间智能成立。 ^[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026.md]

## 关联

- [[entities/xiaomi-dasheng-audio-foundation-model-2026]] — Dasheng 声音基座与 MiDashengLM 前史，语义分支与训练数据由此继承
- [[concepts/world-models]] — 环境级三维表征：局部坐标如何并入世界模型
- [[concepts/embodied-intelligence-frontier]] — 具身智能前沿：补齐音频模态的空间感知
- [[concepts/tool-use-reasoning]] — 把方位感知转为可执行动作的推理接口

→ [[raw/articles/xiaomi-midashenglm-spatial-audio-llm-2026|原文存档]]
