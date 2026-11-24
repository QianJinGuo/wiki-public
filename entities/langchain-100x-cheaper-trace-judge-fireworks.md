---
title: "LangChain × Fireworks 100x Cheaper Trace Judge — 通用 trace 评估器"
created: 2026-06-17
updated: 2026-11-16
type: entity
tags: [langchain, langsmith, trace, evaluator, fireworks, fine-tuning, qwen, agent, observability]
sources: [raw/articles/langchain-100x-cheaper-trace-judge-fireworks]
review_value: 7
review_confidence: 7
review_recommendation: worth-reading
review_stars: 4
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# LangChain × Fireworks 100x Cheaper Trace Judge — 通用 trace 评估器

> Source: [[raw/articles/langchain-100x-cheaper-trace-judge-fireworks|原文存档]] ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]

## 背景

LangChain 与 Fireworks 合作，**针对 trace 评估场景对 Qwen judge 模型进行微调，实现 100x 成本降低**。文章 2026-06-16 发布于 LangChain Blog。 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]

## 核心概念：Perceived Error（感知错误）

### 定义

> Perceived error is when the user thinks the assistant made a mistake or produced something that needed correction.

**关键区别**： ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]
- Perceived Error ≠ 客观正确性
- Perceived Error ≠ 用户满意度
- 反映的是用户主观"觉得 agent 错了"的信号

### 通用性主张

LangChain 通常推荐团队构建**应用特定的评估器**（因为判断 trace 需要应用上下文），但认为"perceived error"是少数可以**通用化**的评估器之一。理论依据：感知错误的信号在不同应用中是**普遍一致的**。 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]

### 信号来源

从 trace 中推断感知错误： ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]
- 用户纠正（user corrections）
- 拒绝 agent 行为（rejection of an agent action）
- 重复请求（repeated requests）
- assistant 主动承认错误（acknowledgements of errors）

### 输出格式

```json
{"perceived_error": true, "reason": "The user corrects the meeting date the assistant used."}
```

## 数据集构建

### 来源

从两个 LangChain 内部生产 trace 数据集采样： ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]

1. **chat-langchain** — LangChain 文档 Q&A agent（处理概念问题、调试问题、构建帮助） ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]
2. **Fleet** — LangSmith Fleet 产品的真实用户 trace ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]

### 多样性保证

通过内部不同应用场景的 trace，保证 judge 模型见到"perceived error"的**不同表现形式**，避免过拟合单一应用。 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]

## 100x Cheaper 关键

### 微调策略

- **基座模型**：Qwen（具体规格未在文章中明确）
- **训练数据**：人工标注的 perceived error 样本 + 内部 trace
- **推理平台**：Fireworks AI（专用推理基础设施）
- **目标**：保持 frontier 性能的同时降低推理成本

### 通用化验证

文章明确做了**泛化性实验**：测试 judge 在训练集以外的 trace 上是否仍然准确识别 perceived error。这是 trace 评估器能否成为"通用工具"的关键。 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]

## 深度分析

### 为什么 Qwen-3.5-35B 是成本/能力权衡的甜点

文章给出明确的基座选型逻辑：更小的模型在多轮 trace 推理上错误率过高，Qwen-3.5-35B 则是「强、便宜、有余量经微调逼近 frontier」的开放模型 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:64]。训练用 Fireworks 托管 SFT + LoRA 完成 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:68]，结果表显示 chat-langchain SFT 把基座从 90.5%/83.2% 提升到 96.1%/90.8%，超过 Claude Opus（91.6%/90.2%），与 GPT-5.5 各有胜负 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:82-90]。另一个反直觉发现：作为高吞吐低价默认选择的 Haiku 级小模型，开箱表现反而输给强开放模型，成本却更高 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:92]——「用最小 frontier 模型跑批量推理」的惯性策略在 judge 场景失效。

### 单数据集训练与跨域迁移的实验设计

训练刻意只用 chat-langchain 数据，把 Fleet 留作完全未见域来检验迁移 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:66]；数据规模为 885/911 条（train 707/727，holdout 178/184）^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:47-50]，其中带 perceived error 标注的占 24% 与 18% ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:60]。结果：仅用 chat-langchain SFT 的 judge 在 Fleet 上超过所有 frontier 模型 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:98]，再加 Fleet 专属微调只有小幅增益 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:100]。这说明判别能力主要来自信号模式而非领域词表，领域增量边际递减——复现团队应先跨域验证，再决定是否追加领域微调 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:102-105]。

