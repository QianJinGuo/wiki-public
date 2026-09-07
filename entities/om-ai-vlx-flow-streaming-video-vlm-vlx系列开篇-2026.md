---
title: "Om AI VLX-Flow: 流式视频理解 VLM — VLX 系列开篇"
created: 2026-07-05
updated: 2026-07-28
type: entity
tags: [vlm, multimodal, vision, om-ai, streaming-video, video-understanding, edge-ai, model-architecture]
sources: [raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Om AI VLX-Flow: 流式视频理解 VLM — VLX 系列开篇

VLX-Flow 是 Om AI（联汇科技）VLX 端侧流式多模态模型系列的第一弹，定位为**流式理解层**，解决「模型如何在用户提问之前就开始观察、记忆并随时响应」的问题。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

## 核心问题：从离线视频到在线感知

传统视频理解依赖「离线模式」——视频已录好、截好、上传好后，模型再抽帧、编码、推理。但真实设备中的摄像头持续采集、屏幕不断变化、机器人第一视角持续运动，输入从「离线文件」变为「实时流」。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

现有 VLM 的两种路线各有取舍：
- **全帧输入**：保留更多信息，但计算量和延迟迅速上升
- **固定采样**：降低计算成本，但容易丢掉帧间的动作细节

VLX-Flow 的方案：**增量视觉上下文建模**，将连续视频拆成小片段，按时间顺序增量处理。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

## 架构核心：双层记忆 + Linear Attention

VLX-Flow 的语言模型包含 **Linear Attention** 模块，通过**可递推状态**保留历史信息，每次只做增量计算。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

双层记忆结构：
1. **视觉缓存**：保留最近时间窗口的帧细节（动作、位置、主体状态、短时变化）
2. **文本承接层**：保留连续描述、用户问题和回答，维持长程语义上下文

多轮交互时文本承接层在合并/裁剪/回放过程中同步模型内部缓存，避免文本历史与模型实际记忆状态错位。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

## Stream Memory 训练范式

VLX-Flow 的训练将「观察」和「回答」分开：^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]
- 短视频窗口（如 16 秒视频拆成 2 秒片段）生成流式 caption，训练模型将连续视觉信息写入可递推的记忆状态
- 在后续时间点提问，模型必须基于累积记忆回答，不能回头重看视频

## 深度分析

### 实时流式理解的工程技术挑战

真实设备中的视频并非「离线文件」形态。摄像头持续采集、无人机持续飞行、机器人第一视角不断运动——输入源是一个永不停止的流。VLX-Flow 要解决的不仅是「看得懂」，更是**如何在资源受限下维护持续更新的视觉状态**。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

对比全帧输入（保留信息但计算量爆炸）和固定采样（低成本但丢失帧间动作细节），VLX-Flow 的增量视觉上下文建模将视频拆成连续小片段，按时间顺序增量处理。旧信息被压缩成可递推状态，避免了历史上下文无限变长。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

### Linear Attention 与 TTFT 优势

在标准自注意力机制中，序列变长意味着 KV Cache 膨胀，显存和计算压力随视频时长线性上升。VLX-Flow 的 Linear Attention 模块通过可递推状态保留历史信息，每次仅做增量计算，使得首 token 生成时延（TTFT）在长序列输入下保持稳定。与 Full Attention（TTFT 持续上升）和 SlideWindow（周期重置产生波动）相比，VLX-Flow 在超过一定输入长度后展现出显著的延迟优势。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

这种稳定的低延迟对真实交互至关重要：摄像头不会每隔 5 秒才看一眼世界，机器人也不能只在用户提问时才观察环境。TTFT 直接影响用户随时提问后的等待体验，是流式 VLM 能否实际部署的工程瓶颈之一。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

### 双层记忆的协同设计

视觉缓存保留短时高频信息（动作、位置、主体状态、短时变化），文本承接层保留长程语义上下文（连续描述、用户问题和回答）。多轮交互时文本承接层在合并、裁剪或回放过程中同步模型内部缓存，避免文本历史与模型实际记忆状态错位。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

这种「短时精度 + 长程连贯」的双层设计，使得模型既能准确回答"刚才谁进来了"这种瞬时问题，也能维持对整段时间线事件关系的理解。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

### Stream Memory 训练的数据构造创新

VLX-Flow 的训练数据构造方式是其核心技术亮点：「观察」与「回答」阶段拆分——短视频窗口（如 16 秒拆分 2 秒片段）生成流式 caption，让模型学习将连续视觉信息写入可递推记忆状态；然后在后续时间点提问（如证据出现后 10 秒或 1 分钟），模型必须基于累积记忆回答，不能重看视频。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

这套范式将视频问答从「打包上传-离线推理」推进到「持续在场-随时响应」，是 VLX 系列区别于传统视频 VLM 的核心差异。此外，Stream Memory 的机制可扩展到事件触发式交互：人员进入、物体遗留、异常动作等可触发主动提醒，减少事后回看需求。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

### VLX 系列的全链路设计

VLX-Flow 是三层链路中的感知前置层：Flow（持续感知）→ Seek（精准定位）→ Go（行动决策）。这种设计意味着视频处理前移到本地，模型在运行中维护状态，语言生成只在交互或事件触发时介入。系统因此不必反复上传和重算完整历史，更适合端侧或边缘节点部署。^[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026.md]

## 实践启示

1. **在线感知与离线推理的范式差异不容忽视**：真实设备中的视频是持续的流，不是打包的文件。构建实时视频理解系统时，必须从架构层面支持增量输入和状态维护，而非试图将离线流程套用到在线场景。

2. **注意力机制的选型直接决定部署可行性**：对于端侧 VLM，Linear Attention 或类似的可递推注意力机制比标准自注意力更适合长视频场景。KV Cache 的管理策略（TTFT 稳定性）是比模型精度更前置的工程约束。

3. **双层记忆是资源与精度的平衡点**：视觉缓存保短时精度 + 文本承接层保长程连贯，这种分层设计避免了单层记忆在容量和精度之间的矛盾。构建类似系统时，应明确短期和长期记忆的不同职责。

4. **Stream Memory 训练范式可推广**：「观察」和「回答」阶段分离，要求模型在不能回看的情况下基于累积记忆回答，这种训练策略不仅适用于视频理解，也可推广到任何需要在持续输入中维持状态并随时响应的场景（如监控、直播、实时字幕）。

5. **事件触发式交互是值得关注的方向**：模型在持续感知过程中可以主动识别关键事件（人员进入、异常动作等）并生成提醒，将 VLM 从「被动问答」扩展到「主动监测」，扩大了应用边界。

## 同系列对比

- [[entities/om-ai-vlx-seek-vlm-3b-fine-grained-perception-2026|VLX-Seek]] — 细粒度视觉感知与定位
- [[entities/om-ai-vlx-go-vlm-navigation-0.6b-2026|VLX-Go]] — 具身导航与执行

→ [[raw/articles/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026|原文存档]]
