---
title: "扩散模型视觉生成一致性框架（2026 综述）"
type: entity
created: 2026-07-02
updated: 2026-08-01
tags: [diffusion, visual-generation, consistency, survey, cv, multimodal, generative-ai]
rating: v7c8
sources:
  - raw/articles/diffusion-model-consistency-survey-ustc-2026
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


## 相关实体

→ [[raw/articles/diffusion-model-consistency-survey-ustc-2026|原文存档]]

> 论文：https://www.preprints.org/manuscript/202606.0870/v1
> 开源仓库：https://github.com/Shawn-CodeDev/Awesome-Consistency-Diffusion-Visual-Generation

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

