---
title: "GKA（Gated KalmaNet）：可变测试时计算的推理架构"
created: 2026-10-14
updated: 2026-10-14
type: entity
tags: [test-time-compute, inference, architecture, linear-attention]
sources: [raw/articles/gka-variable-test-time-compute-amazon-science-2026]
confidence: 0.7
---

# GKA：可变测试时计算的推理架构

Amazon Science 介绍（不重训练即改变架构）：Gated KalmaNet (GKA) 拥有 Mamba-2 与 GDN 都不具备的性质——**可变测试时计算（variable test-time computation）**。Mamba-2/GDN 对每个 token 的计算预算固定；GKA 把推理建模为迭代回归问题，**迭代次数可在服务时调整、无需重训练** ^[raw/articles/gka-variable-test-time-compute-amazon-science-2026.md]。

## 速度-精度前沿

实测：以 10 次迭代替代常规 30 次，AIME 2025 精度仅牺牲约 1%，换来显著推理提速——相当于给推理架构装上运行时「速度 vs 精度」旋钮。这为 [[entities/harness-evolution-vs-test-time-scaling-2026]] 讨论的 test-time scaling 之争提供了第三条路：不靠外部采样预算（Best-of-N），而是架构内生的可调计算深度。与 [[entities/harness-evolution-vs-test-time-scaling-2026|harness 演化 vs test-time scaling]]之争的区别：GKA 不更新权重，只调迭代深度。

## 深度分析

### 把推理变成迭代回归：可变计算为何成立

GKA 的核心思路是把序列推理中的状态更新建模为迭代回归问题（Gated KalmaNet，带门控的 Kalman 式结构）。Mamba-2 与 GDN 这类固定预算架构对每个 token 都消耗相同的计算量，与其内容难易完全无关——一个平凡的续写 token 和一个需要深度推理的 token，开销一模一样；GKA 则把「解得有多准」与「迭代了多少次」解耦，使迭代次数成为服务时可调的旋钮。这意味着精度不再是权重训练时一次性锁定的承诺，而是可以随算力预算伸缩的连续量——「不重训练即改变模型架构」的关键正在于此：权重不动，变的只是求解深度。

从优化的视角看，这相当于把推理显式表达为一个可迭代逼近的估计问题，每次迭代都是对当前状态估计的一次修正。这一结构带来的自由度是二元的架构族（一次前向即完成计算）所不具备的：前者可以中途停止，后者没有「中途」可言。

### 单个模型服务整条速度-精度前沿

原文最有分量的工程结论：一次训练出的模型可以覆盖速度-精度前沿上的多个点，不同几何形状标记前沿上不同的迭代次数。传统做法是蒸馏或分别训练出 small/mid/large 多个模型变体，各自维护训练、评估、部署与版本管线，任何一次改进都要同步到所有变体；GKA 用一个 checkpoint 加不同迭代次数即可提供不同延迟档位（latency tiers），变体维护成本从多套模型降为同一套模型的多档配置。这也把测试时计算从「外部采样预算」（Best-of-N、self-consistency）内化为「架构内生的深度调节」，为 [[entities/harness-evolution-vs-test-time-scaling-2026]] 中 harness 演化与 test-time scaling 之争提供了第三条路径：不靠外部采样堆预算，而是让单个模型自身具备可伸缩的计算深度。

### 10 次迭代换 10% 提速：边际收益递减的帕累托点

实测数据点：在 AIME 2025 上以 10 次迭代替代常规 30 次，精度仅牺牲约 1%，解码反而快 10%。这个不对称比例说明迭代回归的误差收敛是非线性的——前几次迭代吃掉绝大部分误差，后续迭代只是边际收益递减的「精修」。因此曲线上的最佳工作点往往远离满配迭代数，默认配置（30 次）很可能不是帕累托最优。对比 [[concepts/attention-mechanism]] 中 softmax 全量计算的固定成本，这种可截断的迭代结构天然适合在延迟敏感场景做预算裁剪。原文还给出多组上下文长度下的前沿曲线，说明该性质在不同上下文长度下均成立，而非某个特定长度的巧合。

### 「智能 = 按难度分配努力」的架构级体现

Amazon 把 GKA 定位为一条既有原则的架构化身：intelligence is the ability to allocate effort commensurate with difficulty。此前的实践多在系统层（agent 循环、多步反思、动态路由）实现按难度分配努力；GKA 把它下沉到模型架构内部——部署方按 SLA 调整迭代数，demanding speed 的部署少跑迭代，demanding accuracy 的部署多跑迭代。但要注意边界：GKA 目前做到的是「服务时全局可调」，即整个部署统一一个迭代数，而非「逐 token 按内容自适应」；真正的 per-token 自适应计算（难 token 多算、简单 token 少算）仍是开放方向，这一点也与 [[concepts/transformer-architecture]] 中计算量随上下文规模增长的传统权衡形成不同的设计轴。可以预期，从「全局旋钮」演化到「自动旋钮」会是这条路线的下一步。

## 实践启示

1. 推理服务不必为每个延迟档位维护独立的模型变体：优先考虑测试时计算可调的架构，用一个 checkpoint 覆盖 fast/balanced/quality 多档服务，省掉多套蒸馏模型的评估与运维成本。
2. 上线前先在自己的任务上扫一遍「迭代次数-精度」曲线，找到帕累托拐点（GKA 的参考点是 10/30 迭代只损失约 1% 精度），把拐点而非满配迭代数作为默认服务配置；满配很可能是白白浪费的延迟。
3. 实时对话等 demanding-speed 场景与数学推理、代码评审等 demanding-accuracy 场景可共用同一模型，只调迭代预算，从而简化模型治理、评估与版本管理，避免变体漂移。
4. 选型线性注意力/SSM 架构时，把「是否支持可变测试时计算」列为评估维度：Mamba-2/GDN 每 token 计算预算固定，GKA 的迭代回归结构是差异化能力，对延迟分层服务的团队价值最大。
5. 区分两类测试时计算的预算来源与成本模型：Best-of-N 等采样预算花在「多猜几次」，GKA 的迭代预算花在「单次解得更准」；后者不增加请求次数与 KV 复制，延迟收益直接体现在解码时间上。
6. 跟踪逐 token 自适应计算的后续进展：GKA 目前是服务时全局调参，若出现按输入难度自适应分配迭代次数的机制，同等精度下的延迟还能进一步压缩，值得持续关注该架构族的演进。

## 关联
- → [[raw/articles/gka-variable-test-time-compute-amazon-science-2026|原文存档]]
