---
title: "Agent 自改进循环"
created: 2026-06-12
updated: 2026-08-30
type: concept
tags: [concept, self-improvement, agent, bootstrapping, feedback-loop, evolution]
sources: [entities/agent-self-improvement-six-mechanisms, concepts/ai-self-improvement-bootstrapping, entities/hermes-agent-self-evolving]
---

## 定义

Agent 自改进循环：agent 在使用中观察自己的失败/成功、产生改进 hypothesis、修改自己的 prompt / skills / config、验证、生效。和模型微调的关键区别：自改进发生在「prompt 工程层」而非「权重层」，时间尺度从月降到分钟。

## 核心范式

- **反思 → hypothesis**：每次失败后总结「下次应该怎么做」
- **改 prompt / skill 而非改模型**：修改成本低、回滚容易
- **人类作为最终验证者**：自改进不能完全无人监督，否则会跑偏
- **6 种机制**：reflection / skill 累积 / memory 重写 / tool 优化 / prompt 蒸馏 / fine-tuning trigger

### 背景与提出

agent 自改进循环的提出背景是 2023-2024 年 agent 系统落地过程中的一个核心矛盾：LLM 的能力由训练决定，但训练是离线的、周期长的、大量资源消耗的；而 agent 系统在实际部署中每天都在遇到新问题、产生新失败，这些实时产生「改进信号」无法被离线训练利用。解决这个矛盾的第一步是意识到：模型权重之外，还有 prompt、memory、tool、skill 这些「运行时可调参数」，这些参数可以在部署后持续优化。

[[entities/agent-self-improvement-six-mechanisms|Agent 自改进六机制]] 的论文是第一个系统性地整理 agent 如何在运行中改进自己的研究。论文的核心观点是：agent 的「自我改进」不只有「模型权重更新」这一种形式，还包括在 prompt 层、memory 层、tool 层、skill 层的多种改进机制。论文把这六种机制统一命名为「self-improvement loops」，并分析了每种机制适用的场景和失效条件 ^[entities/hermes-agent-self-evolving]。

这个方向和 [[concepts/ai-self-improvement-bootstrapping|AI self-improvement bootstrapping]] 有密切关系——bootstrapping 关注的是「AI 改进 AI 的基础能力」，而 self-improvement loops 关注的是「agent 如何从自己的运行经验中持续学习」。两个方向合并，代表了 AI 系统从「静态部署」到「动态演化」的范式转变。

### 范式细节

**反思 → hypothesis** 是最直接的自改进机制：每次任务完成后，agent 主动问自己「我哪里做得不好？下次应该怎么做？」，把反思结果写成结构化的 hypothesis。典型实现：在任务结束后的 system prompt 末尾追加一条「改进建议」，或者把建议写入专门的 memory 条目供下次检索。效果最好的是「具体、可操作」的 hypothesis（如下次遇到同类任务，先问用户确认需求再开始写代码），而不是模糊的目标（如下次要更仔细）^[entities/agent-self-improvement-six-mechanisms]。

**改 prompt / skill 而非改模型**是自改进机制的设计原则。相比修改模型权重（需要 GPU、回滚困难、验证周期长），修改 prompt 的成本极低（改一行文字即可）、回滚容易（版本控制即可）、验证快速（跑一个测试任务即可）。这个原则在实际工程中非常重要：它把「改进」的门槛从「模型训练团队」降到了「任何能写 prompt 的工程师」。Skill 作为多个相关 prompt 的集合，是比单个 prompt 更稳定的改进单元——一个 skill 可以包含数十个相关 prompt，但通过同一个版本号管理，降低了碎片化风险。

**人类作为最终验证者**是防止自改进跑偏的保障机制。完全无人监督的自改进可能导致「reward hacking」——agent 发现某种「取巧」的方式可以在评估指标上得高分，但实际任务质量下降。引入人类验证者（human-in-the-loop）意味着：agent 提出的改进 hypothesis 需要人类批准后才能生效。这个机制在实践中落地成本高（需要人工审核流程），所以多数系统在初期选择「宽松模式」——改进自动生效，只在检测到异常时才触发人工审核。

