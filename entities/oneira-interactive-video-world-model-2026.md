---
title: "Oneira：状态表驱动的交互式视频世界模型"
created: 2026-10-14
updated: 2026-11-19
type: entity
tags: [world-model, video-generation, interactive, coding-agent]
sources: [raw/articles/oneira-interactive-video-world-model-2026]
confidence: 0.7
---

# Oneira：状态表驱动的交互式视频世界模型

arxiv 2610.01614。针对视频世界模型的两个顽疾——交互作用在错误物体上、镜头移开再返回后物体变化消失——提出「世界中发生了什么」与「画面如何呈现」分离的架构 ^[raw/articles/oneira-interactive-video-world-model-2026.md]。

## 核心机制：World State Table + Coding Agent

- **Coding Agent** 通过 World State Table 记录场景中每个物体的位置与状态；
- 给定动作/目标后，Agent 先读表确定交互对象并规划交互，再把结果**写回表**——状态持久化在符号层，不依赖生成画面的连续性；
- 交互准确性与长时一致性上明显优于现有方法。

这与游戏引擎的显式状态管理同构，但状态表由 agent 在生成过程中维护，场景可随生成内容扩展。与 [[entities/alaya-evoke-hour-level-video-world-model]]（长时视频世界模型）相比，Oneira 把一致性从生成侧转移到状态侧；与 [[entities/code-world-model-coding-agent-as-world-brain-2026]]（Coding Agent 作为世界大脑）共享「代码/符号层承载世界状态」的思想。

- 代码：github.com/Madaoer/Oneira

## 深度分析

### 为什么「画面即状态」会失败

现有视频世界模型已经能随镜头移动不断生成新场景，但两个顽疾始终没有解决：交互常常落在错误的物体上；即使对象选对，镜头移开再返回后，物体的变化也常常消失 ^[raw/articles/oneira-interactive-video-world-model-2026.md:33-35]。团队把病灶归结为同一个存储层错位问题：场景里有哪些物体实例、哪些发生过变化，全部只隐式地存在于生成的像素和上下文里——模型无法区分多个相同物体（两个苹果只能靠文本描述猜），交互结果也只活在画面里，随视频推进而丢失。Oneira 的回答是把「世界中发生了什么」从画面中剥离出来，交给 Coding Agent 用 World State Table 单独记录；一次交互只写符号层（读表确定对象并规划，再把结果写回），像素层只负责呈现 ^[raw/articles/oneira-interactive-video-world-model-2026.md:37-39]。这与游戏引擎的显式状态管理同构，但方向相反：传统模拟器能精确记录状态，场景却必须预先构建；Oneira 的状态表随生成内容动态扩展，镜头里新生成的物体也能直接交互 ^[raw/articles/oneira-interactive-video-world-model-2026.md:35-39]。

### 符号层记录什么、不记录什么：一次刻意的最小化

World State Table 的字段设计是这套架构能 work 的关键：只记可行走地面、语义标签、3D BBox 和交互状态四类与交互直接相关的信息，**外观不写入表**，交给 Video Generation Model 生成 ^[raw/articles/oneira-interactive-video-world-model-2026.md:51]。Engine 先把表渲染成简化的条件视频，真实的外观、动作与交互细节再由生成模型补全 ^[raw/articles/oneira-interactive-video-world-model-2026.md:41]；条件视频的语汇非常克制——每个物体一个彩色方框，颜色变化表示状态改变，方框消失表示物体离位，网格与彩色背景提示相机运动 ^[raw/articles/oneira-interactive-video-world-model-2026.md:55-57]。苹果怎样滚落、滚到哪里，表里没有，也不需要有 ^[raw/articles/oneira-interactive-video-world-model-2026.md:53]——这类「持久层只存结构不存外观」的分界与 [[entities/dawa-scenes-ai-persistent-world-model-2026]] 的取舍一致。另一个隐蔽的好处是：3D BBox 只用颜色编码状态、与物体类别无关，因此模型能对训练中没见过的物体类别执行交互——泛化来自条件化方式，而非记忆 ^[raw/articles/oneira-interactive-video-world-model-2026.md:57]。

### 生成-登记-扩展的闭环

符号层不是一次性建表，而是一个「生成-登记-扩展」的闭环：镜头移动生成出新物体后，Coding Agent 在**生成的画面**上调用现成的 3D 感知工具得到 3D BBox，登记进表 ^[raw/articles/oneira-interactive-video-world-model-2026.md:39]；此后每生成一段视频都重复一次这套登记，可交互物体随探索不断增加 ^[raw/articles/oneira-interactive-video-world-model-2026.md:55]。这也意味着符号层的可信度最终取决于两个外部组件——3D 感知模型的定位质量与生成画面的保真度。多步交互里还能看到闭环的自我对账：目标是「拿起左下角的苹果」时，Agent 不仅选定是哪一个苹果、规划在哪个 chunk（每段约 10 秒视频划 8 个 chunk）执行，还推断出另外两个苹果会随之滚落并把三者一起写入规划；等它们静止后，Agent 再从新画面重新登记位置，连新出现的梨也一并入库 ^[raw/articles/oneira-interactive-video-world-model-2026.md:53]。每段视频的最后一帧连同更新后的表作为下一段的起点 ^[raw/articles/oneira-interactive-video-world-model-2026.md:49]——这相当于用「读感知-写表」的循环对生成漂移做对账，思路与 [[entities/worldtrace-addressable-memory-video-world-models]] 的可寻址记忆同向，但 Oneira 把对账对象从记忆换成了显式状态表。

### 度量与边界

团队构建了 InteractionBench（100 个交互测例 + 50 个记忆测例），对比 MiniMax-H3 Ref2VA、LingBot-World-V2、YUME 1.5 和 AlayaWorld ^[raw/articles/oneira-interactive-video-world-model-2026.md:67]。交互测例上，Oneira 在交互是否发生、时机、对象三项均最优，比最强基线分别高出 29、19、29 个百分点；多相同物体场景中基线常作用在错误物体上或同时改变多个物体，而 Oneira 只改变指定物体 ^[raw/articles/oneira-interactive-video-world-model-2026.md:69]。记忆测例上，镜头返回后 PSNR/SSIM 提升 93%/83%，状态保持率提升 18%，VBench 视频质量仍保持最好或接近最好 ^[raw/articles/oneira-interactive-video-world-model-2026.md:71]。不过边界要清醒看待：视频按约 10 秒分段、每段划 8 个 chunk，一致性是分段条件化加上镜头返回时参考的记忆帧拼出来的，不是无限上下文 ^[raw/articles/oneira-interactive-video-world-model-2026.md:53-57]；符号层精确不等于画面精确，视觉质量仍受限于底层视频生成模型；且评测集由团队自建，跨基线可比性有待第三方复现。整体上，它验证的是一条架构假设——让生成世界真正可交互，瓶颈不在生成质量，而在状态的独立记录与写回 ^[raw/articles/oneira-interactive-video-world-model-2026.md:43]。

## 关联
- → [[raw/articles/oneira-interactive-video-world-model-2026|原文存档]]
