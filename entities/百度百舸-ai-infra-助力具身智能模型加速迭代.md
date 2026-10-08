---

title: "百度百舸 AI Infra 助力具身智能模型加速迭代"
type: entity
created: 2026-07-04
updated: 2026-10-08
tags: [wechat, ai]
rating: v7c7
sources:
  - raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代
  - raw/articles/baidu-cosmos3-training-optimization-domestic-gpu
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 百度百舸 AI Infra 助力具身智能模型加速迭代

**来源**: 百度Geek说

**发布日期**: 2026-02-02^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]


**原文链接**: https://mp.weixin.qq.com/s/96cmhuymVdUUBDpG_C4p4Q ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

---

。

点击蓝字，关注我们

本文整理自
百度智能云主任架构师应茹在文心 Moment 大会 2026·^[raw/articles/baidu-cosmos3-training-optimization-domestic-gpu.md]

上海站多硬件协同分论坛的演讲。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

我今天分享的主题是：在具身智能行业飞速发展的背景下，百度百舸·AI 计算平台看到了哪些趋势和新需求，以及作为技术赋能者，如何助力具身智能模型的加速迭代。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

2025 年以来，我们都能感受到自身处于具身智能的热潮之中。 在云端具身智能领域 ，听到最多的一个词是 VLA（Vision-Language-Action Model），它以视觉与自然语言解析 作为输入，生成 Action 序列，成为一种热门的建模范式。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

从顶会论文统计数据或权威论文发布平台来看，以 VLA 为主题的文献数量在 2025 年呈现了十几到二十倍的爆炸性增长。深入这些文献内容可以发现，此前主流的机器人操作或机械臂评测任务集，平均成功率也已达到 95% 以上。同时，在设计 VLA 模型结构时，架构设计也逐步形成了一些共识。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

我们看到，云端的具身智能开发者与研究者，开始将研究重点转向如何对模型参数量和算力训练规模进行 Scaling，使操作任务变得更长程、更复杂，并关注模型如何真正落地到产业端，例如关注推理速率、考量鲁棒性，能否在长程任务中实现稳定、持续、高成功率的操作。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

在这样的变化背景下，具身行业对 AI Infra 平台、对百度百舸提出了一些新的要求。^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]


首先， 简要介绍一下 具身智能模型基于云平台的典型开发流程与工作流 Workflow 。通常 ，AI Infra 或云厂商会提供包含几个层次的 Infra 底座： ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

首先是底层 IaaS，包含高性能算力，辅以机间高速互联的高性能网络，以及分布式高性能存储能力。^[raw/articles/baidu-cosmos3-training-optimization-domestic-gpu.md]


其上提供成熟的云原生分布式调度框架，将模型的训练和推理任务高效调度和部署到机器上。^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]


具体模型如何运行？训练和推理框架会根据事先定义好的模型结构，调用底层 GPU 或加速芯片开放的算子接口来运行模型。通常还会通过切分的并行技术在多卡或多机之间进行加速运行，这主要由第三层训推加速框架负责。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

具身智能的业务运行在此之上，核心流程主要涉及几个环节：^[raw/articles/baidu-cosmos3-training-optimization-domestic-gpu.md]


首先是训练数据准备。数据来源可以有多种：预置开源热门数据集，客户可直接使用。除了主流的 20 多种开源具身数据集，近期百度百舸也率先针对预训练场景集成了简智开源的 RealOmni 无本体数据集。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

客户也可在云端部署仿真环境，通过键盘遥操作收集轨迹数据，或使用世界模型生成数据，还可将真机数据上传至云上，使用数据增强模型进行扩增。这些数据存储在云端的高性能存储集群中，可便捷地对接到训练集群或开发机，进入训练环节。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

训练环节涉及具身大脑、运动小脑及世界模型的训练。百度百舸全面适配了 RDT、GR00T N1.5、π0.5 等一系列模型，并对开源模型的训练吞吐做了加速优化。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

训练到一定阶段后进入模型效果评估环节，客户可使用云端预置的仿真评测环境进行模型鲁棒性、成功率的评测。百度百舸在此集成了 Isaac、Maniskill3、RoboTwin2 等主流仿真环境及典型任务集，训练完的模型 Checkpoint 可直接对接到仿真环境部署评估。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

这便是当前具身智能在云端的主流开发工作流。 针对上述环节，我们将从 AI In^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]


^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

→ [[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代|原文存档]] ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

## 2026-07-08 补充：Cosmos 3 全链路训练优化 ^[raw/articles/baidu-cosmos3-training-optimization-domestic-gpu.md]

百度百舸团队在国产主流 GPU（无 NVLink）上对 Cosmos3-Nano-Policy-DROID 做全链路优化，效果超越官方 GB200 基准： ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