**6 种机制**的具体分析：reflection（任务后反思生成 hypothesis）、skill 累积（把成功的任务模式固化为可复用的 skill）、memory 重写（修改历史记忆中不准确或过时的内容）、tool 优化（改进 tool 的参数设计或用更好的 tool 替换）、prompt 蒸馏（把复杂任务的 prompt 经验浓缩到更简洁的版本）、fine-tuning trigger（当发现 prompt 层无法解决的问题时，标记数据触发模型微调）。前四种是纯 prompt 层的改动，第五种是 prompt 到 skill 的组织优化，第六种是触发更深层的模型更新。每种机制的触发条件、验证成本、影响范围都不同，实践中通常是前四种组合使用，第五种和第六种作为补充 ^[entities/hermes-agent-self-evolving]。

### 局限与反对声音

自改进循环最大的风险是「改进方向的主观性」。Agent 改进自己的 hypothesis 是基于它自己对这个任务的理解，但这个理解可能本来就是错的——一个对代码风格有误解的 coder，写出来的「改进建议」可能让代码风格变得更差而不是更好。没有客观标准来验证「agent 认为的改进」是否真的是改进，这就是为什么 human-in-the-loop 机制虽然成本高但目前还无法完全被取代。

第二个系统性风险是「改进的累积偏差」。每次改进都是在当前状态上的增量调整，如果初始状态存在偏差，后续的改进会在偏差基础上叠加，最终导致系统行为严重偏离预期方向。这个问题和 [[concepts/catastrophic-forgetting|catastrophic forgetting]] 类似——正向反馈循环会放大初始误差。解法是定期「重置」——放弃所有累积的改进，从头开始，类似于模型训练的 reset。

第三个批评是「自改进的收益递减」。当 agent 已经把 prompt 优化到 local optimum 时，继续运行自改进循环只会产生微小的、在统计上不可信的调整，反而可能因为过度适应特定任务分布而导致泛化能力下降。实践中，自改进机制通常在 agent 上线初期最有价值（从 60 分提升到 80 分），到了平台期后改进收益急剧降低。

### 现实案例

[[entities/hermes-agent-self-evolving|Hermes Agent]] 实现了六种机制中最完整的版本。它的 self-improvement loop 运行在每个任务完成后：首先执行 reflection（生成 hypothesis），hypothesis 写入 skill memory；如果同一个类型的任务连续 N 次成功，agent 自动把这次成功的 prompt 蒸馏为一个可复用的 skill；当某个 skill 被调用 M 次后，agent 检查是否可以用一个更通用的版本来替换多个专用 skill（skill 合并）；当发现某类问题反复出现但 prompt 调整无效时，agent 生成一个 fine-tuning dataset 的候选样本，提交给人类专家审核。

skill acquisition 机制在这个循环中扮演核心角色：agent 不仅要学会新技能，还要能够识别当前 skill 库的空白，主动提出「我需要学习 X 技能来解决这类问题」。这个主动提出需求的能力比被动接受改进建议要高级得多，也是 Hermes Agent 和传统 rule-based agent 在自改进能力上的本质区别。

## 在 wiki 中的关联

- [[entities/agent-self-improvement-six-mechanisms|Agent 自改进六机制]]
- [[concepts/ai-self-improvement-bootstrapping|AI self-improvement bootstrapping]]
- [[entities/hermes-agent-self-evolving|Hermes Agent 自进化]]
- skill acquisition
- [[concepts/hermes-agent-skill|Hermes Agent skill]]

## 进一步阅读

- [[entities/agent-self-improvement-six-mechanisms]]
- [[concepts/ai-self-improvement-bootstrapping]]
- [[entities/hermes-agent-self-evolving]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
- [[moc/loop-engineering|Loop Engineering]]

> [!contradiction] 参见 [[raw/articles/arbor-hypothesis-tree-research-agent-2026|Arbor（假设树科研 agent）]] 持相反观点：经验**保持具体、不做抽象**、per-session 存储、复用需人工同意——与「连续 N 次成功自动蒸馏为 skill、M 次后合并通用化」的路线直接对立。Arbor 的立场对应 [[concepts/agent-memory-lifecycle-philosophies|Memory 生命周期哲学]] 中「skill distillation 最高效固化错误」的警告。综合分析见 [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 涌现观点·观点一（经验抽象度）]]。
