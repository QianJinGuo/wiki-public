---
title: "EMCES (ICML 2026) — Episodic Memory-Guided Controllable Experience Synthesis for Reinforcement Learning"
created: 2026-07-02
updated: 2026-11-10
type: entity
tags: [icml-2026, reinforcement-learning, episodic-memory, diffusion-model, sample-efficiency, experience-synthesis, zhejiang-scitech-university, nanjing-university]
sources: [raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# EMCES (ICML 2026) — Episodic Memory-Guided Controllable Experience Synthesis for RL

## 概述

EMCES (Episodic Memory-guided Controllable Experience Synthesis) 是浙江理工大学马啸讲师与南京大学李武军教授课题组联合提出的高效强化学习样本合成方法，已被 ICML 2026 录用。核心贡献是将**情景记忆机制 (Episodic Memory)** 引入可控扩散模型，利用记忆中的历史最优经验指导扩散模型优先合成对策略学习更有价值的样本，从而显著提升下游强化学习算法的表现。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md]

## 动机：现有样本合成方法的局限

强化学习在真实世界中面临的核心难题是高质量样本的获取成本高昂且伴随风险。近年来基于扩散模型的样本增强方法（代表方法为 SynthER）通过合成高保真样本实现了训练数据扩充。然而，实验表明（Hopper medium-expert 环境，D4RL 基准）：合成样本虽然符合真实环境动态，但**未必最有助于智能体的策略学习**。只有当合成样本集的规模远大于原始样本集时，合成样本才有可能充分覆盖高质量区域并获得策略性能提升。根本原因在于：现有方法的样本合成过程**缺乏有效的可控机制**，难以优先合成对策略学习更有价值的高质量样本。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md]

## 方法架构

EMCES 包含三个关键组件：

### 基于情景记忆的可控扩散模型

引入可控扩散模型，将期望输出设定为状态转移（样本）。模型通过优化去噪目标学习数据分布，使用状态-动作价值函数构造条件信号。传统状态-动作价值函数依赖神经网络额外训练且不稳定。EMCES 创新地引入**情景记忆机制**来估计价值函数——由于情景记忆具有非参数特性，可在无需额外模型训练的情况下实现稳定的价值估计。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md]

### 基于情景记忆时序差分误差的优先条件采样策略

提出 EMTD-误差 (Episodic Memory Temporal Difference Error) 作为衡量样本对策略改进重要性的指标。EMTD-误差的大小表示基于下一状态的价值估计与当前状态的历史最优折扣回报之间的偏差。进一步提出 Softmax 优先采样策略，引导扩散模型优先合成具有较大 EMTD-误差的高价值样本，同时控制采样程度以保持多样性。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md]

### 基于哈希状态表示的情景记忆

为情景记忆设计了一种**基于 Learning-to-Hash 的状态表示方法**（采用李武军教授提出的 IsoHash 方法）。将原始高维状态编码为紧凑的二进制编码，在信息量上比已有方法在存储开销上降低约 8000 倍，时间开销降低 25.5 倍。哈希编码从数据分布中学习得来，能更好地对齐状态空间底层结构，并通过隐式合并多条轨迹构建更高质量的情景记忆。情景记忆使用 KD-树实现存储与检索。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md]

## 深度分析

### 从"合成保真"到"合成价值"：样本增强目标函数的改写

EMCES 开篇的诊断实验对 SynthER 类方法给出了最有力的质疑：在 Hopper medium-expert 环境（D4RL 基准，原始样本约 200 万条）上用合成样本训练 TD3+BC，只有当合成集规模从 10 万扩大到 500 万、远超原始集时，策略性能才开始提升。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md] 这说明高保真合成只是必要条件：均匀逼近数据分布的生成器会把大部分算力预算花在低价值区域。EMCES 的应对是给可控扩散模型注入"价值先验"——条件由紧凑状态编码与情景记忆估计的历史最优折扣回报共同构成，把优化目标从"像数据"改写为"对策略改进有用"。这与 [[entities/diffusion-model-consistency-framework-2026-survey|扩散模型一致性综述 (2026)]] 关注的维度互补：后者解决"如何生成得符合约束"，EMCES 解决"应该生成什么"。

### 非参数价值估计：用记忆替代价值网络的权衡

可控生成的条件信号传统上依赖状态-动作价值函数，而神经网络估计需要额外训练且不稳定。EMCES 改用情景记忆（KD-树检索历史最优折扣回报）做非参数价值估计：零训练成本、随数据即时更新、不受价值网络训练波动影响。代价是检索质量完全取决于记忆覆盖度与状态表示质量——训练早期记忆稀疏时条件信号偏弱，这正是论文把哈希表示与"隐式合并多条轨迹"作为配套设计的原因之一。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md] 按 [[concepts/episodic-vs-semantic-memory-agent|Agent 情节记忆 vs 语义记忆]] 的类型学，EMCES 属于情节记忆路线：以实例级检索直接访问历史经验，而非把经验压缩成语义化知识。

