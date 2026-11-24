---
title: "rd-signal-2：生产规模下的 Agent 行为二元分类（Frontier Classification at Production Scale）"
created: 2026-08-12
updated: 2026-11-14
type: entity
tags: [agent, evaluation, classification, trace, observability, raindrop, llm-judge, cost, production]
sources: [raw/articles/rd-signal-2-frontier-classification]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# rd-signal-2：生产规模下的 Agent 行为二元分类

> **来源**: Raindrop（Ben Hylak、Manav Shah、Ryan D'Onofrio），2026-08-11 发布 Signals 2.0 技术博客。

## 核心主张

Agent 行为评估的本质是**二元分类问题**（行为好/坏），但这是困难的对齐问题：需要先对齐人类对"好/坏"的定义，再让模型/流水线对齐该定义。rd-signal-2 用**确定性过滤 + 语义分类头**的组合，从生产 traces 构建任务特定分类器，以接近 GPT-5.6 Sol xhigh 的精度实现 1600× 的成本下降。^[raw/articles/rd-signal-2-frontier-classification.md]

## 架构模式（可迁移知识）

### 构建时推理 vs 执行时计算分离

- 传统 LLM Judges：成本 = 流量 × 全 trace 推理——每条 trace 重新发现同一证据，判断还每次略不同。^[raw/articles/rd-signal-2-frontier-classification.md]
- rd-signal-2：成本 = 构建一次 + 确定性执行 + 模糊候选 × 紧凑上下文。**模型调用随不确定性扩展，而非随流量扩展**。^[raw/articles/rd-signal-2-frontier-classification.md]

### 确定性过滤 + 分类头两层结构

用代码（而非 prompt）表达可判定的失败模式：收集 trace 中的工具调用 → 条件检查（≥3 次相同输入且全部失败）→ 不满足直接 non-match 早退（零模型调用）；满足才把精简证据 + 语义判断交给任务特定分类头。**失败的定位从"单步"扩展到"跨步骤关系"**——如"重复工具失败与最终成功声明之间的矛盾"。^[raw/articles/rd-signal-2-frontier-classification.md]

### 连续再评估循环

部署后每天随机抽样生产 Signals 用 frontier models 评估，发现 drift/regression 即 rerun prompt optimization 并 retune：deploy → sample → evaluate → find regressions → retune → re-evaluate。模型/harness 变化会改变 trace 形态，Signal 永不真正完成。^[raw/articles/rd-signal-2-frontier-classification.md]

## 关键数据

- 精度对比：Raindrop 71% vs GPT-5.6 Sol xhigh 71% vs Luna xhigh 67%（Claude Sonnet 5 adaptive 78%）；召回 87% vs 91% vs 83%
- 相对成本（每分类 trace）：Raindrop 1× vs Luna 260× vs Sol 1,600× vs Claude Sonnet 5 adaptive 1,650×
- 匹配率对解释的敏感度：同一行为的 4 种解释跑同一 2,000 条 traces，匹配率 0.9%–4.6%，67% 的匹配 trace 被至少一种解释质疑——人类-模型对齐是核心难点
- 规模：median 分类耗时 100ms，每月评估 200 亿+ traces；每个 Signal 默认 untrusted，隔离环境无凭证无 egress

## 深度分析

### "一切皆二元分类"：把评估问题还原为对齐问题

原文的出发点值得细读：它借用 OpenAI 关于幻觉的二元分类视角，却把重心从"模型能力"挪到"组织对齐"。给定一段 agent 行为要么好要么坏，这个判断本身并不需要前沿智能——真正昂贵的是搞清楚"好"的定义。同一个行为的四种解释在同一批生产 traces 上匹配率相差五倍以上（0.9%–4.6%），说明失败判定的最大方差来源是规格说明而非分类器本身。这解释了为什么 Raindrop 把主要工程投入放在"自动化研究循环"（研究 traces → 写代码组装证据 → 训练任务专用模型）而非单纯堆模型：规格错误在循环中被反复暴露和修正，比一次性 prompt 工程更可靠。换个角度看，这也把评估的组织成本显性化了：分类器训练可以自动化，而"好/坏"边界的探索必须由人来主导——流水线的价值在于让人只做一次判断，然后由机器低成本地重复执行；"哪种解释正确"不是模型问题，而是组织必须先自己回答的问题。

### 失败定位的粒度迁移：从单步到跨步骤关系

