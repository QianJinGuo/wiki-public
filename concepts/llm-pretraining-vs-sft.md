---
title: "LLM Pretraining vs SFT — 预训练与后训练的数据分布差异"
type: concept
tags: [llm-pretraining-vs-sft]
related:
  - [[entities/baidu-wenxin-post-training-evolution|百度文心大模型后训练进化（ERNIE 3.0→5.0）]]
  - [[entities/aws-grpo-rlvr-sagemaker-math-reasoning|GRPO+RLVR: Qwen数学推理3.7x提升的工程细节]]
  - [[entities/llm-post-training-full-guide|LLM Post-Training全景指南：从RLHF到GRPO再到AgenticRL]]
  - llm
  - pretraining
  - sft
  - post-training
  - data-distribution
  - rlhf
  - grpo
  - catastrophic-forgetting
created: 2026-05-09
updated: 2026-08-29
---

# LLM Pretraining vs SFT — 预训练与后训练的数据分布差异

预训练（Pretraining）和监督微调（Supervised Fine-Tuning, SFT）是 LLM 训练的两个关键阶段，数据分布差异是许多退化问题的根源。理解两者之间的本质区别，是掌握后训练 pipeline 设计的前提。

## Overview

预训练阶段模型在海量互联网语料上通过**下一个 token 预测**任务学习通用语言能力和世界知识；SFT 阶段则在此基础上使用**高质量、带标注的对话/任务数据**教会模型遵循指令和完成特定任务。两者的核心差异不仅体现在数据规模上，更体现在**数据分布**的本质区别。

预训练语料覆盖广泛但噪声多，模型学到的分布是"互联网文本的原生分布"；SFT 数据则经过精心筛选和格式控制，代表的是"人类期望的模型行为分布"。当两者分布差异过大时，模型在 SFT 后会出现**分布漂移（distribution shift）**，表现为预训练阶段学到的某些能力退化。

这种漂移是导致**灾难性遗忘（Catastrophic Forgetting）**的重要原因之一，也是设计后训练 pipeline 时必须正视的核心挑战。

## Data Distribution Differences

### 预训练数据分布特征

预训练数据具有以下核心特征：

- **规模庞大**：通常在 trillion 级别的 tokens，涵盖网页、书籍、代码、对话等多种来源
- **噪声丰富**：包含拼写错误、语法不规范、重复内容、过期信息等
- **分布异构**：不同领域（学术论文、社交媒体、技术文档）的分布差异巨大
- **长尾分布**：头部高频 token 占据大部分概率 mass，但长尾知识对模型能力至关重要
- **续写任务**：目标函数是下一个 token 预测，模型学习的是"如何续写"而非"如何回答"

预训练阶段模型通过大规模续写任务习得了丰富的世界知识和语言多样性，但也学会了许多不符合人类期望的输出模式（如冗长、跑题、有害内容）。

### SFT 数据分布特征

SFT 数据与预训练数据的分布存在根本性差异：

- **规模有限**：通常在百万到十亿级别 tokens，质量远高于预训练数据
- **格式严格**：通常包含明确的指令模板（如 ChatML、Alpaca 格式），对话结构清晰
- **噪声低**：经过人工或自动化筛选，不良内容比例极低
- **分布集中**：聚焦于特定任务场景（问答、摘要、代码生成等）
- **响应期望**：模型学习的是"如何给出符合人类期望的响应"

SFT 数据天然倾向于**格式规范、响应简洁、符合人类价值观**的输出，这与预训练中"自由续写"的分布形成张力。当 SFT 数据分布过度偏离预训练分布时，模型会逐渐遗忘预训练阶段习得的长尾知识和多样化的表达能力。

### 分布漂移的具体表现

**Token 退化（Token Degradation）**是分布漂移的典型症状：预训练中常见但 SFT 数据中罕见的某些 token 模式在微调后退化。MiniMax 案例揭示了这一问题——模型在特定领域任务上表现优异，但预训练阶段学到的某些通用知识或表达能力明显下降。 ^[raw/articles/minimax-token-degradation-jiqia.md]

分布漂移还会导致以下问题：

- **能力窄化**：模型在 SFT 分布内表现良好，但跨领域泛化能力下降
- **格式依赖**：过度适应 SFT 的模板格式，生成灵活性降低
- **知识遗忘**：预训练中记忆的事实知识在 SFT 后难以检索
- **风格偏移**：从多样化的互联网文本风格收敛到单一的标准应答风格

