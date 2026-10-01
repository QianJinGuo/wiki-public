---
title: "Code World Model：让代码接管世界演化（Coding Agent as World Brain）"
created: 2026-09-08
updated: 2026-10-02
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

## 深度分析

### 为什么把世界演化从视频先验中剥离是范式转变

传统视频世界模型把状态、规则、记忆与视觉预测全部压进同一条生成序列，根本困境在于：视频只记录演化后的可见结果，产生结果的规则在画面变成训练视频那一刻就被丢掉，模型只能从稀疏视觉后果中反推隐藏逻辑^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:49]。更致命的是时间尺度失配：视频上下文通常短于一分钟，而角色关系与事件后果可能在世界时间内的几年里持续演化，且大量关键变化发生在镜头之外——扩大视频训练规模只增加"见过的结果"，不会自动给出白盒机制^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:51]。

Code World Model 的回答是把问题倒过来：把"世界如何演化"与"世界如何被看见"拆开^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:53]——低频复杂推理（理解事件、规划后果、修改机制）交给 [[concepts/world-models|世界模型]] 语境下的 Coding Agent，高频重复执行交给可执行代码，视频模型降级为把明确状态实现为观察的渲染器。这不是增加一种控制条件，而是改变了职责分工——把可执行世界状态从生成序列的附带产物提升为系统中心^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:115]，世界从"连续生成的画面"变成"能被理解、修改、执行并持续留下后果的运行系统"^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:127]。

### Proxy：1/16 token 成本买到的是白盒可寻址性

纯文本无法以低延迟逐帧描述位置、遮挡与相机轨迹；让 Agent 直接构建完整 3D 世界控制更强，却需要一整套渲染系统，还会过早限定最终画面^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:73]。Proxy 选择中间路线：由世界状态确定性编译出的粗粒度视觉条件，只保留观察必须遵守的最小状态（位置、近似尺度、姿态、轨迹、遮挡、相机运动）^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:75]。

成本很清楚：横纵分辨率取目标视频的 1/4，视觉 token 约为 1/16，额外推理负担相对有限^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:79]。真正的收益不在省 token，而在于控制信号全部可追溯到世界状态——"状态—条件"通路成为可检查、可寻址、可局部修改的白盒系统^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:77]。权衡在于精细度：过丰富会要求 Agent 同步维护大量细节轨迹，过稀疏又约束不住视频模型，"可构建性"与"约束强度"的平衡才是设计核心^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:79]。

### 与现有视频世界模型的对比定位

- **WorldTrace**（[[entities/worldtrace-addressable-memory-video-world-models|WorldTrace：可寻址记忆视频世界模型]]）：同样诊断长时程崩溃，但解法停留在生成序列内部——用 slot-rank 虚拟位置让压缩记忆槽保持可寻址，状态仍以 KV 缓存形式隐式存在；Code World Model 把状态整体搬出视频通路，记忆即代码里的实体属性与事件历史。
- **LoopWM**（[[entities/loopwm-looped-world-models|LoopWM：循环世界模型]]）：用循环调用同一组 Transformer 块在潜空间反复推演环境状态，以迭代深度换参数深度；但潜向量不可白盒寻址，外部 Agent 也无法直接修改规则，而 Code World Model 的演化载体是可读写的程序。
- **Gamma World**（[[entities/nvidia-gamma-world-multi-agent-world-model|Nvidia Gamma World：多智能体世界模型]]）：解决多 Agent 独立可控与置换对称，控制粒度是"哪个 Agent 表现如何"；Code World Model 控制的是"世界规则本身如何变化"，且能保持离屏因果链。

三者都在"视频先验"一侧做纵深优化；Code World Model 把重心移到先验之外的可执行状态，属于互补而非替代。

### 数据策略：把"对齐"而非"像素"当作可复用资产

游戏数据的独特价值不是画质，而是运行时可同步记录相机、实体、场景与交互状态，能从同一次运行反复编译不同覆盖范围与粒度的 Proxy，无需重新采集 RGB 视频^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:87]。真实视频入口用 KITTI-360 几何辅助做了概念验证：位姿、3D 重建与物体标注只用于离线编译 Proxy，模型仍接收 RGB + Proxy + 文本，动作标签无需独立定义^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:89]。原型仅用 5.6 小时、rank-128 LoRA 即完成适配^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:93]——视频模型要学的只是"读 Proxy 生成画面"这一层映射，规则知识全由代码侧承担。

## 实践启示

1. **拆分职责再选型**：构建世界模拟器时，先区分"低频复杂推理"与"高频重复执行"两类负载^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:53]；前者用 LLM Agent，后者用确定性代码，不要试图把两者都压进一个视频模型。
2. **状态先行，观察随后**：把可执行世界状态（程序、实体属性、规则、事件历史）作为第一公民持久化^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:115]，视觉渲染只是状态的一个视图——这天然解决离屏事件与长时程一致性。
3. **控制条件要白盒可寻址**：逐帧空间约束优先选"由状态确定性编译"的中间表示（Proxy 模式），token 开销可控制在目标视频的 1/16 量级，且每条控制信号都可追溯、可局部修改^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:77]。
4. **用游戏引擎制造对齐数据**：从同一次运行记录反复编译多粒度条件，比重新采集视频便宜得多；真实视频侧则借几何重建离线编译条件，接口保持统一^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:87]。
5. **微调而非重训视频模型**：视频模型只需学习"状态→观察"映射，小规模 LoRA（原型 5.6 小时数据）即可^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:93]；规则知识由代码承担，不为视觉先验付费。
6. **长时生成用状态锚点而非画面拼接**：重叠窗口保局部连续性、同一首帧外观锚点保全局身份，而 Proxy 条件贯穿完整时间轴——即使画面风格周期变化，轨迹与空间关系仍有持续状态来源^[raw/articles/code-world-model-coding-agent-as-world-brain-2026.md:109]。

## 相关实体

- [[concepts/world-models|世界模型概念]]
- [[entities/worldtrace-addressable-memory-video-world-models|WorldTrace：可寻址记忆视频世界模型]]
- [[entities/loopwm-looped-world-models|LoopWM：循环世界模型]]
- [[entities/feifei-li-masked-visual-actions-world-model-2026|李飞飞：掩码视觉动作世界模型]]
- [[entities/nvidia-gamma-world-multi-agent-world-model|Nvidia Gamma World：多智能体世界模型]]
- [[entities/qwen-agentworld-language-world-models|Qwen AgentWorld：语言世界模型]]

→ [[raw/articles/code-world-model-coding-agent-as-world-brain-2026|原文存档]]