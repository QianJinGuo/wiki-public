---
title: "Scaling Domain Data Repetition in LLM Pretraining — 域数据重复率随规模扩展"
created: 2026-08-19
updated: 2026-08-19
type: entity
tags: [llm, pretraining, data-repetition, data-scaling, tokens-per-parameter, training, ai2, domain-data]
sources: [raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining]
confidence: 0.8
provenance_state: extracted
---

# Scaling Domain Data Repetition in LLM Pretraining — 域数据重复率随规模扩展

## 概述

arxiv 2608.14071（2026-08-14 提交，Jingwei Li 等 8 位作者）研究 LLM 预训练中**域数据重复**（domain data repetition）如何随模型规模扩展。核心背景：LLM 规模扩大时训练 token 预算必须同步增长以维持合适的 tokens-per-parameter（TPP）比，但**高质量域数据远比通用网页数据难扩展**——这使「重复使用稀缺域数据」成为预训练的关键权衡。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md]

论文聚焦一个反直觉的核心发现趋势：最优重复率如何随模型规模 / 训练 token 预算的变化而缩放，为「数据重复多用」这一业界常见但缺乏理论锚定的做法提供实证尺度。^[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining.md]

## 意义

该研究直接回应 LLM 训练中的数据工程难题——当高质量域数据供给不足时，重复率该如何设定。它与 [[entities/generalization-dynamics-lm-pretraining|LM 预训练泛化动力学]]、[[entities/notes-on-pretraining-parallelisms-and-failed-training-runs|预训练并行与失败记录]] 同属 LLM 预训练实证研究前沿，为数据配比与训练预算设计提供可迁移的 scale 规律。

→ [[raw/articles/arxiv-2608-14071-scaling-domain-data-repetition-llm-pretraining|原文存档]]
