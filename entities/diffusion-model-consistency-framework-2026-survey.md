---
title: "扩散模型视觉生成一致性框架（2026 综述）"
type: entity
created: 2026-07-02
updated: 2026-09-18
tags: [diffusion, visual-generation, consistency, survey, cv, multimodal, generative-ai]
rating: v7c8
sources:
  - raw/articles/diffusion-model-consistency-survey-ustc-2026
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 扩散模型视觉生成一致性框架（2026 综述）

中国科学技术大学、清华大学、华中科技大学、剑桥大学等机构联合发表的重磅综述，系统梳理了 500+ 篇文献，揭示了扩散模型视觉内容生成繁荣表象下的「一致性危机」，并提出三类一致性的统一分析框架：外部一致性、内部一致性和规范一致性。 ^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

## 三类一致性关系

该综述将扩散视觉生成中的一致性问题归纳为三种基本关系：^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]


### 外部一致性
生成结果与用户条件之间的一致。模型是否真正实现了文本 prompt、布局、参考图或编辑指令中的要求？常见失败模式包括物体遗漏、属性错绑、数量错误和空间关系混乱。代表方法：Attend-and-Excite、BoxDiff、GLIGEN、ControlNet、T2I-Adapter、IP-Adapter、DiffEdit、Prompt-to-Prompt、InstructPix2Pix。^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

### 内部一致性
多个生成结果之间的一致。当同一个主体出现在不同图片、不同视角或不同时间时，模型是否仍然维护着同一个对象和同一个世界？涵盖个性化生成（DreamBooth、PhotoMaker、InstantID）、多视图生成（Zero-1-to-3、SyncDreamer、MVDream）、视频与故事生成（AnimateDiff、StoryDiffusion、TaleCrafter）。核心挑战：身份漂移、物体消失、动作断裂、事件矛盾。^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

### 规范一致性
生成内容与人类及现实世界标准的一致。即使模型完美执行了 prompt 指令，仍可能不符合人类偏好、包含不安全内容，或违反物理和因果规律。代表方法：ImageReward、HPS、VisionReward、Diffusion-DPO、FlowGRPO、DiffusionNFT。相关基准：PhyBench、VideoPhy、PhyGenBench。^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

## 一致性的实现位置

一致性的优化可在扩散生成流程的五个不同阶段实现：^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

1. **训练阶段** — 改变数据和目标函数，将约束写入模型参数（如 DreamBooth 的身份训练、Diffusion-DPO 的偏好优化）
2. **条件接口** — 约束条件如何被编码和注入模型（ControlNet、T2I-Adapter、GLIGEN、IP-Adapter）
3. **去噪轨迹** — 直接干预采样过程修正注意力/中间 latent（Attend-and-Excite、Prompt-to-Prompt、BoxDiff）
4. **联合生成** — 多图片/多视角/多帧共享特征、注意力或状态（SyncDreamer、MVDream、AnimateDiff）
5. **事后验证** — 生成完成后用奖励模型、安全过滤器、重排序器筛选结果

## 评价困境

单一总分无法衡量一致性，原因在于不同一致性属性无法在同一种观察对象上被测量：^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]
- Prompt 一致性需比较一张图片和一段文本
- 身份一致性需观察同一主体的多个生成结果
- 多视图一致性需检查多个视角
- 视频一致性需沿时间追踪状态

评价需明确四要素：观察单位（单图/图像对/集合/序列）、检查维度（语义/结构/身份/几何/时间）、测量方法（VQA/特征相似度/几何信号/奖励模型）、输出类型（正确率/保持度/偏好分数/风险诊断）。^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]


## 冲突与权衡

不同一致性目标之间存在根本性冲突：^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]
- 更严格的 prompt 执行可能损害审美质量
- 更强的身份绑定可能限制可编辑性
- 更紧密的时间耦合可能压缩运动多样性
- 更严格的安全/物理约束可能限制开放创造

未来方向：从分别强化不同约束走向理解、解释和处理约束冲突的生成系统，具备冲突感知、持久但可编辑的状态、可解释评价和世界结构化能力。^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]



## 深度分析

### 「一致性」是一个伞词，而不是一个指标

三类关系共用一个词，却连测量对象都不共享：外部一致性比较的是「输出 与 输入条件」，内部一致性比较的是「输出 与 另一个输出」，规范一致性比较的则是「输出 与 一套长期生效的外部标准」。这意味着"提升了一致性"是一个不完整句子——它省略了主语（哪类关系）、观察单位（在什么粒度上测）和量纲（输出的是正确率、保持度还是偏好分）。而这恰恰解释了为什么该领域方法数量爆炸却没有可累积的排行榜：不同论文的"一致性"往往在互不相交的坐标上读数，数值无法互相校准。 ^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

由此可以推出一个相当有用的判据：任何一个一致性主张都可以被规范化成三元组——在 X 类关系上、对 Y 观察单位、用 Z 测量并报告输出量纲。凡是不足以填满这三项的表述，实际上不是可检验的技术声明，而是修辞。

### 干预位置的五层成本结构

