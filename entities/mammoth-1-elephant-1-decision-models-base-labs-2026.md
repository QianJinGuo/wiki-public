---
title: "Mammoth-1 与 Elephant-1：Base Labs 的决策模型技术报告"
created: 2026-10-22
updated: 2026-10-22
type: entity
tags: [model, post-training, decision-model, uncertainty, attention, benchmark, qwen]
sources: [raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026]
confidence: 0.75
provenance_state: extracted
---

# Mammoth-1 与 Elephant-1：Base Labs 的决策模型技术报告

Base Labs（Baseten）2026-10-20 发布技术报告，推出两个决策模型 **elephant-1（9B）** 与 **mammoth-1（27B）**，由 Qwen3.5-9B / Qwen3.5-27B **后训练**而来。mammoth-1 在 Jev Decision Index 0.4 的 208 个模型中排名第一（71.24），elephant-1 是 9B 档最强（超过 Jev 1.14），并在 Artificial Analysis Decision Index 上与上一代最强 d1-ultra 打平（65.2 vs 65.0，mammoth-1 为 65.9）。作者自述两个方法要点：**(a) 全注意力层使用 block-bidirectional attention mask**（在数据固定时优于 causal attention），**(b) 高度可扩展的数据配方**。^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

## 什么是决策模型（decision model）

决策模型接收一个 **state**（JSON 值，如工单或文档，图片/视频以内联引用）与一组 **questions**，对每个问题直接返回答案上的概率分布，不生成自由文本。问题有三种指令类型：^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

- **Choice**：1–255 个选项（可带描述），每题返回各选项概率（和为 1）与 confidence，通常把最高概率重标定使 0 = 均匀猜测、1 = 绝对确定——这是决策模型的规范输出形态。
- **Yes/no**：给出真/假判定说明，返回单个 P(true)。
- **Score**：2–10 档量表（如"客户有多生气"，0 平静 → 4 暴怒），返回每档概率、期望档位（概率加权平均）与 confidence（全概率落在单一档为 1，均匀分布为 0）。

作者给出的四点相对普通 LM 的优势：a) 答案即给定选项之一，**无需解析生成文本**；b) **不生成任何 token**，因此便宜；c) **易于阈值化**——概率足够高才自动执行，否则升级给人；d) 可以把模型**训练成不确定性校准（uncertainty calibrated）**的。普通语言模型虽然也能做同样的判断，但需要解码文本再解析答案，且除非显式提示否则不提供可靠的自身确定性度量。^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

## 数据配方与不确定性感知数据构造

报告方法学章节以"data, data, data"定调：数据来源、数据处理、**对数据处理过程做 looping**、**数据演化（data evolution）**，以及 **uncertainty-aware data construction**（不确定性感知的数据构造）构成方法主体；评测基准为 Jev Decision Index（37 个公开 benchmark + 未披露数量的私有测试）与 Artificial Analysis Decision Index（知识、推理、工具调用与安全评测）。附录 A1 给出每个 Jev Decision Index benchmark 的分项得分。^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

## 谱系位置

decision model 这一 **格式** 由 TypeSafe 于 2026 年 9 月随 Jev 与 System One API 引入（命名取自 Daniel Kahneman 关于大脑"直觉系统"快速决策的描述），此后多家训练并托管决策模型。mammoth-1/elephant-1 是同一族谱上向后训练与数据侧加码的迭代，与 [[entities/liquid-ai-d1-edge-decision-models]] 属同一"决策模型"家族的不同团队路线（d1 系列此前是 Artificial Analysis Decision Index 上的最强档，本报告的 elephant-1 与之打平）。^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

## 工程要点摘要

| 项 | 值 / 做法 |
|----|-----------|
| 基座 | Qwen3.5-9B（elephant-1）、Qwen3.5-27B（mammoth-1），后训练 |
| 关键架构改动 | 全注意力层 block-bidirectional attention mask（数据固定时优于 causal） |
| 输出形态 | 每问题一个概率分布 + confidence；无 token 生成 |
| 主要基准 | Jev Decision Index 0.4（37 公开 + 私有）、Artificial Analysis Decision Index |
| 高分 | mammoth-1 71.24（#1/208）、65.9 AA；elephant-1 65.2 AA |
| 对手 | d1-ultra、Jev 1.14、Microsoft-Decision-1、Perplexity Decider v1.1 27B |

## 关联

- [[entities/liquid-ai-d1-edge-decision-models]] — 同族（decision model）另一团队路线，AA Decision Index 上的直接对手
- [[concepts/attention-mechanism]] — block-bidirectional mask 是注意力掩码层面的变体
- [[concepts/inference-optimization]] — "不生成 token 因此便宜"是决策模型格式的部署侧动因

→ [[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026|原文存档]]
