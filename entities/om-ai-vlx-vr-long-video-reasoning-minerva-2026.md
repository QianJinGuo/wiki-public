---
title: "Om AI VLX-VR：长视频推理 VLM（MINERVA 登顶）"
created: 2026-09-07
updated: 2026-09-07
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

## 与 VLX 系列关系

[[entities/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026|VLX-Flow（持续感知）]]、[[entities/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026|VLX-Seek（细粒度感知）]]、[[entities/om-ai-vlx-go-vlm-navigation-0.6b-2026|VLX-Go（导航决策）]]共同构成 VLX 端侧流式多模态模型系列；VLX-VR 是其中面向动态长视频推理能力的新增变体。^[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026.md]

→ [[raw/articles/om-ai-vlx-vr-long-video-reasoning-minerva-2026|原文存档]]