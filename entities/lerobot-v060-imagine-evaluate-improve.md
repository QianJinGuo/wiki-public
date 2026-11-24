---
title: "LeRobot v0.6.0 — Imagine, Evaluate, Improve"
created: 2026-08-14
updated: 2026-11-14
type: entity
tags: [lerobot, robotics, embodied-ai, world-model, vla, reward-model, huggingface, open-source]
sources: [raw/articles/lerobot-v060-imagine-evaluate-improve]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LeRobot v0.6.0 — Imagine, Evaluate, Improve

Hugging Face LeRobot 框架 2026 年发布 v0.6.0，主题是**闭合机器人学习回路**：策略先想象未来再行动（world model policies）、奖励模型判断机器人是否成功、部署 CLI 把失败转化为训练数据、六个新仿真基准统一度量。^[raw/articles/lerobot-v060-imagine-evaluate-improve.md]

## 核心新增

- **World model policies**：VLA-JEPA、FastWAM、LingBot-VA 三类学会"想象未来"的策略
- **新 VLA 家族**：GR00T N1.7、MolmoAct2、EO-1、EVO1、Multitask DiT
- **Reward models API**：Robometer、TOPReward，判断任务成功与否
- **`lerobot-eval`**：六个新仿真基准统一度量入口
- **`lerobot-rollout` CLI**：DAgger 式人在回路纠错，失败样本直接沉淀为训练数据
- **训练基建**：FSDP 训练、HF Jobs 云训练
- **数据管线**：深度图支持、VLM 自动语言标注、自定义视频编码、数据加载提速 2x

## 深度分析

### 想象的三种成本结构：world model 放在哪一层

v0.6.0 的三个 world model policies 其实是对"想象放在哪一层"这个问题的三种回答^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:54-56]。VLA-JEPA 把想象放在训练期：一个 JEPA 世界模型在训练中被迫从模型自身的动作预测即将到来的帧，推理时整个世界模型被丢弃——相当于"世界模型监督免费送"，以零额外推理成本拿到表征级监督，这与 [[entities/yann-lecun-jepa-world-model|Yann LeCun 的 JEPA 世界模型]] 的表征预测思想一脉相承^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:58-69]。LingBot-VA 把想象留在推理期：自回归 video-action 模型逐 chunk 同时预测未来视频与动作，并回灌真实观测让想象保持贴地，还支持把"机器人想象的视频"存下来与实际发生的事情对比（单张 24–32 GB GPU 即可推理），对应 wiki 已收录的 [[entities/lingbot-va-20-embodied-video-action-pretrain-ant-lingbo-2026|LingBot-VA 实体]]^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:71-75]。FastWAM 则直接质疑测试期想象的必要性：约 5B 的视频生成专家与紧凑动作专家装进同一个网络，让模型"学会梦见自己的 rollout"，但推理时完全跳过做梦、直接去噪动作块^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:77-79]。三条路线合起来是一条成本—可靠性光谱：训练期想象最便宜但只提供间接监督；推理期想象最直观、可审计，但最贵；跳过想象则是押注动作专家已在训练中内化了想象能力。选型时应把"想象"当作用算力购买的可验证性，而不是默认它一定提升成功率。

### 成功判据的缺口：预训练奖励模型 vs 零样本 VLM

文章明确把成功检测与进度估计称为机器人学习回路中"缺失的拼图"，并用统一的 `lerobot.rewards` API 把四种奖励模型收编进一个接口^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:119-121]。其中两条新增路线代表了两种解法。Robometer 走预训练通用路线：基于 Qwen3-VL-4B，用超过一百万条机器人轨迹的轨迹对比训练而成，指向任意 LeRobot 数据集就能从原始视频加语言指令给任务进度和成功与否打分，无需任务特定训练^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:123-127]；它与 [[entities/discretizing-reward-models|奖励模型离散化]] 关心的是同一层问题——奖励信号本身的可靠性。TOPReward 走零样本路线：完全不训练任何奖励权重，直接包一层现成 VLM，读它在"轨迹视频 + 任务指令"条件下输出 "True" 这个 token 的 log-probability，于是任何够强的 VLM 立刻变成一个奖励函数^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:129-131]。两者都附带标注脚本，把逐帧进度曲线写回数据集，供 reward-aware behavior cloning、数据质量检查和进度叠加视频使用^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:133]。这一步之所以是回路闭合的前置条件：没有可靠的成功判据，后面的 DAgger 纠正与 RA-BC 都无法筛选"哪些样本值得学"。

### 六个新基准的分工：从鲁棒性到记忆

