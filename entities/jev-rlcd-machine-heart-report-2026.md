---
title: "2400万人围观前OpenAI研究员做了个闭嘴模型（机器之心）：Jev 与 RLCD 校准决策训练"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: [jev, system-one-model, rlcd, typesafe, diogo-almeida, calibrated-decisions, model-architecture]
sources: [raw/articles/2400万人围观前openai研究员做了个闭嘴模型]
confidence: 0.85
provenance_state: merged
---
# Jev 与 RLCD：机器之心对 TypeSafe System One Model 的报道（第 4 来源）

机器之心（2026-09-17）对 TypeSafe 首款 System One Model **Jev** 的深度报道，为 [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026|Jev 快判断层 entity]] 补齐三个此前未覆盖的角度：**作者背景与立项动机**、**RLCD 训练方法叙事**（含「0% 幻觉」的论证与争议）、以及**外部实测与社区质疑**。核心架构与工程拆解见主 entity。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

## 背景：做出 RLHF 的人，开始反思 RLHF

Diogo Almeida 是 2022 年 InstructGPT 论文共同作者、OpenAI 早期 RLHF 与 GPT-4 相关工作的参与者——今天 ChatGPT 式「能听懂人话、会聊天」的训练路线，他是奠基者之一。离开 OpenAI 后他用四年怀疑这条路线，答案很直接：**AI 不需要这么爱说话**。「聊天能力真的是软件自动化需要的核心能力吗？」这是 Jev 立项的起点。两年隐身开发后，他创立 TypeSafe 发布了 Jev——**主动放弃自由文本生成**的模型。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

## Jev 的产品形态：非自回归的概率判断机

传统 LLM 自回归地逐 Token 生成「根据用户的描述，我认为这封邮件属于……」，程序还要再从文本里解析字段。Jev 直接砍掉中间这段话：程序提前规定输出 schema，模型读取输入后**并行产生**类型确定的结构化判断，附带概率与置信度——「Unstructured state in, typed probabilistic decisions out」。因为没有整段文本要「写」，并行替代顺序计算，这是速度优势的架构根源。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

## RLCD：Reinforcement Learning for Calibrated Decisions

这个名字本身就是对 RLHF 的回应：RLHF 优化「人类偏好」（聊天助手目标），RLCD 优化**校准**（软件自动化目标）——模型对一类判断长期给出 90% 置信度，这些判断就应真的约 90% 正确。校准让软件能围绕概率写逻辑：95%+ 直接执行 / 70% 交给更强模型 / 40% 送人类，AI 成为可插进程序的「模糊 if 语句」，即 TypeSafe 所说的 composable intelligence。与 [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|RL 算法演化史（PPO/DPO/GRPO）]] 对照：RLHF 家族优化偏好排序，RLCD 把目标换成概率的真实性（epistemically honest probabilities）。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

## 「0% 幻觉」的论证与偷换

通常的幻觉指编造不存在的事实；TypeSafe 指的是**模型生成了程序未定义、无法处理的输出**——普通 LLM 被要求固定格式仍可能输出选项外的新类别或额外解释，下游程序无从处理。Jev 的 type safety 从源头砍掉这种可能：所有输出及类型提前定义，程序只允许「投诉/退款/咨询」三个答案，它就不可能创造第四个。官方注明 0% hallucination 不是统计经验数据，而是 schema matching 被构造性保证后直接记为 0%。机器之心点破其中的文字游戏：校准≠没有幻觉——一个模型可以概率完全校准但答案全错。calibration 回答「程序要不要执行」，type safety 回答「程序能不能处理」，都不等于「答案正确」。这与 [[concepts/self-calibrating-epistemic-instrument|自校准认识论]] 的立场一致：不确定性应作为输出的一部分。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

## 性能数字：官方口径与外部实测