## Post-training Pipeline（RLHF/GRPO）

SFT 只是后训练的第一步，现代 LLM 后训练通常包含完整的 pipeline：SFT → RLHF/DPO → GRPO/RLVR 等阶段。百度文心后训练演进和 AWS GRPO 数学推理案例展示了两种典型的后训练工程范式。

### 百度三阶段 RL 后训练飞轮

百度 ERNIE 3.0→5.0 的后训练将目标分解为**三阶段异步训练**：

1. **有用性阶段（RLHF）**：奖励模型驱动生成质量改善，模型学习生成更有帮助的响应
2. **安全性阶段（DPO）**：专家红队迭代，让模型学习安全、合规的回复方式
3. **诚实性阶段（加固 DPO）**：巩固安全对齐，防止越狱攻击和有害输出

百度明确反对「多目标同阶段混合」，主张**优先级递增**（安全 > 有用）。三种目标不可同阶段混合，因为 Reward Model 的条件分布假设决定了当多个目标存在耦合时，单一验证器无法同时捕捉安全约束与有用性信号的最优梯度方向。分阶段训练的代价是训练周期变长，但百度通过**异步飞轮**（不同阶段用不同数据池独立演进）来摊薄这一成本。

RL 后训练形成飞轮闭环：RL 训练改善生成 → 丰富样本池 → 提升验证器 → 更精准 reward signal → 回馈 RL 训练。

### AWS GRPO+RLVR 双奖励系统

AWS SageMaker 上的 GRPO（Group Relative Policy Optimization）+RLVR（Reward Model-based RL with Verifiable Rewards）展示了另一种后训练范式，特别适合数学推理等可自动化验证的任务。

**双奖励系统设计**：

- **format_reward（0.5分）**：驱动格式规范，确保输出可解析
- **correctness_reward（1.0分）**：驱动答案正确性，确保最终结果正确
- 两者解耦避免 reward hacking——任一维度作弊都会导致总奖励低于理论最大值 1.5

**GRPO vs PPO**：传统 PPO 需要大量样本估算 baseline（critic 网络），而 GRPO 在每个 prompt 的 group 内估算 baseline，大幅降低方差。Group 内多个 completion 互相比较，最优的获得正向更新、最差的获得负向更新，policy gradient 直接来自相对排名而非绝对 value 估计。

**8-shot 阈值激活规律**：实验发现 shot 数对 GRPO 效果呈非线性——0-shot(6%) 和 2-shot(3%) 反而低于 base(11%)，4-shot 跳到 33%，8-shot 达到峰值 41%。过少的 shot 无法建立有效的 group baseline，GRPO 训练形成的推理 pattern 需要足够多的 group 比较样本才能激活。

**RLVR 泛化条件**：RLVR 只适合输出可客观验证的领域（数学推理、代码生成、符号推理），不可验证领域（开放式写作）不适用。

### 后训练 pipeline 的设计原则

综合两个案例，后训练 pipeline 设计遵循以下原则：

- **分阶段优于混合**：多目标分离训练，避免 Reward Model 歧义
- **异步飞轮摊薄成本**：不同阶段用不同数据池独立演进，避免相互干扰
- **可验证领域优先 RLVR**：答案可客观验证时优先使用 GRPO+RLVR，成本低、效果好
- **格式与内容解耦**：双奖励系统分别驱动格式和正确性，防止单维度 reward hacking
- **group size 至关重要**：GRPO 需要足够大的 group（建议 8-shot 以上）才能激活有效推理 pattern

## Relationship to Catastrophic Forgetting

### SFT 导致灾难性遗忘的机制

灾难性遗忘（Catastrophic Forgetting）是指模型在学习新任务时大幅遗忘之前习得的知识和能力。SFT 阶段是灾难性遗忘的高发期，原因在于：

**参数漂移假说**：SFT 的梯度更新直接作用于模型参数，当新任务的分布与预训练分布差异过大时，模型参数被"拉向"新分布，同时远离预训练知识所在的参数区域。

**分布压缩假说**：预训练模型的知识分布是宽泛的、混沌的；SFT 将其压缩到特定任务的窄分布内。在压缩过程中，与新分布无关的知识节点被弱化甚至删除。

**任务干扰**：当 SFT 数据包含多种任务类型时，模型需要在多个任务目标之间平衡，但单一模型容量有限，多任务学习必然导致任务间的性能权衡（task interference）。