综述给出的五个实现位置并非并列清单，而是一条按介入时机排序的成本梯度：越早介入越根本、也越不可撤销。写入参数（训练阶段）获得最强的持续性，代价是重训开销与对模型其他能力的潜在侵蚀——而且它绑在权重上，无法按请求开关。条件接口（ControlNet 一类）可插拔，代价是外部条件的表达带宽被接口设计提前限定。去噪轨迹干预无需重训、反馈即时，但干预强度一旦超过阈值就会反噬画质、多样性与采样效率。联合生成把一致性从「样本属性」提升为「过程属性」，代价是显存与延迟随样本数增长。事后验证接入成本最低，却只能筛除已经产生的坏样本，无法降低坏样本的产生概率——它提升的是系统的可见质量，而不是模型的能力。 ^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

一个容易被忽略的推论是：位置的组合并不等于收益的累加。多个模块若同时修改同一组特征，会互相覆盖或提出相互矛盾的要求，而组合越深，这种耦合越难归因。

### 一致性与多样性：不是简单零和，但账必须记

综述列出的冲突关系是结构性的：更严格地落实 prompt 往往迫使模型生成不自然的构图；更强的身份绑定会把服装、背景与姿态一并锁死，压缩可编辑空间；更紧的跨帧耦合抑制闪烁的同时也抑制运动幅度；过激的安全擦除会误伤无害概念。这些不是调参能消掉的误差，而是目标之间的真实张力。 ^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

真正的困难因此不在「把某个指标做到最高」，而在多目标同时激活时决定谁让路。这需要系统能区分硬约束（不可退让的条件，如数量、身份、安全性）与软偏好（可协商的审美与构图倾向），并在冲突发生时说明为了提升一个目标牺牲了什么。当前多数系统仍是一个模块负责一个目标的事后拼接，而拼接本身不产生协调机制——这是从「高质量生成」走向「可靠生成」之间最缺的一块。

### 自动指标与人类判断能否收敛

综述的判断是：许多评价失败不是指标不够先进，而是观察单位选错。单张图片里不存在「跨帧身份漂移」；两张相邻帧看起来平滑，也不能证明几十秒后角色与场景仍然一致；人脸相似度很高，不代表服装与配饰没有变化。 ^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

据此可以把两种分歧分开处理。第一类是测量对象错位——用单图指标去回答序列问题，用图文相似度去回答空间关系问题。这类分歧原则上可以收敛，只要把观察单位（单图 / 图像对 / 集合 / 视角组 / 长序列）与检查维度对齐到同一个对象上；[[concepts/agent-evaluation-benchmark-frameworks|评测基准框架]]的许多设计原则在这里同样适用。第二类分歧则来自规范一致性本身没有单一真值：偏好、安全边界与「可接受的物理合理性」随场景与文化漂移。「机器人和仿真需要严格物理约束，超现实主义创作不需要」——这类判断不可能被任何单一指标吸收。对第二类，可解释的评价报告（说明在哪种关系上成功或失败）是唯一现实的共存方案，而不是追求一个收敛的总分。 ^[raw/articles/diffusion-model-consistency-survey-ustc-2026.md]

### 一个可证伪的一致性主张长什么样

把上面几条合起来，"可证伪"的条件相当具体：必须指明观察单位、检查维度（语义 / 结构 / 身份 / 几何 / 时间状态 / 规范）、测量手段与输出量纲；必须报告改进所付出的代价（画质、多样性、可编辑性、延迟）；并且改进应当出现在它所声称的那一类关系上，而不是在另一个未声明的坐标上。反面例子是常见的那种句子——"本方法显著提升了一致性"，既没有观察单位也没有代价声明，它无法被证实，也同样无法被证伪。一个诚实的报告应该形如：在「同一主体跨视角」这一内部一致性的集合级观察单位上，身份保持度上升，同时可编辑性下降 X。这类句子可以被下一代方法推翻，因此才算得上科学主张。

## 实践启示

1. **按关系拆分评测，禁止用总分宣称"更一致"** — 报告至少分别覆盖外部 / 内部 / 规范三类关系，并在每类下写明观察单位与量纲；单一总分在一致性问题上没有可比较语义。
2. **先选观察单位，再选指标** — 单图、图像对、集合、视角组、长序列互为不可替代；用错单位得到的不是噪声而是系统性偏差，指标再先进也救不回来。
3. **显式区分硬约束与软偏好** — 把必须满足的条件（对象数量、身份、安全）与可协商的偏好（构图、风格、光照）分开配置，冲突时才有依据决定谁让路；否则系统会在无意中牺牲真正的硬约束。
4. **按成本梯度选干预位置** — 能承担重训就把约束写入参数（最持久）；需要快速迭代就用条件接口或去噪轨迹干预；只有资源受限时才把事后过滤当兜底——但要清楚它只改变可见质量，不改变模型的生成倾向。
5. **任何一致性改进都必须附带代价声明** — 画质、多样性、可编辑性、延迟中的哪一项被换掉了、换掉多少。没有代价声明的改进结论不可复现，也不应被采信。
6. **面向长视频、多视图与具身场景时，把一致性当状态维护问题** — 交叉实例共享特征、注意力或外部记忆（[[entities/ard-agentic-autoregressive-diffusion-for-long-video-consistency|长视频一致性方向]]）比单帧修正更接近问题本质；这与 [[concepts/world-models|世界模型]] 需要维护对象、状态与因果演化的要求是同一件事。

## 相关实体

→ [[raw/articles/diffusion-model-consistency-survey-ustc-2026|原文存档]]

> 论文：https://www.preprints.org/manuscript/202606.0870/v1
> 开源仓库：https://github.com/Shawn-CodeDev/Awesome-Consistency-Diffusion-Visual-Generation

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