v0.5.0 把 LeRobot 定位成 VLA 评估枢纽，v0.6.0 用 `lerobot-eval` 这一个 CLI 把六个新仿真基准收编进来，与既有基准合计九个家族，每个都配文档页、Docker 镜像和 CI 冒烟过的 SmolVLA 基线^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:175-179]。关键在于这六个基准不是重复建设而是各有分工：LIBERO-plus 用约一万个跨七个维度的扰动变体（光照、相机视角、改写指令等）压测 VLA，回答"策略何时会崩"；RoboTwin 2.0 提供 50 个带强域随机化的双臂操作任务并附带十万级可直接训练的轨迹；RoboCasa365 以 365 个厨房任务、2500 个程序化厨房铺开最大任务面；RoboCerebra 专测长程行为，每条 episode 串联 3 到 6 个子目标并带语言锚定的中间指令；RoboMME 是记忆考试（数重复次数、追踪被隐藏物体、模仿演示流程）；VLABench 考操作中的知识与推理（从物理问题到端到端煮咖啡）^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:181-186]。这份清单读起来就是一张能力探针表：鲁棒性、双臂协调、任务规模、长程规划、记忆、知识推理——恰好是检验 world model policy 是否真的"想得更好"所需的六个维度，也和 [[concepts/agent-evaluation-benchmark-frameworks|评估基准框架]] 中"基准即基础设施"的打包模式一致。评估本身也提速：并行评估默认改为异步向量化环境，基准测试最高快 2 倍^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:198]。

### 部署即采集：DAgger 飞轮的 CLI 化

部署过去是对 `lerobot-record` 的 hack，v0.6.0 让它有了独立 CLI `lerobot-rollout`，带五种可插拔策略^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:204-206]。其中 `dagger` 策略是回路的关键一环：人看着策略跑，出错瞬间按键（或踩 USB 踏板）用领导臂接管记录纠正，纠正前领导臂会被驱动到从动臂位姿、交接无抖动，每一帧纠正都带 `intervention` 标记，产出的数据集直接可用于下一次微调——文章把它总结为 "deploy, collect corrections, fine-tune, repeat"，机器人学习飞轮从此是一个 CLI flag^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:208-222]。数据侧则在同步补齐回路的其他环节：深度图端到端支持（毫米采集、12-bit 深度视频流压缩、训练时还原物理单位，覆盖 SO-100/101 到 Unitree G1 等本体）^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:151-153]；`lerobot-annotate` 用 VLM 自动观看 episode 填充富语言标注（时间戳子任务、计划、纠正、语音、逐相机 VQA），再经 YAML recipe 渲染成聊天式训练消息——正是下一代长程对话式策略要吃的数据形态^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:157-169]；数据加载最高快 2 倍，大数据集子集加载从 275 秒降到 0.06 秒，且采样确定性可续跑^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:171-173]；训练侧 FSDP 分片让参数/梯度/优化器状态跨 GPU 切分并回收到单文件 safetensors，同一 `lerobot-train` 命令加一个 flag 就能上 HF Jobs 从 T4 到 8x H200 按量付费云训练^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:226-242]。至此采集、标注、判成功、纠错、训练、评估六环全部落在同一框架内成为标准件——这种框架级闭环正是 [[concepts/embodied-intelligence-frontier|具身智能前沿]] 反复出现的"规模 + 飞轮"叙事的具体化，而富语言标注也正是 [[entities/simplememvla-native-visual-context-robot-memory-2026|SimpleMemVLA]] 等记忆向策略依赖的数据形态。

## 实践启示

1. **按推理预算选想象路线，别为想象付双份钱**：推理预算紧张、只需要更好的监督信号时选 VLA-JEPA 式训练期想象；需要可视化审计"策略在想什么"且能接受 24–32 GB GPU 时选 LingBot-VA 式在线想象；追求最低推理延迟则直接用 FastWAM 式跳过想象^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:56-79]。
2. **奖励模型先零样本验证、再谈定制**：给任务加成功判据时，先用 TOPReward 的 VLM log-prob 路线零样本试出信号是否有区分度，确认有用后再考虑 Robometer 式预训练模型或任务微调，避免一开始就为每个任务训练奖励分类器^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:121-133]。
3. **报分数之前先压扰动**：平均分掩盖泛化假象，先用 LIBERO-plus 这类扰动基准找到策略在光照、视角、指令改写下的崩溃点，再谈基准排名^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:181]。
4. **把部署默认当成数据管线**：真实部署应默认开启 DAgger 式采集——即便多数时间无人接管，`intervention` 标记也让纠正样本天然可筛选，接管成本只是按一次键^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:206-222]。
5. **长程与记忆要单独出题**：不要用单步任务分数推断长程能力；用 RoboCerebra 的 3–6 子目标链和 RoboMME 的记忆套件这类专门探针分别测规划与记忆^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:184-185]。
6. **小模型路线已经够用**：EVO1 把策略压进 0.77B 参数、MolmoAct2 在 bf16 下推理只需约 12 GB——消费级 GPU 上的微调加实时推理已足以驱动真实任务，VLA 竞赛不止"更大"一条路^[raw/articles/lerobot-v060-imagine-evaluate-improve.md:92-103,115-117]。

## 与 wiki 内相关主题的关系

与 机器人具身智能 领域直接相关：world model policies 是 [[entities/amap-abot-earth-0.5-3d-native-world-model|3D 原生世界模型]] 一族的策略侧延伸；reward models 方向与 [[entities/embodied-ai-data-market-landscape-97-players-44-billion-2026|具身数据市场]] 的评估环节互补。LeRobot 定位为开放机器人学习栈，与商业 VLA 方案形成开源对照。

→ [[raw/articles/lerobot-v060-imagine-evaluate-improve|原文存档]]
