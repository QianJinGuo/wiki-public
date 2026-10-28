---
title: "Scenes AI：把世界模型做成可读写的持续世界状态"
created: 2026-10-26
updated: 2026-10-26
type: entity
tags: [world-model, real-time, diffusion, inference, rendering, architecture]
sources: [raw/articles/dawa-scenes-ai-persistent-world-model-2026]
confidence: 0.75
provenance_state: extracted
---

# Scenes AI：把世界模型做成可读写的持续世界状态

达瓦科技（重庆）的实时世界模型 Scenes AI 在 2026 NVIDIA 创业企业展示半程展示中拿下「半程展示卓越企业奖」——本届 580 多家企业报名、30 家进入半程。产品的核心主张是一句可以当判据用的话：**A video is an output. A world is a process.** 视频是一个结果，世界是一个过程。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

## 范式差异：更新状态增量，而不是重画下一帧

主流视频生成比拼"下一帧像不像"，误差在时间轴上累积——身份、几何、光照、物理都会漂移。Scenes AI 维护一个持续运行的世界状态：场景、人物、相机、灯光、物体、物理各自是可单独读写的变量；**每发生一次变化，模型只更新变化的那部分状态（ΔS），再把结果写回世界**。于是它回答的问题从"下一帧长什么样"变成"改变世界里的一个变量，世界接下来会发生什么"。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

最直观的对照是输入一句"把灯光变成夕阳"：传统视频模型会重生成一遍画面，场景、人物、相机可能一起漂移；Scenes AI 把其余变量冻结为保持条件，只让"光照"改变，再由新状态生成画面。这个差别决定了模型是否可被**编程**——不是给提示词，而是给状态变量赋值。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

## 架构拆解：Persistent World = 五件套

官方给出的分解是 `World Memory + Neural Physics + Sparse Diffusion + Neural Rendering + AI Lighting`，对应五个必须同时成立的能力面：

| 组件 | 解决的问题 | 工程含义 |
|---|---|---|
| World Memory | 跨帧记忆，细节不被重画冲掉 | 状态以显式变量持久化，而非藏在隐特征里 |
| Neural Physics | 因果与物理一致性 | 事件驱动的状态演化，不是像素统计 |
| Sparse Diffusion | 只生成变化区域 | 计算量与"变化量"绑定，不随分辨率线性爆炸 |
| Neural Rendering | 保持几何/遮挡/透视 | 与 3D 表征对齐，避免逐帧重绘导致结构漂移 |
| AI Lighting | 光照作为独立可写变量 | 拆出可控旋钮的典型例子 |

^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

达瓦把必须同时通过的门槛归纳为五道关：**跨帧记忆、局部可控、物理与因果、实时计算效率，以及能解释"世界为何这样变化"的数据**。用他们的话说，难点不是造一个更大的模型，而是状态、控制、物理与算力必须同时成立。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

## 评测考点：结构保持与局部改写持久性

两个 Demo 考点暴露了这条路线与逐帧生成的判别差异。**考点一（全局重渲染）**：整屋换画风时，唯一被改写的是"画风"变量，人物动作、家具位置、遮挡关系、空间透视被锁为保持条件——椅子仍在少女身后、小熊仍在原墙角、墙面的折线仍在草地上隐约可见。**考点二（局部改写持久性）**：小朋友在墙上写下"DW未来"后离场，随后长出树、燃起篝火、飘雪下雨，墙上四个字的位置和笔画始终不变；因为"墙上有字"在写下的那一刻已成为世界状态的一部分，没有事件去改它就一直在。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

对照 9 月首发演示的同构案例：雪怪破土离开后地上的坑留在原地；铁艺椅子连续两次被改写成狼首铁王座，主播还能坐回原位；窗帘下摆起火时紧邻的玩偶与花盆未被波及。**"改一个变量，保住整个世界"**在演示层面是被反复检验的同一命题。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

## 实时性与输入形态

整套生成跑在本地一张 RTX 显卡上（报道标题口径为"一张 5090 跑起 34B 实时世界模型"），画面右上角实时显示帧率；输入不是游戏引擎或代理画面，而是**一台普通摄像头拍到的真实空间**。这两点合起来，把"世界模型"从离线视频生成推向了可交互的实时系统，也把算力约束推到了前台。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

## 判断

这条路线真正的差异化不在模型规模，而在**接口设计**：一旦世界被表达为可读写状态，控制、回放、局部编辑、因果解释都变成工程问题；反之若状态只存在于隐空间，所谓"可控生成"永远只能靠提示词近似。风险同样明显——状态空间的完备性与物理保真度决定了"世界"能撑多久不崩，演示级的局部持久性不等于长时程一致性。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md]

## 深度分析

### 状态增量 ΔS：误差从"像素弥散"变成"变量可定位"

逐帧生成的误差是弥散的：身份、几何、光照、物理的偏差混在像素里，随时间累积，且无法归因到某个部件。Scenes AI 把交付物换成一组建模好的状态变量，每步只算 ΔS，等于把误差的载体从像素搬到变量上——出错时能问"是哪个变量错了"，而不是"这一帧为什么不像"。这也正是它可被编程的原因：接口从提示词变成了赋值。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:30-36] ^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:92]

代价随之转移：状态空间的完备性成了新瓶颈——没被建模成变量的东西不会被"保住"，只会被重画。达瓦划的那条线是"瞬态"与"持久"：天气、火焰交给渲染层处理完即可，写下的字、地上的坑必须进入世界状态。这个分类是设计决策，不是模型自发现的能力。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:132-134]

### 五件套：不是流水线，是五个会各自坏掉的地方