| 优化项 | 技术 | 效果 |
|--------|------|------|
| 任务启动 | Parquet 列裁剪 + 数据路径重构 | 37.2min→25s，内存 1,734GB→46GB |
| I/O 瓶颈 | ColorJitter CPU→GPU 迁移（占 78.5% 耗时） | 吞吐 +50% |
| 编译优化 | torch.compile 禁用 mix-order reduction | 吞吐 +28.6% |
| 显存利用 | 分层 Activation Checkpointing（layer-wise AC） | 吞吐 +3.1% |
| 多机扩展 | 弹性 RDMA + HSDP | 12 节点 98.3% 扩展效率 |

**MFU 0.42**，超过 Cosmos 3 官方论文 GB200 基准（0.23-0.3）。所有优化均为无损精度。 ^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

---
## 深度分析

### 双系统分层架构决定了两条根本不同的 Infra 需求

演讲最有信息量的部分是把 VLA 的「大脑（System 2）/小脑（System 1）」共识拆成两条实现路径并给出各自的 Infra 画像：端云协同学派把大脑放云端（参数量可超 200B、MoE 结构），需要的是极致吞吐的多机多卡并行与通信开销优化；单体模型学派把双系统压进 10B 以下端侧模型，需要的是与模型结构解耦、兼容 Hugging Face、敏捷试错的中小规模训练框架。两条路径对平台的诉求几乎正交——这解释了为什么百舸同时维护「大 MoE 加速」与「结构解耦敏捷框架」两条产品线。世界模型联合训练则是第三条平行路线：预测未来帧比 Action 序列有更密集的监督信号，数据利用效率更高，百舸为此专门优化了输入序列的多机多卡并行。^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

### 强化学习是具身 Infra 与 LLM Infra 真正分岔的地方

LLM 领域沉淀的 RLHF 工具链在具身场景会遇到三个新问题：多模态输入输出数据量大导致单控制节点数据预处理与分发成为瓶颈；多模态序列长度差异大导致多机负载不均；VLA 训练必须与仿真环境实时交互（生成动作→环境更新→新观察），这是现有开源 RL 框架缺失的环节。这个分岔说明具身智能不能简单复用文本 LLM 的 RL 基础设施，仿真器接入是必须自建的能力。^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

### Cosmos 3 实战：国产 GPU 无 NVLink 反超 GB200 基准的方法论

2026-07-08 补充的全链路优化把「工程优化能吃掉硬件代差」量化到了极点：在无 NVLink 的国产 GPU 上 MFU 0.42，超过 Cosmos 3 官方 1024×GB200 基准（0.23-0.3）。五步优化的顺序本身就是教科书——先用 Profiling 定位（ColorJitter 占 CPU 耗时 78.5%）再动手，任务启动从 37.2min OOM 到 25s 靠 Parquet 列裁剪消除冗余读取；I/O 迁 GPU +50%；torch.compile 禁用 mix-order reduction 适配国产 GPU 显存限制 +28.6%；分层 AC 精细化控制 +3.1%；弹性 RDMA + HSDP 让 12 节点扩展效率达 98.3%。关键约束是所有优化均无损精度（Loss 曲线与官方 Baseline 一致），说明这是纯粹的工程空间而非精度换速度。^[raw/articles/baidu-cosmos3-training-optimization-domestic-gpu.md]

### GEN-0 的 Scaling Law 验证是平台叙事的支点

演讲引用 GEN-0 基于 27 万小时真机数据预训练的结果，验证具身基础模型同样存在显著 Scaling Law——这是所有云厂商押注具身 Infra 的前提假设：如果 scaling 不成立，算力平台就没有复利空间。百舸的全景（昆仑芯超节点、MoE 优化 RDMA、开源模型加速版、Isaac CPU 敏感型任务调优）本质是对这个假设的下注。^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]

## 实践启示

1. 评估具身训练平台时先确认自己的架构路线（端云协同 vs 单体端侧 vs 世界模型联合训练），三者对并行策略、框架解耦性、序列并行的需求完全不同。^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]
2. 做 VLA 强化学习前先检查 RL 框架是否支持仿真器在线交互；缺这一环就要自建环境接口，不能套用 LLM RLHF 工具链。^[raw/articles/百度百舸-ai-infra-助力具身智能模型加速迭代.md]
3. 训练吞吐优化永远从 Profiling 开始：Cosmos 3 案例中 78.5% 的耗时在 CPU 端 ColorJitter，盲目堆 DataLoader Worker 无效，迁移单个算子就拿到 +50%。^[raw/articles/baidu-cosmos3-training-optimization-domestic-gpu.md]
4. 数据加载 OOM 优先查列存储的冗余字段读取：Parquet 列裁剪 + 拷贝路径重构把 1734GB 内存需求压到 46GB，比加机器便宜 89 倍。^[raw/articles/baidu-cosmos3-training-optimization-domestic-gpu.md]
5. 国产 GPU 适配编译器的正确姿势是按底层资源规格关策略（如 mix-order reduction），而非硬调超参。^[raw/articles/baidu-cosmos3-training-optimization-domestic-gpu.md]

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

