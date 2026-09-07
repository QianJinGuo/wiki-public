---

title: "Google DeepMind Co-Scientist 升级：AI 首次在真实实验室制造半导体"
created: 2026-09-01
updated: 2026-09-07
type: entity
tags: [deepmind, co-scientist, ai-for-science, multi-agent, semiconductor, materials-science, gemini]
sources: [raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

## 概述

Google DeepMind 与杜克大学联合发布了 AI 科研系统 Co-Scientist 的重大升级版本。Co-Scientist 是 Google 基于 Gemini 构建的多智能体科研系统，此前版本主要聚焦于计算机内的假设生成。此次升级的核心突破在于：**Co-Scientist 首次成为一个以真实实验执行为基础的闭环科研伙伴**。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]

## 核心突破

### 从"想"到"做"的范式转变

当前 AI 科研面临的最大瓶颈：「AI 能在计算中提出什么」与「能在物理世界中验证什么」之间的鸿沟。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]

### 材料科学：半导体制造

**1. 无毒新路线合成 MXene**
- Co-Scientist 针对杜克实验室 CVD 系统的具体几何结构和现有原料
- 从 272 个候选方案中提出以六氯乙烷替代剧毒四氯化钛作为前驱体
- 经过 25 轮迭代优化，成功生长出高结晶度的二维层状晶体 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]

**2. 一次成功合成三种单层半导体**
- 仅根据设备硬件约束，在没有任何历史配方参考的情况下
- **第一次尝试中即成功合成高质量单层 MoS₂**
- 推广到实验室此前从未合成过的 MoSe₂ 和 WS₂，同样一次成功
- 借助 Gemini 3 Deep Think，**数分钟内**生成配方并转化为设备控制指令 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]

### 生物学与计算机科学验证

- **生物学**：从稀疏成像数据中预测工程大肠杆菌群集形态，四项形态指标中三项达到定量吻合
- **计算机科学**：自主设计推理时扩展架构 Agent_H，在 HealthBench Hard 与 Professional 基准上超越六个前沿模型

### 可靠性机制

- 将幻觉和抄袭惩罚直接纳入优化目标
- 30 位领域专家、450 次独立评审的双盲研究证实机制有效
- 内置安全系统拒绝了 98.7% 的有害研究请求

→ [[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08|原文存档]] ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]