第一版 Signals 评估单轮 input-output，假设相关证据都在对话内；agent 时代的失败则散布在多轮、工具调用和子 agent 展开中，有时横跨数十万 token。典型案例是 update_record 三次超时后助手仍宣称成功——失败不在任何单步内，而在"重复工具失败"与"最终回复"的矛盾关系上。rd-signal-2 的应对是把这种关系编码进确定性代码：收集全部相关工具调用、比对输入、检查失败次数，条件不满足时零成本早退；满足时才把"失败尝试 + 最终回复"这一精简证据交给语义分类头。这个设计隐含一个重要取舍：代码路径表达能力有限（只能覆盖可形式化的失败模式），换来的是判断的确定性和可复现性——模糊的语义判断被压缩到最小必要范围。维护层面这也是一种克制：失败模式以代码形式沉淀为可审计资产，规格变更时只需重写那几行条件，而不是重新调教整段 prompt。

### 成本函数的形状：摊销 vs 按流量付费

传统 LLM Judge 的成本模型是 流量 × 全 trace 推理，且每条 trace 都重新发现同样的证据、每次判断还略有漂移。rd-signal-2 把推理成本前置为一次性构建成本，摊销到未来所有 trace；执行期只有确定性检查（免费）加上"模糊候选 × 紧凑上下文"的少量调用。这产生一条可推广的原则：模型调用应随不确定性扩展，而非随流量扩展——绝大多数 trace 因不满足触发条件而零调用，只有真正模糊的候选才消耗推理。1600× 的成本下降主要来自这个结构，而非更小的模型硬啃全量上下文；小模型本来处理不了长上下文中的微妙稀疏失败，所以正确路径是"代码缩小上下文 + 专用头做窄判断"，而不是让通用模型硬扛。这也解释了商业模式差异：Signals 便宜到可以随平台免费提供，而要求客户自付推理费的平台把 judge 成本留在客户侧——成本结构决定了评估能否成为人人都用得起的基础设施。

### 分类器作为持续运营的资产：漂移、隔离与服务化

Signal 永不真正完成：模型或 harness 的变化会改变 trace 形态，规格本身也可能被推翻。每日随机抽样 + frontier model 复评 + 回归后 retune 的闭环，实质是把分类器当成需要 SLA 的运营资产而非一次性交付物。运营侧同样有设计感：每月数百亿条 traces 的规模下 median 分类耗时仅 100ms，依赖 queueing/retry 体系，且 trace context 只取一次并缓存到 evaluation worker 附近；每个 Signal 默认视为 untrusted，跑在无凭证、无 internet egress 的隔离环境，runtime 只能访问本组织数据。这提示任务特定分类器的规模化不只是模型问题，更是"把不可信生成的代码当作多租户负载来服务"的问题——安全边界（默认隔离、数据就近）与性能（缓存、排队）必须与分类质量一起设计。对自建评估体系的团队，这个闭环还意味着要为分类器预留监控预算：与其追求一次性的高精度上线，不如把抽样复评和 retune 常态化，让精度成为随时间可维护的量。

## 工程启示

1. **确定性优先**：能用代码表达的条件判断绝不交给模型——大部分 trace 可零模型调用早退，这是成本下降的主要来源。^[raw/articles/rd-signal-2-frontier-classification.md]
2. **证据组装是分类器的一部分**：对跨多轮/多工具/长上下文（数十万 token）的失败，先写代码组装相关证据（deterministic filtering），再把精简后的语义判断交给小模型。^[raw/articles/rd-signal-2-frontier-classification.md]
3. **评估器也需要 drift 监控**：分类器部署后因模型/harness 演化而漂移，需持续抽样再评估闭环。^[raw/articles/rd-signal-2-frontier-classification.md]

## 相关实体

- [[entities/ai-agent-trace-evals-stability-cost-evaluation-zhangyanfei|Trace 即 Evals（张雁飞）]] — 同为 trace 驱动评估范式；rd-signal-2 给出生产规模实现路径
- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation|YAML 驱动 Agent 评估框架]] — 声明式评估对照
- [[entities/agent-improvement-data-mining-trace-framework|Agent 改进的数据挖掘视角]] — trace 数据的另一消费方式
- [[entities/agent-observability-5-layer-architecture|Agent 可观测性五层架构]]
- [[entities/agent-protocol-cost-evolution-roundtable-2026|Agent 落地真相：协议、成本与进化]] — 成本治理上下文

## 相关概念

- Agent 可观测性
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]
- [[concepts/context-engineering|Context Engineering]]

→ [[raw/articles/rd-signal-2-frontier-classification|原文存档]] ^[raw/articles/rd-signal-2-frontier-classification.md]
