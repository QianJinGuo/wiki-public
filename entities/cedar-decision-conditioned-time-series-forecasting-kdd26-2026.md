---
title: "CEDAR：决策条件模拟的双重解耦框架（KDD'26 Oral，中科大 × 阿里 1688）"
created: 2026-09-17
updated: 2026-09-17
type: entity
tags: [time-series, forecasting, decision-conditioned-simulation, causal-inference, transformer, llm-as-feature-extractor, kdd26, alibaba, ustc]
sources: [raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026]
confidence: 0.8
provenance_state: extracted
---

# CEDAR：决策条件模拟的双重解耦框架（KDD'26 Oral）

> **Background**：本文基于 CEDAR 团队（中科大 + 阿里巴巴）在量子位的投稿原文提炼（KDD'26 Oral）。核心问题不是「预测未来会发生什么」，而是「**如果采取这套行动方案，未来会发生什么**」——即决策条件模拟（decision-conditioned simulation）。

## 问题形式化：从「趋势外推」到「动作条件下的推演」

多数时间序列预测模型回答的是「照这个趋势，下周大概卖这么多」，而对决策者「打算做什么」要么视而不见，要么把决策当成普通特征拼进输入，学出的是**相关而非因果**的规律。CEDAR 把商家真实需要的问题形式化为：给定历史状态与一套未来动作序列，推演时间序列在该方案下的演化轨迹。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

典型场景：「如果商家在接下来三周将广告预算提高 50%，同时打九折，销量会变成什么样？」这类 what-if 分析在能源调度、交通管理、金融投资中同样存在——决策者需要的不是趋势，而是候选方案下的未来。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

## 两个非平稳来源：现有方法的结构性缺陷

CEDAR 把传统模型做不好决策条件模拟的症结归为两层，恰好对应这类序列的两个非平稳来源。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

- **来源一：人为决策的影响被「平均」掉了。** 传统协变量建模（如 TFT）把人为决策（调价、投放、调度指令）与历史观测拼在一起喂给模型，学到的是「决策和序列通常一起出现」的相关性，而非「决策驱动序列变化」的因果机制，结果是**自回归惯性**：模型沿历史趋势外推，对反事实的新决策方案极不敏感——商家加大投放，它仍按老剧本走。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]
- **来源二：外界干预与决策效果被「搅」在一起。** 突发热点、节假日、天气、宏观冲击都会扰动序列；不把外界干预与决策效果分离，模型就会「张冠李戴」，把一次外部热点带来的暴涨错记在人为决策头上，进而给出灾难性的决策建议。原文的比喻是车速变化既来自踩油门（人为决策）也来自突然下雨（外界干预），模拟器必须把两件事分开建模。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

## 设计：最终预测 = 决策条件基础推演 + 外界干预残差修正

CEDAR（Controlled and Event-Driven Demand Forecasting via Residual Decomposition）的设计哲学是「既然生成过程天然由两部分组成，就用两个模块分别建模、分阶段训练」。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

**核心点一：动作交错 Transformer（AIT）显式建模人为决策。** AIT 不再把决策动作当附属特征，而是把状态 S 与动作 A 都视为「一等公民」的独立 token，按真实因果顺序交错排列（上一期状态 → 本期决策 → 本期状态，即 s(t-1)→a(t)→s(t)）。这一顺序编码赋予模型**结构性归纳偏置**，让注意力机制显式对齐「决策→状态转移」这条影响通路。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

**与 Decision Transformer 的区别**：DT 类离线 RL 方法学的是「什么策略最优」，CEDAR 学的是**可控的状态转移算子**——不求给出唯一最优策略，而是稳定推演任意候选决策方案下的未来轨迹，供决策者比较挑选，这正是 what-if 分析的关键能力。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

**核心点二：残差修正模块做外界干预的语义融合。** 做法分三步：用大模型（Qwen-Plus）从小红书、抖音等平台的热搜榜单中滤掉娱乐八卦等噪声、只提取可能引发需求变化的商品关键词；再把关键词与未来时间窗内的 26 个节假日/购物节信息合成为一段连贯的「市场描述」文字；最后用文本编码器转向量、修正外生扰动引发的序列偏移。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

两阶段解耦训练还带来一个额外收益：决策条件动态在策略漂移下保持稳定，残差模块可灵活适应非平稳外部环境；消融实验证明一阶段联合训练的效果明显更差。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

## 工程验证：3200 万条轨迹、误差降 57%、A/B 双指标提升

团队在阿里 1688 约 **3200 万条商品轨迹**上验证：预测误差较最优基线降低超 **57%**；线上 A/B 测试带来商家 **LTV 提升 13%、店铺 ROI 提升 15%**。^[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026.md]

从「论文指标」到「线上业务指标」的双重读数，是这类决策条件建模工作少见的落地证据；对做预测型 agent/决策辅助系统的团队，可迁移的部分是**问题形式化（动作条件下的轨迹推演）+ 因果顺序编码（状态/动作交错 token）+ 用 LLM 做外生变量的语义过滤**这条组合拳。

## 相关

- [[entities/decathlon-chronos-2-demand-forecasting-at-scale|Chronos-2 需求预测规模化部署]] — 通用时序基座模型的规模化路线，与本页的决策条件建模互补
- [[entities/crewai-cpg-supply-chain-demand-forecasting-agentic-ai|CPG 供应链需求预测 Agent]] — 供应链场景的 agent 化预测实践
- [[entities/time-series-forecasting-augmentation|时序预测数据增强]] — 训练数据侧方法

→ [[raw/articles/cedar-decision-conditioned-time-series-forecasting-kdd26-2026|原文存档]]