- 官方：速度 20-200x，成本最高降至 1/400，输出 Token 免费，端到端 70-500ms，输入 $0.042/百万 Token；内部 workflow eval 最高 193.6x 速度 / 444.6x 成本优势（自测有设计偏差风险）。
- 外部实测（Every 评测负责人 Mike Taylor）：37 篇文档 777 次独立判断总耗时 <0.7s；写作质量判断 0.35s vs Fable 5.1 的 8.83s——快 25 倍、成本低 580 倍。

在高度结构化的「判断题」上，效率曲线确实惊人；但所有数字的共同前提是任务被限制在封闭输出空间内。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

## 社区质疑与冷静面

Jev 爆火（2400 万浏览）同时招致三层质疑：Hugging Face 研究员 Niels Rogge 吐槽「一个 JSON 分类器竟然有 1200 万浏览，我们正处于泡沫中」；开发者 Harsha Gundala 用 Qwen2.5-1B 两小时复刻相似版本（并行多字段 JSON 判断 + 预设候选上直接算概率，无需新训练）；RLCD 的训练方法、reward function 与系统性校准指标至今未完整公开——「校准」恰是三者中最难验证的一环。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

## 深度分析

**1. 质疑反而证明了这条路线的价值。** 如果一个 1B 小模型两小时就能做出类似能力，说明的未必是「Jev 没有技术含量」，而是**这类能力根本不需要昂贵的大模型**——这正是 Jev 想讨论的问题：过去几年我们把越来越多任务交给通用大模型，对大量任务是「杀鸡用牛刀」。Jev 把最普通、数量最大、最易被忽略的判断单独拿出来，主动放弃聊天与自由生成。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

**2. 「幻觉」一词的语义分裂暴露了评测话语的错位。** TypeSafe 用 0% hallucination 指代 type safety（构造性保证），读者却按「编造事实」理解（经验性统计）。它划开两类风险面——面向人的系统担心事实错误，面向程序的系统担心 schema 逃逸。混用同一个词，说明行业还没有为「程序消费的 AI」建立独立于「人消费的 AI」的评测词汇。^[raw/articles/2400万人围观前openai研究员做了个闭嘴模型.md]

**3. 校准比 type safety 难得多，也是 Jev 真正的技术赌注。** type safety 是 schema 工程问题，开源社区两小时可复现；校准是训练目标问题——logits 概率人人都有，但「80% 把握时真的 80% 正确」需要 RLCD 级别的专门优化，证据尚未公开。判断 Jev 成败的关键指标不是速度榜，而是外部可复现的校准曲线（reliability diagram）。这与 [[concepts/self-calibrating-epistemic-instrument|自校准认识论]] 互相印证：诚实的概率比斩钉截铁的答案更值钱。

**4. 产品哲学：从「贴地飞行」看第二条叙事路线。** AI 产品的常规出场方式是刷新 benchmark、证明能力上限；Jev 走另一条路——把模型能力压到「刚好够用」，紧贴真实工作流里数量庞大、重复发生、对延迟和成本敏感的问题。一个不炫目的「JSON 分类器」获得 2400 万浏览，说明市场开始奖励「解决每天都遇到的问题且能力恰好够用」，产品竞争将从能力上限转向**最简求解路径**。

## 实践启示

- **分流设计**：把工作流调用按「生成 vs 判断」分类，判断类调用（分类/打分/路由/结构化输出）可下沉到 Jev 类 System One 模型，延迟与成本收益立竿见影。
- **概率阈值分层执行**：围绕校准概率写业务逻辑（≥95% 自动执行 / 中段升级强模型 / 低段转人工），把「模糊 if 语句」落成降级链。
- **校准验证先行**：采用任何「校准模型」前，先要求第三方 reliability diagram 证据——type safety 可当场验证，校准必须跑数据。
- **词汇纪律**：在架构文档中区分 factual hallucination（事实编造）与 schema hallucination（类型逃逸），勿用「0% 幻觉」做采购决策依据。

→ [[raw/articles/2400万人围观前openai研究员做了个闭嘴模型|原文存档]] · 相关源：[[raw/articles/jev-rlcd-decision-function-feixue-ai-2026|飞雪 AI：Jev 决策函数解析]]
