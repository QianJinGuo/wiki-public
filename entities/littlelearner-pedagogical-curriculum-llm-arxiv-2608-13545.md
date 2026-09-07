---
title: "LittleLearner：课程受控预训练——预训练过滤设定能力上限的实验证据"
created: 2026-08-17
updated: 2026-09-07
type: entity
tags: [pretraining, curriculum, data-centric, knowledge-acquisition, elicitation, arxiv, mpi]
sources: [raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LittleLearner：课程受控预训练——预训练过滤设定能力上限的实验证据

> **Background**：MPI for Intelligent Systems × ELLIS Institute Tübingen × ETH Zürich（Fanfei Li 等 7 人），arXiv:2608.13545。以美国小学 K–5 课程为界的受控预训练实验——同一架构、同一 token budget，唯一变量是预训练语料分布。^[raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545.md]

## 核心方法：把知识边界做成受控变量

现代 LM 的训练数据「一次全见」，因此无法分辨一个新技能究竟是被**学到**（acquired，预训练分布中包含的知识浮现）还是被**引出**（elicited，已有能力在提示/后训练下被激活）。LittleLearner 直接约束训练分布：**88B-token LittleCurriculum** 从 FineWeb-Edu 蒸馏，五阶段过滤流水线对齐 Common Core（K–5），五年级以上概念被显式排除。^[raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545.md]

三档模型（0.6B/1.3B/5B）从零在课程语料上训练，每档配同架构、同 token、同 recipe 的 Unfiltered 对照——唯一差异是 pretraining corpus。知识边界因此成为可探测的实验变量：模型「只知道五年级学生知道的东西」。^[raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545.md]

## 关键发现：Elicitation, not Acquisition

scaling、SFT+GRPO 后训练、in-context learning 都能放大**课程内**（in-scope）所学，但**都无法有意义地提升课程外**（out-of-scope）表现。结论：**预训练过滤器设定了有效能力上限**——标准干预只能「引出」已内化的能力，不能补上预训练分布从未教过的知识。^[raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545.md]

这一发现与「后训练能教新知识」的直觉形成张力：后训练（SFT/GRPO）放大的是分布内能力，而非在分布外新增知识。对 RLHF/GRPO 后训练的边界判断有直接参考意义。

## 资源与可复现性

- 托管 5B 模型浏览器可对话；模型 checkpoints 分 base / GRPO（MathCAMPS 数学后训练）/ chatty 三档：^[raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545.md]
- 数据集（LittleCurriculum）与五阶段过滤流水线全开源，对比实验设计（matched control）完整可复现。^[raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545.md]
- BibTeX: `arXiv:2608.13545`^[raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545.md]

## 相关研究

- 与 [[entities/notes-on-pretraining-parallelisms-and-failed-training-runs|预训练失败模式]] 同属预训练实证研究族，本作聚焦数据分布侧。
- 与 [[entities/emo-pretraining-mixture-of-experts-for-emergent-modularity-ai2|预训练涌现模块性]] 互补：那里研究结构涌现，这里研究知识边界。
- 与 [[entities/generalization-dynamics-lm-pretraining|LM 预训练泛化动力学]] 概念同源：分布内 vs 分布外泛化的边界证据。

## 实践启示

数据工程视角：预训练语料的过滤/编排决策不只是「质量 vs 数量」权衡，而是直接决定能力天花板——课程化语料可以按知识范围精确塑造模型能力边界（教育、领域专用模型场景）。评估视角：能力评测需区分「分布内表现」与「分布外泛化」，后训练指标好看不等于新增了分布外知识。

→ [[raw/articles/littlelearner-pedagogical-curriculum-llm-arxiv-2608-13545|原文存档]]