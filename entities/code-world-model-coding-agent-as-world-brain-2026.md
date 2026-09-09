---
title: "Code World Model：让代码接管世界演化（Coding Agent as World Brain）"
created: 2026-09-08
updated: 2026-09-08
type: entity
tags: [world-model, video-generation, coding-agent, agent-architecture, open-world, westlake]
sources: [raw/articles/code-world-model-coding-agent-as-world-brain-2026]
confidence: 0.72
provenance_state: extracted
---

# Code World Model：让代码接管世界演化（Coding Agent as World Brain）

> 西湖大学 AGI Lab 与南洋理工提出 Code World Model（arXiv:2608.25927），把"世界如何演化"与"世界如何被看见"拆成两个互补问题：Coding Agent（作为世界大脑）用可执行代码决定世界规则与状态如何持续演化，视频世界模型负责把演化后的状态渲染为高保真视觉观察，中间由可编译的 Proxy 提供逐帧空间与时间约束。^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]

## 核心问题

现有视频世界模型的局限不在于画质或交互速度，而在于**建模对象**——它们仍以"接下来应该生成什么观察"为核心，视觉历史只能记录看得见的结果，无法保存世界规则、角色关系、离屏因果链与跨长时程的事件后果。视频上下文通常短于一分钟，而角色关系、社会结构与事件后果可能在世界时间内的几天甚至几年中持续演化。扩大视频训练规模只能增加"见过的结果"，不会自动提供生成这些结果的白盒机制。^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]

## 架构：三组件分工

- **Coding Agent（世界大脑）**：读取当前世界状态、解释新的交互或事件、决定哪些实体与机制需要改变，选择调用已有代码或局部改写世界程序。Agent 不必以视频帧率工作，只需处理低频但复杂的推理（理解事件、关联世界知识、规划后果、修改机制）^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]
- **代码（可执行延伸）**：持续完成密集、确定、可复用的状态更新（位置、数值、日程、冷却、碰撞、规则），无需每一步都再调用大模型。代码不是写死的游戏逻辑，也不是外部控制器——Agent 可组合、调用或修改代码，改变的不仅是当前状态，还包括世界今后如何运行^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]
- **视频模型**：利用大规模视觉数据学到的外观、运动与交互先验，把可执行世界状态实现为高质量观察。它不需要从零学习完整规则，只需把明确的世界状态实现为视觉^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]
- **Proxy（视觉通道）**：由世界状态确定性编译得到的粗粒度视觉条件（实体位置、近似尺度、姿态、运动轨迹、遮挡、相机运动），横纵分辨率取目标视频的四分之一（视觉 token 约为目标视频的 1/16）。结构化文本负责身份/外观/动作语义，Proxy 负责逐帧空间与时间约束，整条"状态—条件"通路保持可检查、可寻址、可局部修改的白盒系统^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]

## 世界状态的双轨拆分

完整世界状态分为两部分：**可执行状态**保存程序、实体属性、规则、关系与事件历史；**视觉状态**保存由视频模型生成且需要在时间上保持一致的外观与运动信息。两部分由不同机制更新但彼此耦合——代码确保规则与后果持续存在，视频模型提供高保真观察实现。^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]

## 训练数据与实现

- 游戏天然适合 Proxy 对齐：运行 GTA V 同步记录画面与相机/实体/场景/交互状态，代码可从同一次运行反复编译不同覆盖范围与粒度的 Proxy，无需重新采集 RGB 视频
- KITTI-360 几何辅助概念验证：校准相机位姿、语义 3D 重建与物体标注仅用于离线编译 Proxy，模型最终仍接收 RGB 目标 + Proxy 视频 + 结构化文本
- 原型适配 MiniMax-H3 Ref2VA 视频模型：157 段游戏视频约 5.6 小时，两秒间隔采样出 9,420 个五秒片段（RGB 124 帧 @1344×768@24FPS，Proxy 336×192）；rank-128 LoRA 覆盖 50 个 Transformer block（约 5.96 亿参数），8 张 NVIDIA H800 训练 3 epoch
- 推理：GPT-5.6 Sol 作 Coding Agent，GPT Image 2 依首帧 Proxy + 文本提示生成外观锚点
- 长时生成：带重叠的 124 帧窗口（相邻重叠 34 帧、每次推进 90 帧），前一窗口末尾 RGB 保证局部连续性，同一首帧锚点维持全局外观^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]

## 范式意义

- **职责重分工**：把高层推理与低层执行解耦。Agent 不必以帧率工作，代码不必具备开放式常识推理，视频模型无须从零学完整规则。即使未来语言与视频能力统一进同一多模态网络，外部代码维护的持续世界状态仍需要高效可控的输入接口——Proxy 讨论的"状态—视觉条件"问题依旧存在^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]
- **应用方向**：能保持规则、记住离屏变化并生成高保真观察的开放世界，可成为训练与评估智能体的环境，服务具身智能、自动驾驶与长期规划^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md]

## 相关实体

- [[concepts/world-models|世界模型概念]]
- [[entities/worldtrace-addressable-memory-video-world-models|WorldTrace：可寻址记忆视频世界模型]]
- [[entities/loopwm-looped-world-models|LoopWM：循环世界模型]]
- [[entities/feifei-li-masked-visual-actions-world-model-2026|李飞飞：掩码视觉动作世界模型]]
- [[entities/nvidia-gamma-world-multi-agent-world-model|Nvidia Gamma World：多智能体世界模型]]
- [[entities/qwen-agentworld-language-world-models|Qwen AgentWorld：语言世界模型]]

→ [[raw/articles/code-world-model-coding-agent-as-world-brain-2026|原文存档]]