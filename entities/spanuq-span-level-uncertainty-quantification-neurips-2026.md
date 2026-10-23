---
title: "SpanUQ：从 Token 到 Span 的 LLM 不确定性估计（NeurIPS 2026）"
created: 2026-10-23
updated: 2026-10-23
type: entity
tags: [uncertainty-quantification, llm, neurips-2026, evaluation, calibration, probe, distillation, multi-agent, research]
sources: [raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026]
confidence: 0.78
provenance_state: extracted
---

# SpanUQ：从 Token 到 Span 的 LLM 不确定性估计（NeurIPS 2026）

Amazon 团队在 NeurIPS 2026 提出 **SpanUQ**：把 LLM 的不确定性估计从 Token 级、Sequence 级推进到**语义片段（Span）级**，用一个约 25M 参数的轻量 Probe 在单次前向中同时完成语义片段检测与不确定性打分，相对多次采样方法快 10–20 倍。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

## 问题：不确定性该在哪个颗粒度上衡量

Token 级方法（entropy、token probability）计算开销低、颗粒度细，但 token 并不天然对应完整语义单元：一个日期、数字或实体名可能由多个 token 组成，单个 token 的不确定性不能直接反映整条事实声明的可靠性；模型也可能"非常确定地生成错误事实"，或在表达方式多样时对正确内容表现出高 token entropy。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

Sequence 级方法（多次采样 + 语义一致性）能给整体可靠性一个分数，却难以定位问题所在。原文给出的例子是 "Marie Curie ... won the Nobel Prize in 1901"：前两项事实正确、年份错误，而下游真正关心的是出错的那个具体事实，不是否定整段回答。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

因此论文提出新任务 **SLUE（Span-Level Uncertainty Estimation）**，要求同时满足三点：语义完整性（每个 Span 是可独立评估的语义单元）、细颗粒定位（只标出不确定的部分）、可执行性（下游能针对高不确定片段采取不同策略，例如只验证这两条 claim 而不是重查整段）。作者特别强调第三点：不确定性不该只是评测指标，而应成为下游系统可消费的**决策信号**。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

## 方法：单次前向的 25M Probe

**特征侧**：从冻结的预训练 LLM 抽取多层 Hidden States，经 Multi-Layer Fusion 与 Token Encoder 转成 Span 预测特征；基础模型参数全程冻结，只训练 Probe。作者明确说明这不是"读取一个已存在的事实正确性内部变量"，而是学习 Hidden States 到不确定性标签的映射。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

**片段检测**：不采用 BIO 逐 token 标注，而是借鉴 DETR 的目标检测思路设计 **DETR-Style Span Decoder**——用一组可学习 Span Queries 通过 Cross-Attention 与文本特征交互，并行预测多个候选片段，从而更自然地处理重叠片段与灵活边界。实测 Span Detector F1 = 0.910，相比最强启发式方法相对提升 39.4%。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

**不确定性建模**：不直接回归单个 0–1 分数，而用 **Mixture of Beta Distributions（MoB）** 建模有界连续分布，训练目标是 Beta NLL（拟合连续标签）加 Contrastive Ranking（保证不同 Span 的相对排序正确）——排序能力对下游"先验证哪一条"至关重要。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

**迭代精化**：**UCIR（Uncertainty-Conditioned Iterative Refinement）** 把第一轮预测的不确定性反馈回共享 Decoder 修正片段预测，额外开销低于 15%，并提升不确定性排序相关性。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

## 数据与结果

配套 **SpanUQ-Bench**：20K prompts、约 293K 标注 Span、连续（而非二分类）不确定性标签。标签通过 Multi-Sample Claim Verification 生成 soft label——同一 claim 在多次采样/验证中一致性高则不确定性低，但这只是经验信号，不等于绝对事实正确的概率。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

在五个 LLM backbone（两个模型家族，覆盖 Dense 与 MoE）上，SpanUQ 的 AUROC 为 0.908–0.944、MAE 为 0.110–0.129，优于 Token Entropy、MLP Probe、SelfCheckGPT-NLI、P(True) 等基线；效率上相对 Sampling-Based 方法约 10–20× 加速。另一个有意思的发现是 Sequence 级不确定性**在一定程度上可由 Span 级分解**，说明不同颗粒度的不确定性之间存在可建模的结构关系。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

## 三个下游方向与开放问题

**Uncertainty-Aware Verification**：只对高不确定 claim 调用检索或验证模型，按不确定程度分级（直接用 / 轻量验证 / 外部检索与多模型交叉验证）；但低不确定性不等于事实正确。**Uncertainty-Guided Self-Refinement**：从整段重写走向只修正高不确定 Span，避免改动本来正确的内容。**Uncertainty-Aware Multi-Agent Communication**：Agent 之间不只传自然语言，还传每个 claim 的可靠性信号，接收方可据此请求补证或交叉验证。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

作者也列出未解问题：Calibration 是否足以让分数成为可信决策信号；跨模型（不同基座）输出的 uncertainty score 是否可比——若一个模型的 0.9 与另一个模型的 0.9 含义不同，错误校准的信号会在 Multi-Agent 系统中传播放大；以及能否扩展到复杂推理、代码生成等结构化输出。^[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026.md]

## 关联

- 不确定性作为决策信号，与 [[concepts/self-calibrating-epistemic-instrument|自校准认知仪器]] 和 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] 直接相关：把"可验证性"从事后评测前移到生成时。
- 方法侧借鉴目标检测（DETR query）与蒸馏（multi-sample → 单次前向 Probe），可对照 [[concepts/model-distillation-compression|模型蒸馏与压缩]]、[[concepts/inference-optimization|推理优化]]。
- 多 Agent 传递可靠性信号一节，属 [[concepts/multi-agent-systems|多 Agent 系统]] 与 [[concepts/multi-agent-collaboration-patterns|多 Agent 协作模式]] 的通信协议层问题。
- 同类"生成侧可靠性工程"实践可对照 [[entities/agent-reliability-context-drift-tool-hallucination|Agent 可靠性与工具幻觉]]。
- 论文：arXiv 2607.05721；代码 github.com/damon-demon/SpanUQ；数据集 SpanUQ-Benchmark。

→ [[raw/articles/spanuq-span-level-uncertainty-quantification-neurips-2026|原文存档]]
