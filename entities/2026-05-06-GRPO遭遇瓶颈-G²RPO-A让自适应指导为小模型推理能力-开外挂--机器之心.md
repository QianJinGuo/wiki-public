---

title: "GRPO遭遇瓶颈？G²RPO-A让自适应指导为小模型推理能力「开外挂」"
created: 2026-07-01
updated: 2026-08-01
type: entity
tags: [ai, training]
sources: [raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心]
confidence: 0.8
---

# GRPO遭遇瓶颈？G²RPO-A让自适应指导为小模型推理能力「开外挂」

---
title: GRPO遭遇瓶颈？G²RPO-A让自适应指导为小模型推理能力「开外挂」^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

source: wechat
url: https://mp.weixin.qq.com/s/AZ2uV91D4bfKQGNpu9VfVg ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]
mp_name: 机器之心
publish_date: 2026-05-06^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

---

# GRPO遭遇瓶颈？G²RPO-A让自适应指导为小模型推理能力「开外挂」

**来源**: 机器之心

**发布日期**: 2026-05-06^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]


**原文链接**: https://mp.weixin.qq.com/s/AZ2uV91D4bfKQGNpu9VfVg ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

---

大模型时代的「炼金术师」们，或许都曾面临一个共同的困扰：当我们试图将 DeepSeek-R1、OpenAI-o1 那种惊艳的推理能力迁移到小规模语言模型（SLMs）时，效果却总是差强人意。现有的强化学习方法如 GRPO 在 7B+ 的大模型上效果显著，但一旦应用到 1.7B 甚至更小参数的模型上，性能提升就微乎其微。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

针对小模型在强化学习中的推理困境，香港中文大学（深圳）T-Lab 唐晓莹教^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]


## 核心要点

> 本文为微信公众号文章，由 WeChat backfill 收录。

## 详细信息

---
title: GRPO遭遇瓶颈？G²RPO-A让自适应指导为小模型推理能力「开外挂」^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

source: wechat
url: https://mp.weixin.qq.com/s/AZ2uV91D4bfKQGNpu9VfVg ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]
mp_name: 机器之心
publish_date: 2026-05-06^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

---

# GRPO遭遇瓶颈？G²RPO-A让自适应指导为小模型推理能力「开外挂」

**来源**: 机器之心

**发布日期**: 2026-05-06^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]


**原文链接**: https://mp.weixin.qq.com/s/AZ2uV91D4bfKQGNpu9VfVg ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

---

大模型时代的「炼金术师」们，或许都曾面临一个共同的困扰：当我们试图将 DeepSeek-R1、OpenAI-o1 那种惊艳的推理能力迁移到小规模语言模型（SLMs）时，效果却总是差强人意。现有的强化学习方法如 GRPO 在 7B+ 的大模型上效果显著，但一旦应用到 1.7B 甚至更小参数的模型上，性能提升就微乎其微。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

针对小模型在强化学习中的推理困境，香港中文大学（深圳）T-Lab 唐晓莹教授携课题组博士毕业生郭永新、邓文博提出了全新算法 G²RPO-A（Guided Group Relative Policy Optimization with Adaptive Guidance）。已被 ACL 2026 主会议（Main Conference）接收。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

该方法通过在 roll-out 过程中注入高质量思维轨迹，并根据训练状态动态调整指导强度，有效缓解小模型面临的奖励稀疏问题。在 Llama、Qwen、DeepSeek 等多个主流模型家族上的实验表明，G²RPO-A 在数学推理和代码生成任务上显著优于 vanilla GRPO，其中 Qwen3-1.7B 在 MATH500 上从 50.96 提升到 67.21，HumanEval 上从 46.08 提升到 75.93。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

- 论文地址：G²RPO -A: Guided Group Relative Policy Optimization with Adaptive Guidance

- 论文链接：  https://arxiv.org/abs/2508.13023

- 代码仓库：  https://github.com/T-Lab-CUHKSZ/G2RPO-A

- 作者： Yongxin Guo♠,♡,, Wenbo Deng♠,, Zhenglin Cheng♣, Xiaoying Tang♠

- 单位： ♠ 香港中文大学（深圳）  ♡ 淘天集团（郭永新为香港中文大学（深圳）T-Lab毕业博士生） ♣ 西湖大学

「我们用 GRPO 训练了 Qwen3-1.7B，结果高奖励候选始终太少，模型很难稳定学到有效的推理策略……」 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

一个灵魂拷问随之而来： 难道小模型注定与高级推理能力无缘吗？^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]


图 1：Naive Guidance 的困境。使用 Qwen2.5-Math-7B 在 s1K-1.1 数据集上训练，简单的固定长度指导在早期训练阶段有短暂提升，但很快与 vanilla GRPO 无异。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

### 

### 一、小模型的「推理瓶颈」到底卡在哪？

### 

当前，尽管 GRPO 等强化学习算法在大模型上取得了巨大成功，但在小规模语言模型（SLMs）上却面临严峻挑战。研究团队通过深入分析发现，问题的核心在于「稀疏奖励」困境： ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

由于 SLMs 自身能力有限，面对复杂推理任务时，它们很难生成高质量的思考链，导致大部分 roll-out 都无法获得正向奖励。如下图所示，Qwen3-1.7B 在代码任务上的奖励分布极其稀疏： ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

图 2：Qwen3-1.7B 在代码任务上的奖励热力图对比。引入 guidance 后，模型更容易采样到高奖励候选，奖励信号显著变得更密集。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

研究团队形象地将其比作「新手司机开手动挡」：无论引擎（模型）如何努力，缺乏正确的引导（指导）依然难以完成复杂的驾驶（推理）操作。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

### 二、G²RPO-A 核心算法架构

### 

为了缓解小模型在 RLVR 中的先天劣势，G²RPO-A 并不是简单地把标准答案喂给模型，而是在 roll-out 的部分轨迹中注入高质量 thinking trajectory，并根据训练状态动态调整 guidance 强度。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

图 3：G²RPO-A 的整体框架。每一步训练都会将 roll-out 分成 guided 和 unguided 两组，再根据当前奖励与历史奖励的比值动态调整后续 guidance length。 ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

G²RPO-A 的核心创新包含两个关键组件：^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

指导机制（Guidance Mechan^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]


## 原文

→ [[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心|原文存档]] ^[raw/articles/2026-05-06-GRPO遭遇瓶颈-G²RPO-A让自适应指导为小模型推理能力-开外挂--机器之心.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

