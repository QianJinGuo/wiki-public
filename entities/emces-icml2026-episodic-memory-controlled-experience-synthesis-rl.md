---
title: "EMCES (ICML 2026) — Episodic Memory-Guided Controllable Experience Synthesis for Reinforcement Learning"
created: 2026-07-02
updated: 2026-09-07
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

## 理论定位

EMCES 是**首个将情景记忆引入可控扩散模型并用于指导强化学习样本合成**的工作。它与 扩散模型 架构相关，但核心创新在记忆驱动的可控生成范式。与 [[entities/mragent-memory-reconstructed-not-retrieved-nus-icml2026|MR-Agent (NUS ICML 2026)]] 同属 ICML 2026 记忆增强学习论文，但 EMCES 聚焦于样本合成场景，MR-Agent 聚焦于 memory-reconstructed retrieval。

论文地址：https://openreview.net/forum?id=mjYcL7esQO

→ [[raw/articles/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl|原文存档]]
