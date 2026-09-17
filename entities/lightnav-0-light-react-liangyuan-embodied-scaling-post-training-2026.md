---
title: "亮源新创 LightNav-0 / Light REACT：把大模型的三段范式搬进具身"
created: 2026-09-12
updated: 2026-09-14
type: entity
tags: [embodied-ai, vision-language-navigation, post-training, online-rl, preference-alignment, resilience, whole-body-in-context-learning, rvq, qwen3-vl, liangyuan, scaling]
sources: [raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026, raw/articles/liangyuan-lightparkour-lightnav0-light-react-qbitai-2026]
confidence: 0.65
---

# 亮源新创 LightNav-0 / Light REACT

## 核心论点：具身智能也在重走「预训练—对齐—部署」三步

文章把大模型五年验证出的三段范式——**规模化预训练 → 规模化对齐 → 规模化部署**（能力、可用性、进化速度分别来自三个不同环节）——直接翻译成具身版本：先在仿真里规模化合成出泛化能力，再用对齐把能力变成可用性，最后靠真实部署回流经验驱动持续进化。亮源新创 CEO 姜旭（曾在 OpenAI 从事 RLHF 研究）的创业信条是「通用比专用重要」：先全场景泛化，再走向全场景精确，这在「先在单一场景做精再谈泛化」仍为主流的具身赛道算是一条反共识路线。^[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026.md]

## LightNav-0：零样本跨本体的视觉语言导航（9 月 1 日开源）

LightNav-0 解决「认路」：以开源视觉语言模型为基座，**不添加任何导航专用模块**，同一套模型权重可零样本部署到人形、四足、轮式乃至飞行机器人上，无需采新数据、不做微调即可听懂自然语言指令自行找路。团队在 10 项公开仿真评测中取得领先，且仅用单目 RGB、不依赖深度与里程计，在动态跟踪基准上的成绩超过使用全景与多相机的系统。^[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026.md]

工程细节上有三处值得记录的做法：^[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026.md]

- **扩展词表而非专用头**：基座为 Qwen3-VL-4B-Instruct，团队不为导航增加专用输出头，而是用**双通道指点（dual-channel pointing）token** 在图像坐标系中表达与任务、场景、本体无关的空间意图。
- **RVQ 动作分词器**：残差向量量化把空间意图落成 10 步 SE(2) 轨迹——一个粗码本加两级残差码本，分辨率依次约 0.9 米、7 厘米、4 厘米，全部经由基座原生的自回归语言头解码。等价于把「连续控制」表述成语言模型的原生输出形式。
- **遗忘曲线式历史压缩**：为避免历史观测撑爆上下文，按人类记忆的遗忘曲线分配注意力——采样率随帧龄指数衰减、空间池化步长指数增长，支持 256K 至 1M 三档像素预算。

数据侧没有走遥操作采集，而是把 2000 余个真实场景转化为仿真资产，在其中合成 4000 余小时训练数据：真实世界负责定义数据分布边界，仿真负责在边界内规模化合成，具身后训练由此跨过冷启动门槛。后训练沿「ER 中期训练 — 具身 SFT — 在线 RL」三阶段推进，发布时附带面向部署的评测集：210 个真实室内外场景、1097 个 episode。^[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026.md]

## Light REACT：把「韧性」做进全身控制

Light REACT（REsilient humAnoid ConTrol）解决「摔不垮」，其目标是把韧性作为规模化部署的最后一公里——机器人先得摔不垮，部署产生的经验流才不会中断，数据飞轮才转得起来。文章把韧性拆成一座四层金字塔：身体完好时按指令行走；被推搡、被撞倒后自己爬起来；部分关节断电或锁死时换一种步态（跛行、单足跳）继续移动；伤到双足行走物理上不可行时转入爬行，保住「移动能力」本身而非某一种走法。^[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026.md]

配方分三步：先在动力失效、关节锁死、膝折叠约束三类伤损域上训练 6 个域专用教师（每域一个「恢复—行走」教师与一个「爬行」教师）；随后以 DAgger 式多教师蒸馏并入单一学生策略——学生只接收速度指令与本体感知，**伤损识别被整体推迟为对交互历史的上下文推断**；最后以偏好强化学习对齐「可直立则直立」。承载这一切的是支持全身上下文学习（Whole-Body In-Context Learning）的 Transformer 单模型：不依赖故障标签、不做模型切换、不做部署期权重更新。^[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026.md]