### Perceived Error 作为通用信号的方法论含义

定义刻意与客观正确性、用户满意度切割：答案正确但用户烦躁不算 agent 错 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:21]。可通用化的依据是信号跨应用普遍一致 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:23]——用户纠正、拒绝 agent 行为、重复请求、assistant 承认错误四类 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:27]，且实验专门针对泛化性设计 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:25]。工程上还有两个值得注意的取舍：训练/预测只保留 Human-AI 消息、丢弃全部 tool call（假设信号主要在对话层，留作后续实验杠杆）^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:54-56]；标签用模型评审团一致、二次评审、人工兜底的分级流程生成 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:60]。评估器以 `{"perceived_error": true, "reason": ...}` 结构化富化 trace ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:29]，可直接被下游消费。

### 在 agent 数据挖掘版图中的位置

宏观判断是 agent 已产出世界多数数据，trace 将成为理解 agent 系统行为最富的数据源 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:13]；fine-tuned judge 以 10-100x 成本优势匹配 frontier 精度，且 trace 量越大节省越多 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:111]。这与「开放模型已越过智能阈值」的判断互为印证 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:117]：专用小模型加专用推理，把 trace 从「存起来贵」变成「逐条 mining 得起」。后续方向是帮团队设计训练目标与 rubric 来造自己的 evaluator ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:119]，即 evaluator 本身也在被工程化生产。

1. **成本甜点是 35B 级 + LoRA**：更小模型多轮推理错误率高，35B 微调后即可逼平或超越 frontier ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:64]
2. **先测跨域迁移再谈领域微调**：单域训练的 judge 在未见域已胜过全部 frontier，领域微调仅小幅增益 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:98-100]
3. **通用的是信号不是模型**：可通用化的是四类交互信号与 JSON 输出契约，模型层仍可按域继续微调 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md:27-29]
4. **trace mining 的经济门槛被打穿**：10-100x 成本下降使「每条 trace 过一遍 judge」从不可行变成默认动作，trace 由此接通 [[entities/agent-improvement-data-mining-trace-framework|agent 改进数据挖掘框架]] 与 [[entities/trace2skill-trajectory-distillation-agent-skills|trace2skill 轨迹蒸馏]] 的数据供给
5. **生态位**：与 [[entities/llm-as-a-judge-agent-eval-offline-huolala-2026|LLM-as-judge 离线评估实践]] 相比，本文把 judge 本身变成被优化的对象（选基座、SFT、迁移验证）；与 [[concepts/llm-observability-4-layer-model|LLM 可观测性四层模型]] 对读，perceived error 属于信号层而非存储层。

## 实践启示

1. **通用 trace 评估器是可行的** — perceived error 是一个跨应用、跨域的稳健信号 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]
2. **专用小模型 + Fireworks 推理 = 100x 成本下降** — 比用 GPT-4 类大模型评估每条 trace 便宜两个数量级 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]
3. **trace 评估器分层构建** — 应用特定的 evaluator 仍然必要，但通用层（如 perceived error）可以作为基线 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]
4. **agent 时代 trace 是金矿** — "agents now produce a majority of the world's data"，trace 评估会变成核心基础设施 ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]

## 适用场景

- **LangSmith 用户** — 可直接使用 LangChain 提供的 perceived error evaluator 增强 trace
- **自建 agent observability** — 可借鉴"信号推断 + 小模型 + 专用推理"的范式
- **evaluator 研发** — 验证"特定评估任务是否可以通用化"的方法论

## 相关实体
- [[entities/langsmith-engine-self-improving-agent-trace-based|langsmith engine: trace-based self-improving agent]]
- [[entities/skillsui-enterprise-agent-middle-layer|skillsui 企业 agent 中间层]]
- [[entities/gaode-uplift-model-iteration-agent-long-running-harness|gaode uplift model iteration agent long running harness]]

→ [[raw/articles/langchain-100x-cheaper-trace-judge-fireworks|原文存档]] ^[raw/articles/langchain-100x-cheaper-trace-judge-fireworks.md]
