---
title: "Om AI VLX-VR：长视频推理 VLM（MINERVA 登顶）"
created: 2026-09-07
updated: 2026-11-20
type: entity
tags: [vlm, multimodal, vision, om-ai, long-video, video-reasoning, minerva, model-architecture, edge-ai]
sources: [raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026]
confidence: 0.7
---

# Om AI VLX-VR：长视频推理 VLM（MINERVA 登顶）

Om AI 的 VLX-VR 是 VLX 端侧流式多模态模型系列的第 4 个变体，将视觉推理能力从持续感知（VLX-Flow）、精准定位（VLX-Seek）、行动决策（VLX-Go）扩展到**动态长视频的跨时段复杂推理**。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md]

## 核心贡献：打破「时长诅咒」

长视频理解的难点是注意力稀释——视频越长，进入上下文的信息越多，关键线索之间的距离越远，模型需要重新建立事件之间的关联。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md]

- 无需人工切片，原生支持小时级超长视频端到端推理，保留连续时间上下文。
- 在谷歌 DeepMind MINERVA 视频推理基准上以 78.8% 视频问答准确率登顶，超越 Gemini 3.5、GPT-4.1。
- 跨时长准确率方差 CDAV 仅 2.97（标准差 ~1.72 个百分点），长视频档不降反升：15 分钟以上 Gemini 2.5 Pro Thinking 降至 57.97%、GPT-4.1 降至 47.25%，VLX-VR 反而升到 80.92%。

## 推理过程可追踪

MINERVA 为每题提供人工标注推理轨迹（~92 词，99.6% 含时间戳，平均 4 个时间点）。VLX-VR 正确样本中，模型推理与人工参考轨迹一致性高达 96.2%——对机器人、智能终端、视频分析等真实场景的可追溯能力尤其重要。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md]

## 深度分析

### 评测范式转变：从「画面识别」到「时间线推理」

MINERVA 的设计意图值得注意：1000 多道题、200 多段视频、时长横跨 2 分钟到 1.5 小时以上，其问题普遍要求模型综合多个时间片段的信息才能作答——一个人物早期出现的动作，可能要到数十分钟后才成为答题的关键证据。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:58-66] 这与传统 VideoQA「答案就在当前画面附近」的范式有本质区别：评测对象不再是单帧感知质量，而是跨片段的事件关联与状态维护能力。因此 78.8% 的登顶成绩反映的不只是识别精度，而是模型内部是否构建了持续可用的时间线表征。对比之下，人类在该基准上达到 92.54%，而原论文最佳模型 Gemini 2.5 Pro Thinking 仅 66.20%，说明长视频推理仍是人与最强模型之间差距最大的能力域之一。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:86-88]

### 「时长诅咒」逆转的技术含义

长视频掉分的根源是注意力稀释：输入信息越多，关键线索在上下文中相距越远，模型必须自行判断「什么值得被记住」。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:78-84] 行业通行解法是先切片再分别处理，但这会割裂连续时间上下文，反而增加重建片段关联的负担。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:70-75] VLX-VR 选择原生端到端处理小时级输入，其分时长曲线印证了这条路线的有效性：5 分钟以下 76.70%，15 分钟以上反而升至 80.92%，而 Gemini 2.5 Pro Thinking 与 GPT-4.1 在同档分别跌至 57.97% 和 47.25%，领先幅度从约 7.8 个百分点扩大到接近 23 个百分点，跨时长方差 CDAV 仅 2.97。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:92-102] 长视频档准确率不降反升，暗示模型具备某种主动的证据筛选与记忆保持机制，而非被动压缩全部输入——这与 [[concepts/attention-mechanism]] 中长程依赖的固有难题直接相关，也把「性能-时长曲线」从行业默认的下行变成了可争夺的指标。

### 推理轨迹可核验：从黑盒答案到过程审计