## 可迁移的工程模式

剥掉公司叙事，这篇的复用价值集中在四个模式上：其一，**把连续控制重新表述为基座模型的原生输出**（扩展词表 + 动作分词）比外挂专用头更利于跨本体零样本迁移，可与 [[entities/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026|跨本体基因组继承]] 的「本体无关表征」思路对照；其二，**按遗忘曲线压缩历史观测**是长程上下文的通用省算力手段，与 Agent 侧的上下文压缩路线同源；其三，**多教师蒸馏 + 推理期上下文推断伤损**，把「状态识别」从输入侧移到历史侧，避免部署期模型切换；其四，**把失败与损伤工况当作一等训练分布**，与 [[entities/ropedia-homie-gen2-experience-scaling-law-embodied-ai-2026|经验 Scaling Law]] 的「真实交互经验驱动进化」互补。^[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026.md]

局限同样明确：文章由厂商投放（「机器之心发布」），核心数字（10 项仿真领先、210 场景评测）均来自团队自报技术报告，尚无独立第三方复现的横向评测；「十天两连发」的节奏也把发布营销与工程验证混在了一起。评估时应把它的方法模式视为可借鉴的工程做法，而把性能声明视为待验证项。^[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026.md]

相关工作见 [[concepts/embodied-intelligence-frontier|具身智能前沿]]、[[concepts/world-models|世界模型]]、[[entities/urbanground-embodied-navigation-benchmark-2026|UrbanGround 导航基准]]、[[entities/lingbot-vla-2-60000h-open-source-vla|LingBot-VLA 2（6 万小时开源 VLA）]]、[[entities/embodied-native-llm-embodied-intelligence-new-stage|具身原生 LLM]]、[[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 与可验证强化学习]]。

→ [[raw/articles/lightnav-0-light-react-liangyuan-embodied-scaling-post-training-2026|原文存档]]

## 第 2 来源 — 量子位（2026-09-13）：三项技术并列，补上 LightParkour

同一批发布被量子位以「三项技术」而非「两次发布」组织，与首发来源（机器之心）互补之处集中在第三项技术 LightParkour，以及把 Scaling 的定义从「参数量/数据量」改写成「能力能否持续规模化扩展」的表述。v×c 约 42（同题跨号重发，主体重叠高）。^[raw/articles/liangyuan-lightparkour-lightnav0-light-react-qbitai-2026.md]

- **LightParkour（维基此前未覆盖）**：从简短的人类动作片段出发，通过物理仿真与课程学习扩展复杂接触技能，对应三行发布中的「跑酷」方向——即机器人跨越障碍的接触密集运动技能获取。^[raw/articles/liangyuan-lightparkour-lightnav0-light-react-qbitai-2026.md]
- 三项技术被明确映射为大模型三段范式：LightParkour 偏能力构建、LightNav-0 对应「对齐」、Light REACT 对应「部署」，三项共同指向「训练—对齐—真实部署」的学习闭环。^[raw/articles/liangyuan-lightparkour-lightnav0-light-react-qbitai-2026.md]
- **Scaling 的重新定义**：此前阶段问的是机器人「会多少技能」，下一阶段问的是这些能力能否经历更多环境、覆盖更多任务、迁移到更多机器人本体，并在进入物理世界后继续适应训练阶段未见过的状态——把评价标准从技能清单转向可持续扩展性。^[raw/articles/liangyuan-lightparkour-lightnav0-light-react-qbitai-2026.md]
- 数据侧口径一致：2000+ 互联网来源真实场景 → 可反复使用的仿真环境，合成 4000+ 小时视觉/语言/动作经验，用于通用导航后训练；发布时附带 210 个真实室内外场景、1097 个 episode 的部署评测集。^[raw/articles/liangyuan-lightparkour-lightnav0-light-react-qbitai-2026.md]

→ [[raw/articles/liangyuan-lightparkour-lightnav0-light-react-qbitai-2026|第 2 来源原文]]
