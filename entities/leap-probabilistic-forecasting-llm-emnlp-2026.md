---
title: "LEAP：基于似然抽取与聚合的 LLM 概率预测（EMNLP 2026）"
created: 2026-09-06
updated: 2026-09-14
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

## 深度分析

### 分工：LLM 只做局部解释，概率模型负责组合

LEAP 的支点是一条职责切分线：LLM 处理局部语义，概率模型负责组合。每次调用只读任务与**一条** evidence 并返回结构化 likelihood parameters，看不到其他证据、累计结果或中间 posterior；prior 给出起点，全部局部判断完成后由确定性的闭式贝叶斯更新算出 posterior。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:42-63]

Monolithic Prediction 把证据解释、冲突处理与最终决策压进同一次调用，自然语言 rationale 只能说明模型「声称」的理由，无法还原每条材料在计算中的真实影响。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:34] 拆开后每个环节都可单独核验：LEAP 不要求模型更聪明，只是把「一次做对」改成许多件可分别检查的事。

### 一次性整体推理为什么掉校准

整批证据一次喂入，误差复合几乎不可避免：解释、消解冲突、折算不确定性、收敛到唯一答案都发生在同一次 forward pass 内，偏差被逐步吸收却不留痕迹。

拆开后两者都可控：诊断子集显示预测跨度从 7 天拉长到 30/60 天时，LEAP 相对 Monolithic 的绝对改进持续扩大，面对更间接的证据倾向于给出更宽的 posterior，减少高置信度错误。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:87] 收益既来自算得更准，也来自「知道自己有多不确定」——与 [[concepts/self-calibrating-epistemic-instrument|自校准认知仪器]] 的自我校准思路同源。

### 似然抽取：把定性证据折算成可比较的数字

Elicitation 把「这条材料支持哪个结果」的定性判断，转成可参与代数运算的数值：连续目标输出分位数，单选输出候选项概率，多选输出各选项的 Bernoulli marginal；三者都约束在概率空间内，因此证据之间可直接比较与叠加，而不靠自然语言描述强度。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:63]

Prior 的取法刻意避免重复计数：有历史数据时用历史序列，缺数据时用一次**不读取当前证据**的 LLM 调用估计基础分布。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:56] 证据很少真正独立，LEAP 因此聚类同源证据、用重复采样检查单条 elicitation 的一致性、过滤数值异常^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:65]——护栏不提升语义能力，却决定聚合输入是否干净。

### 聚合机制：从点估计到可归因的分布

局部 likelihood 就绪后，一次确定性的闭式更新算出 posterior：同一 evidence set 必然得到同一结果，输出是分布（候选概率、分位数或 marginal）而非单点答案，置信度天然携带「有多分散」的信息。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:42-63]

把任意一条 evidence 移除后再跑同样的更新，新旧之差即该证据的 leave-one-out contribution，无需模型事后补写解释。由此可看出哪些材料推动了某个候选、哪些来源几乎没改变预测、置信度是否依赖少数证据。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:71-75]

### 受控对比、skill 化与评估边界

公平性论证采用消融式设计：两种流程共享同一 evidence set，只比较最终预测方式（与 [[entities/benchmark-score-comparability-reference-traceability-zatona-2026|基准分数可比性]] 关注的问题一致），控制实验另给 Monolithic 相同 prior 或相近 token budget。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:79-85] 五个 base model 上 FutureX 绝对增益 3.6–18.1 点，并复用 DeerFlow、Hermes、OpenClaw、MiroFlow 的原始 agent trace，提升不依赖特定模型或框架。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:81-83]

代价是 p95 延迟更高（整次预测要等最慢的 evidence-level call）^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:85]；流程已开源为可插拔 skill，evidence set 固定后调用即可，预测成为 tool call 而非 prompt 技巧——这与 [[concepts/harness-engineering-framework|Harness Engineering]] 中「能力以工具边界暴露」的做法一致。^[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026.md:93] 边界也清楚：结论限于 FutureX / GAIA / BrowseComp 类事实型任务，self-elicitation bias 与跨域标定仍是开放问题。

## 实践启示

1. **按 stakes 分流**：低风险、粗粒度的判断直接让 LLM 给点估计；高风险、证据丰富、需事后审计的场景才铺开 elicit + aggregate，收益主要来自可追溯性。
2. **先冻结 evidence set**：搜索与证据整理留给上游 agent，证据固定后依次跑 prior estimation → 逐条 elicitation → posterior aggregation，避免中途新增证据污染计算。
3. **用 leave-one-out 审计证据**：逐条移除重算，按影响力排序，揪出「看似相关却没改变预测」的材料，并检查结论是否只靠少数来源支撑。
4. **把去重与一致性检查当护栏**：聚类同源证据、重复采样验证单条 likelihood 的稳定性、过滤数值异常，防止同一份报告被当成多条独立支持。
5. **用校准指标而非单点正确率评估**：NCRPS、Spherical 同时惩罚错误方向与过度自信；跨方法比较须共享 evidence set 与 token budget。
6. **为尾延迟与适用边界留预算**：逐条 elicitation 抬高 p95，适合离线或非实时决策；self-elicitation bias 与跨域标定仍属开放问题。

→ [[raw/articles/leap-likelihood-elicitation-aggregation-probabilistic-forecasting-2026|原文存档]]