World Memory 管跨帧记忆，坏法是遗忘与漂移——镜头离开再回来时重新"想象"一面墙；Neural Physics 管因果，坏法是越界——火烧过界、布料被穿透；Sparse Diffusion 只生成变化区，坏法是接缝——生成区与缓存区拼不齐；Neural Rendering 管几何与遮挡，坏法是透视漂移；AI Lighting 把光照拆成独立可写变量，坏法是泄漏——改光时人物与环境被一起重画。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:96] ^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:162-174]

串起它们的是同一条链路：世界状态 → 行动或接触事件 → 状态更新 → 局部生成与合成 → 结果写回；分工原则是**控制层做重、生成层做轻**——状态因子在控制层被拆解、约束与推演，渲染层只接收条件下发的"需要变的部分"。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:110-120] 五个模块的失效症状都表现为"画面某处不对"，根因却在状态层，这是调试这类系统最容易走错的地方。World Memory 的查询路径可参照 [[entities/worldtrace-addressable-memory-video-world-models]] 的"记忆可寻址"思路。

### 评测为什么难：一个考绑定，一个考生命周期

考点一（整屋换画风）考的不是风格迁移质量，而是**实体绑定**：模型得先把人、椅子、玩偶、柜子认成可保持的实体，再在全局重绘中把它们钉回原位，而且每一帧都成立。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:52-56] 像素相似度指标测不出这件事——画风全换时像素必然全变，唯一的不变量是对象间关系：椅子相对少女的遮挡、小熊相对墙角的位置、墙交汇的折线在草地上残留。

考点二（墙上的"DW未来"）考的是**状态生命周期**：字在写下的瞬间进入世界状态，此后长树、燃篝火、飘雪下雨都不该动它。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:58-66] 难点反直觉——正确行为是"静止"，系统的默认倾向却是遗忘。逐帧模型每帧都要重新决定墙上有无字，等于每帧给遗忘一次机会；状态化系统只需保证"没有事件就不改写"，考的是不变式而非记忆容量。衡量也应是双指标：目标编辑成功率 98.2% 对应"改动集改对了"，人物身份保持率 93.3% 对应"保持集守住了"。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:176] 评测设计可对照 [[entities/world-model-evaluation-position-paper-nju-2026]]。

### 实时性不是优化项，而是输入形态的函数

典型镜头每帧真正变化的区域只占画面 10%–20%，于是墙面、地板走潜空间缓存直接复用，算力集中给人脸、手部、布料、烟雾与交互区；34B 总参数每步只激活约 4B（11.8%）；两者叠加换来 GPU 消耗降 40%–60%、推理提速 3–5 倍，单卡跑 720P/25fps。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:186-206] 变化范围超过阈值（整屋切换）时按范围动态分配算力，局部编辑与全场景转换共用同一条管线。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:208-210]

这也解释了输入为什么必须是普通摄像头：若输入是引擎渲染，状态可由引擎直接给出，模型就退化成渲染器；面向真实拍摄，状态必须从像素里解析出来再维护、推演、写回。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:136-140] 25fps 一旦是硬指标，容量就只能靠稀疏激活换、不能靠堆参数——实时是设计前提，不是事后优化。

### 与视频生成、3D 重建的边界

对视频生成：交付物从"一段结果"变成"一个进程"，但画质下限仍由渲染层决定，状态正确不等于好看。对 3D 重建：重建交付几何，这里交付"几何+物理+叙事"的可写状态，几何只是其中一个变量，所以它能表达"坑还在""字还在"这类非几何事实。对游戏引擎：引擎状态是人事先写的规则，这里的状态从真实画面解析而来——前者可控可解释，后者适配真实空间但可控性更弱。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:136-140] 同类实时交互路线可对照 [[entities/pixverse-r2-real-time-world-model-interactive-entertainment]]。真正稀缺的不是模型，而是带过程标注的状态数据："不是 Video Dataset，是 State Dataset"。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:228-234]

## 实践启示

1. **先定状态 schema，再选模型。** 把系统拆成可读写的变量（场景、人物、相机、光照、物理、叙事），并写明每个变量默认的改动集与保持集；编辑语义定在 schema 层，模型只负责执行。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:114] ^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:148-152]
2. **把瞬态/持久做成显式判定，别交给模型猜。** 误判代价不对称：持久当瞬态会被冲掉，瞬态当持久会让状态只增不减。
3. **为"无事件即不变"写不变式与回归用例。** 构造长事件序列（生长、燃烧、降雪、降雨）后断言早期写入对象的属性逐项不变。
4. **算力预算绑变化量，不绑分辨率。** 缓存稳定区域、只对变化区采样，同时保留按变化范围放大预算的路径，让局部编辑与全场景切换共用管线。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:188-208]
5. **用稀疏激活换容量。** 大参数量保能力、小激活量保延迟，两者分开调；这是实时约束下仍能扩容量的方向。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:202-204]
6. **数据从过程采，不要从成片剪。** 成片会剪掉机位、打光、接触的过程信息；没有片场就用仿真补精确状态、用生成数据补长尾覆盖。^[raw/articles/dawa-scenes-ai-persistent-world-model-2026.md:228-242]

## 关联

- [[entities/baai-orca-next-state-prediction-world-model]] — 同属"下一状态预测"而非"下一帧生成"的建模范式
- [[entities/code-world-model-coding-agent-as-world-brain-2026]] — 把世界模型当作 agent 内部世界表征的另一条路线
- [[entities/amap-abot-earth-0.5-3d-native-world-model]] — 3D 原生世界模型，与稀疏局部生成形成对照
- [[concepts/agent-harness-engineering-paradigm]] — 状态可写是世界模型接入 agent 闭环的前提

→ [[raw/articles/dawa-scenes-ai-persistent-world-model-2026|原文存档]]