### EMTD-误差：把优先级机制从经验回放前移到生成空间

优先经验回放的经典做法是用 TD-误差给已存储样本加权；EMCES 的 EMTD-误差（下一状态价值估计与当前状态历史最优折扣回报的偏差）则前移一步——用 Softmax 优先条件采样控制扩散模型"生成什么"，而非挑选已有样本。优先化强度取下界时退化为均匀采样（近似 SynthER 行为），调大则聚焦高 EMTD-误差区域、以多样性为代价，这是探索-利用权衡在生成层的重现。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md] 高 EMTD-误差意味着"记忆认为该转移可能带来更高回报"，优先合成它们相当于主动制造信息量最大的训练数据，而不是被动等待环境中偶然出现。

### 哈希表示是记忆规模化的真瓶颈

对照实验揭示了一个易被忽视的优先级：索引结构沿用团队前期 KD-树实现不变，仅把状态表示从随机投影换成基于 IsoHash 的学习式二进制编码，就换来约 8000 倍存储、25.5 倍时间开销的降低，且不损失归一化分数。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md] 原因有二：哈希编码从数据分布学习而来，与状态空间底层结构对齐，减少无关状态混淆；相似状态共享编码，隐式合并多条轨迹，反而提升记忆构造质量。实验还显示不同状态表示下 EMCES 均优于 SynthER，说明收益来自整体框架而非哈希本身。记忆系统的扩展瓶颈通常在表示层而非索引层——这与 [[entities/admem-memory-policy-selective-memory-sjtu-tencent-2026|AdaMem (上交×腾讯)]] 的"存什么"取舍、[[entities/mragent-memory-reconstructed-not-retrieved-nus-icml2026|MR-Agent (NUS ICML 2026)]] 的"如何取回"分别构成记忆系统的不同杠杆点。

### 双设置实验证据

论文在离线（HalfCheetah/Walker2d/Hopper/Maze2D × TD3+BC/IQL/EDAC）与在线（quadruped-walk 等 6 环境 × SAC，对比 SynthER 与 PGR）双设置下验证：合成样本训练效果经常达到甚至超过原始样本集；条件设计与采样策略的消融进一步确认收益来源。^[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl.md] 实验硬件仅为 36 核 Xeon Gold 6240 / 377GB 内存 / 8×RTX 2080Ti，说明方法不依赖顶级算力。

## 实践启示

1. **先诊断价值覆盖，再扩大合成规模。** 下游 RL 性能不涨时，先用记忆类信号检查合成数据是否集中在低价值区域，而不是一味加大合成集——SynthER 需要合成 500 万条（原始集的 2.5 倍）才开始见效，代价高昂。
2. **即插即用的数据层增强。** EMCES 不改动下游算法，TD3+BC/IQL/EDAC/SAC 均可直接受益；工程上可在现有训练管线中以"合成→混合训练"步骤叠加。
3. **价值估计不必上神经网络。** 凡是需要价值/优先级信号的场景（样本加权、课程构造、数据筛选），非参数记忆检索都能提供零训练成本且稳定的估计，尤其适合训练不稳定或算力受限的情形。
4. **表示层优先于索引层。** 移植情景记忆时优先替换学习式哈希状态表示（IsoHash 类），其收益（8000× 存储/25.5× 时间）远大于更换 KD-树等索引实现。
5. **优先化强度是新的可调超参。** 取下界即退化为均匀合成，调大则提升高价值样本合成率但可能牺牲多样性；应与下游算法超参联合扫描。
6. **"记忆评价→生成控制"模式可跨域复用。** 同样的范式适用于 agent 经验复用（如 [[entities/evoscientist-experience-memory-autoskills-2026|EvoScientist]] 把经验聚类成 Skill）与受约束生成，值得在非 RL 场景试验。

## 理论定位

EMCES 是**首个将情景记忆引入可控扩散模型并用于指导强化学习样本合成**的工作。它与 扩散模型 架构相关，但核心创新在记忆驱动的可控生成范式。与 [[entities/mragent-memory-reconstructed-not-retrieved-nus-icml2026|MR-Agent (NUS ICML 2026)]] 同属 ICML 2026 记忆增强学习论文，但 EMCES 聚焦于样本合成场景，MR-Agent 聚焦于 memory-reconstructed retrieval。

论文地址：https://openreview.net/forum?id=mjYcL7esQO

→ [[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl|原文存档]]