### RLHF/GRPO 阶段的遗忘风险

RLHF 和 GRPO 阶段同样存在灾难性遗忘风险，但其机制与 SFT 不同：

- **Reward hacking 驱动的遗忘**：模型在最大化 reward 的过程中可能学会"作弊"——生成高 reward 但实际上缺少深度理解的形式化答案，导致真正的推理能力退化
- **安全对齐挤压有用性**：百度强调安全、有用、诚实三个维度不可同阶段混合，因为安全对齐（防止有害输出）可能会挤压有用性表现空间，导致模型过于保守
- **过度优化到退化**：当 RL 训练轮次过多时，模型可能过度优化特定 reward 模式，丧失预训练阶段的多样性和泛化能力

### 缓解策略

基于现有实践，缓解灾难性遗忘的策略包括：

**分阶段训练（百度经验）**：按优先级递增（安全 → 有用 → 诚实）分阶段独立训练，每阶段只聚焦一个目标，避免多目标混合导致的 Reward Model 歧义。

**混合预训练数据**：在 SFT/RLHF 数据中混入一定比例的预训练语料，保持模型对原生分布的接触频率，对抗分布压缩。

**Rehearsal 方法**：定期用预训练数据的一部分样本进行"回忆训练"，保持模型对预训练分布的敏感性。

**LoRA/QLoRA 微调**：使用低秩适配器而非全参数微调，将参数更新限制在低维子空间，减少对预训练参数的干扰。AWS 案例中使用 QLoRA（4bit 量化 + LoRA rank=16）在 0.5B 模型上取得良好效果。

**Early Stopping**：在验证集上监控预训练阶段关键能力的保留程度，在性能开始退化前停止 SFT/RLHF。

**双奖励解耦（AWS 经验）**：format_reward + correctness_reward 双奖励系统分别驱动格式规范和答案正确性，避免单一 reward 驱动下模型走向极端。

## Related Entities

- [[entities/baidu-wenxin-post-training-evolution|百度文心大模型后训练进化（ERNIE 3.0→5.0）]] — 三阶段 RL 后训练飞轮、TransNets 架构、Agent 能力训练
- [[entities/aws-grpo-rlvr-sagemaker-math-reasoning|GRPO+RLVR: Qwen数学推理3.7x提升的工程细节]] — 双奖励系统、8-shot 阈值激活、QLoRA 配置
- [[entities/llm-post-training-full-guide|LLM Post-Training全景指南：从RLHF到GRPO再到AgenticRL]] — 后训练技术全景对比
- [[entities/minimax-token-degradation-jiqia|MiniMax Token 退化研究]] — SFT 数据分布导致 token 退化的具体案例
- [[concepts/llm-tokenizer|LLM Tokenizer]] — 分词器决定了词表粒度，影响预训练和 SFT 的分布匹配
- [[concepts/scaling-laws|Scaling Laws]] — 训练数据量与模型能力的关系，指导预训练和 SFT 的规模决策
- [[entities/skill-design-patterns|Skill Design Patterns]] — Anthropic 14 模式中的 RL 后训练相关策略对比

## 来源

- MiniMax 案例揭示了 SFT 数据分布与预训练差异导致的 token 退化问题（2026-05-09）
- 百度 ERNIE 后训练三阶段分治策略（2026-05-01） ^[raw/articles/baidu-wenxin-post-training-evolution.md]
- AWS GRPO+RLVR 数学推理工程细节（2026-05-08） ^[raw/articles/aws-grpo-rlvr-sagemaker-math-reasoning.md]

## 关联实体

**上游依赖**:
- [[entities/baidu-wenxin-post-training-evolution]] — 提供基础理论/方法
- [[entities/aws-grpo-rlvr-sagemaker-math-reasoning]] — 提供基础理论/方法

**下游应用**:
- [[entities/llm-post-training-full-guide]] — 具体应用场景
- [[entities/baidu-wenxin-post-training-evolution]] — 具体应用场景
- [[entities/aws-grpo-rlvr-sagemaker-math-reasoning]] — 具体应用场景

**平行协作**:
- [[entities/llm-post-training-full-guide]] — 替代/补充方案
- [[entities/minimax-token-degradation-jiqia]] — 替代/补充方案
- [[entities/skill-design-patterns]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-1-llm-principles|Layer 1 Llm Principles]]
