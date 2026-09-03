---
title: "Meta MSL（Multi-Scale Latent）：余家辉团队连发图像视频模型"
created: 2026-07-08
updated: 2026-07-08
type: entity
tags: [meta, image-generation, video-generation, multimodal, msl]
confidence: 0.6
provenance_state: extracted
sources: [raw/articles/meta-msl-image-video-models-余家辉]
---

# Meta MSL：图像视频统一建模新范式

Meta 基础 AI 研究团队（FAIR）在余家辉带领下连发图像与视频生成模型，基于 Multi-Scale Latent（MSL）架构实现图像和视频的统一建模。MSL 通过在多个尺度上学习潜在表示，同时捕捉空间细节与时间动态，在同一框架内支持图像生成和视频生成任务。^[raw/articles/meta-msl-image-video-models-余家辉.md]

## 核心产品

### Muse Image：Agentic 图像生成

Muse Image 是 Meta 推出的图像生成模型，走的是与常规文生图工具不同的路线——不是简单的文本到像素转换，而是像 AI 智能体一样工作。收到用户需求后，它不会急着直接出图，而是先拆解梳理完整创作思路，碰到需要真实信息的内容会主动调用配套工具辅助^[raw/articles/meta-msl-image-video-models-余家辉.md]

**技术特点：**

- **工具调用能力**：Muse Image 在训练过程中学会了通过编写代码来生成精准的图表和二维码，还可以将生成的图像与代码结合，制作动图、嵌入图像的网页甚至可运行的互动小游戏。例如，它能根据用户的宠物照片编写出一个完整的 HTML 和 JS 互动游戏^[raw/articles/meta-msl-image-video-models-余家辉.md]
- **网络搜索**：通过搜索网页获取实时信息和视觉参考，在处理涉及新闻事件或现实常识的提示词时大幅提升画面准确度^[raw/articles/meta-msl-image-video-models-余家辉.md]
- **自主修正**：通过强化学习训练，当模型在思考链中发现画面细节有偏差时，会主动进行局部修改；如果发现方向完全错了，则会重新生成或调用工具来辅助。这种自我修正行为并非人工设定，而是模型在追求更高生成质量过程中自主学习到的成果^[raw/articles/meta-msl-image-video-models-余家辉.md]
- **推理时间扩展**：与大语言模型类似，Muse Image 支持推理阶段的计算扩展。给模型更多思考时间，它能进行更多推理步骤、调用更多工具并进行多次自我修正，从而产出质量更高的图像。实验表明推理投入与图像质量之间呈近似对数线性扩展关系^[raw/articles/meta-msl-image-video-models-余家辉.md]

**产品整合**：

Muse Image 与 Meta 生态深度打通：支持多参考图合成（用户可在提示词中同时输入文字和多张参考图片，把特定的人物、衣服、背景等揉合到同一张画作中）；支持多轮对话编辑（用户可连续提出修改意见，如先改风格、再保留特定元素、最后输出对比图）；支持 @Instagram 好友（Muse Image 可拉取对方公开发布的照片来生成相关内容）。所有 AI 生成的图片都带 Content Seal 隐形水印，裁剪、压缩、截图均无法去除^[raw/articles/meta-msl-image-video-models-余家辉.md]

### Muse Video：同步预览

Muse Video 与 Muse Image 同底座训练，具备高视觉保真度、原生支持音频、提示词理解良好。目前 Arena 文生视频排行榜暂列第三（排在谷歌 Gemini Omni Flash、字节 Seedance 2.0 之后）。需要改进的方面包括音画同步和高速运动场景的物理准确性，将在未来几个月内持续优化^[raw/articles/meta-msl-image-video-models-余家辉.md]

## 深度分析

### 1. Multi-Scale Latent（MSL）架构的统一建模价值

MSL 架构的核心创新在于图像和视频共享同一骨干网络，仅在输入/输出层有差异。这种设计大幅简化了训练和部署流程，使得同一个预训练模型可以同时支持生成和视频理解任务。与 [[entities/diffusiongemma-4x-faster-text-generation-google-2026-06|DiffusionGemma]] 等纯自回归扩散模型不同，MSL 在多尺度潜在空间上学习表示，既能捕捉图像的精细局部细节，又能理解视频帧之间的时间动态^[raw/articles/meta-msl-image-video-models-余家辉.md]

### 2. Agentic 生成范式的转变

Muse Image 最引人注目的设计是「Agentic Image Generation」——将 LLM 作为图像生成的核心控制器。这与 [[entities/meta-agent-image-generation-model|Meta 的 Agent 生图模型]] 方向一致，代表了图像生成从「模型中心」走向「Agent 中心」的范式转变。LLM 不再只是文本理解模块，而是生成流程的总指挥：

- 传统扩散模型：文本编码 → 扩散采样 → 图像输出（黑盒过程）
- Agentic 范式：LLM 规划 → 工具调用 → 迭代修正 → 最终输出（可解释的推理驱动过程）^[raw/articles/meta-msl-image-video-models-余家辉.md]

### 3. 社交图谱与图像生成的融合

Muse Image 允许用户在提示词中 @Instagram 好友，拉取其公开照片进行参考生成。Meta 将这一能力定义为「Native Social Context」——把社交图谱长进了图像模型里。这既是产品创新也是隐私挑战：Meta 的方案是允许用户在设置中选择 opt-out 禁止他人使用自己的公开照片做 AI 二创，同时所有生成内容都带 Content Seal 水印^[raw/articles/meta-msl-image-video-models-余家辉.md]

### 4. 团队的学术与产业背景

MSL 视觉团队的核心成员构成反映了当前 AI 研究的高端人才流动趋势。首席科学家赵晟佳（清华本科、斯坦福博士）2022 年毕业后加入 OpenAI，全程参与从初代 ChatGPT 到 o3 的预训练，2025 年 6 月加入 Meta。多模态负责人余家辉（中科大少年班、UIUC 博士）在谷歌时担任 Gemini 多模态视觉联合负责人，2023 年 10 月加入 OpenAI 担任感知团队负责人，参与 GPT-4o 到 o4-mini 的研发，2025 年 6 月与赵晟佳一起加入 Meta^[raw/articles/meta-msl-image-video-models-余家辉.md]

这种从「OpenAI → 一线大厂」的人才回流，与 [[entities/agent-harness-dingtalk-recruitment|Agent Harness 招聘实践]] 中人才流动的宏观趋势相呼应。

## 实践启示

1. **推理时间扩展是生成质量的「杠杆」**：Muse Image 证明给生成模型更多「思考时间」可以直接提升输出质量。在构建图像生成应用时，应考虑在推理阶段分配可调节的计算预算，让用户根据需要权衡速度与质量。

2. **Agent 范式刷新了 AI 工具的交互设计**：Muse Image 的自主修正、工具调用、链式推理等能力，使得用户可以用自然语言进行复杂的多步创作。这种交互范式正在从代码生成领域扩展到视觉生成领域。

3. **社交数据与隐私的平衡设计**：Muse Image 的 Instagram 集成展示了 AI 产品中社交数据利用的隐私设计模式——opt-out 机制 + 隐形水印 + 公开数据只读引用。这是构建 AI 社交产品时的参考范式。

4. **多参考图合成的工作流价值**：在电商、设计、品牌营销等场景中，Muse Image 的多参考图合成为一个提示词即可完成复杂视觉创作的能力，有望大幅降低图像素材生产的边际成本。

→ [[raw/articles/meta-msl-image-video-models-余家辉|原文存档]]