MINERVA 为每题标注了人工推理轨迹（平均约 92 词、99.6% 含时间戳、平均 4 个时间点），VLX-VR 正确样本与人工轨迹的一致性高达 96.2%。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:118-132] 这组数字的深层意义在于把评测从「答案对不对」升级为「过程是否可信」：当模型判断偏差时，开发者可以区分错误来自时间定位、视觉理解还是推理链条，而不是面对一个无法解释的结果。对机器人与智能终端这类安全敏感场景，可追溯的推理过程是部署前置条件；这一思路与评测基准领域的过程审计趋势（如 [[concepts/agent-evaluation-benchmark-frameworks]] 所概括的框架化评测）相互呼应。

### 能力维度组合与端侧物理 AI 定位

VLX-VR 的四个能力维度——阅读理解 90.76%、情境感知 90.32%、时间推理 85.95%、数值推理 85.71%——共同构成「找到关键事件 → 连接时间线 → 完成判断」的完整推理链。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:134-149] 更能说明泛化性的是基准外的开放任务：篮球馆中识别反常动作（无预设目标，需先理解场景再主动发现异常）与棋局跨时间状态追踪（持续维护事件状态而非识别单瞬间），VLX-VR 均表现优于对比模型。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:153-171] 结合其技术谱系——2025 年 2 月开源 VLM-R1 首次将 R1 式强化学习推理引入视觉领域，VLX-VR 将其扩展到动态长视频^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:175-179]——可以看出 Om AI 的路线是把 [[concepts/reinforcement-fine-tuning-rft]] 驱动的推理能力与端侧流式架构结合，让机器人、智能眼镜等设备「看到的不再只是眼前一帧，而是一段能够被持续理解的世界」，这与 [[concepts/world-models]] 及小时级视频世界模型（如 [[entities/alaya-evoke-hour-level-video-world-model]]）的研究方向殊途同归。

## 实践启示

1. **选型长视频模型时，把「分时长准确率曲线」而非「总分」作为首要指标。** 总分领先可能掩盖长档位崩塌；关注 CDAV 或等价的跨时长方差，要求厂商提供 15 分钟以上档位的实测数据。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:92-102]
2. **架构评估优先考察是否原生端到端，而非「支持长输入」的营销口径。** 切片式处理会割裂时间上下文并引入额外的片段关联负担；小时级端到端推理且长档不降分，才是可信赖的信号。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:70-75]
3. **在生产系统中要求推理轨迹可回溯。** 部署视频推理能力时，确保输出附带时间戳级的证据链，使失败可归因于时间定位、视觉理解或推理环节；VLX-VR 96.2% 的轨迹一致率可作为选型基准线。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:118-132]
4. **用开放异常检测任务做验收测试，不只跑标准 VideoQA。** 像「找出视频中反常动作并给时间点」这类无预设目标的任务（篮球馆测试）能暴露模型是否真正理解场景，比固定答案集更能区分真实泛化能力。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:153-171]
5. **将「时间」视为视觉系统的一等公民维度。** 构建机器人、巡检等物理 AI 管线时，设计上应显式维护事件状态与跨时段关联（如棋局追踪任务所示），而不是逐帧独立处理。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:163-171]
6. **关注 RL 推理框架向视频域迁移的路线图。** VLM-R1 到 VLX-VR 的演进表明 R1 式强化学习推理可从静态图像扩展到长视频；在自研多模态系统时，可优先复用此类 RL 推理基础设施而非从零构建。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md:175-179]

## 与 VLX 系列关系

[[entities/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026|VLX-Flow（持续感知）]]、[[entities/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026|VLX-Seek（细粒度感知）]]、[[entities/om-ai-vlx-go-vlm-navigation-0.6b-2026|VLX-Go（导航决策）]]共同构成 VLX 端侧流式多模态模型系列；VLX-VR 是其中面向动态长视频推理能力的新增变体。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md]

→ [[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026|原文存档]]