---
title: "LEAP：基于似然抽取与聚合的 LLM 概率预测（EMNLP 2026）"
created: 2026-09-06
updated: 2026-09-07
type: entity
tags: [llm, probabilistic-forecasting, evidence-aggregation, emnlp, agent-evaluation, uncertainty]
sources: [raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LEAP：基于似然抽取与聚合的 LLM 概率预测（EMNLP 2026）

LEAP（Likelihood Elicitation and Aggregation for LLM-based Probabilistic Forecasting）是 EMNLP 2026 Main Conference 收录论文提出的预测流程，核心思路是把「读完所有证据后一次性猜答案」（Monolithic Prediction）改造成「逐条证据抽取似然 + 显式概率聚合」，让每条证据对预测结果的影响可计算、可追溯。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]

## 核心机制

**分工原则：LLM 负责局部语义，概率模型负责组合。** 每次 LLM 调用只读取任务与一条 evidence，返回结构化 likelihood parameters；模型看不到其他证据、累计结果或中间 posterior，避免在局部判断时提前合并信息。Prior 提供预测起点，全部局部判断完成后由显式概率模型统一计算 posterior。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]

**可追溯性**：对进入 posterior 的任意 evidence item，系统可暂时移除它再运行一次相同的闭式更新，新旧结果之差即该证据的 leave-one-out contribution——无需模型事后补写解释。使用者可查看哪些材料推动了某个候选结果、哪些来源几乎没改变预测、置信度是否依赖少数证据。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]

## 实证结果

- 任务集：FutureX、GAIA、BrowseComp（预测 + 信息检索），LEAP 与 Monolithic 共享同一 evidence set，只比较最终预测方式。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]
- 基座模型：DeepSeek-V3.2、Gemini-3.1-Flash-Lite、Claude-Haiku-4.5、GPT-5.4-mini、Grok-4.20-Fast，5 个模型上 LEAP 均提升 FutureX/Spherical/Accuracy/NCRPS；FutureX 绝对增益 3.6–18.1 点，Spherical 增益 2.5–15.1 点。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]
- 外部框架：直接使用 DeerFlow、Hermes、OpenClaw、MiroFlow 的原始 agent trace，macro-average 五项指标均有提升。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]
- 诊断子集：预测跨度从 7 天到 30/60 天，LEAP 相对 Monolithic 的绝对改进持续扩大；面对更间接的证据时给出更宽的 posterior，减少高置信度错误。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]

## Skill 化接入

代码已开源（github.com/layingfish/LEAP），以可插拔 skill 形式提供。已有 agent 可保留自己的搜索/浏览/证据整理方式，在 evidence set 固定后调用 LEAP 完成概率预测。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]

## 意义与关联

LEAP 属于「证据 → 结论」可追溯性的工程化路径，与 agent 推理可信度、评估框架直接相关：它不改变搜索与证据收集流程，只重设计 evidence→forecast 的转换，因此可作为 skill 叠加在任意 agent loop 上。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md]

- 相关：[[entities/agent-evaluation-four-layer-outcome-decision-action-reliability-aliexpress-2026|Agent 评估四层框架]]、[[entities/time-series-forecasting-augmentation-methods|时序预测增强方法]]、[[entities/decathlon-chronos-2-demand-forecasting-at-scale|Chronos 需求预测]]
- 概念：[[concepts/agent-memory-lifecycle-philosophies|Agent 记忆生命周期哲学]]（记忆与证据的长期管理）

→ [[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026|原文存档]]