---
title: "用 100 GRPO 步微调 350M 模型：结构化输出合规（IFStruct）"
created: 2026-09-04
updated: 2026-11-22
type: entity
tags: [llm, post-training, grpo, rl, trl, structured-output, fine-tuning]
sources: [raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 用 100 GRPO 步微调 350M 模型：结构化输出合规（IFStruct）

生产中最常见的 LLM 任务之一，是让模型可靠返回符合指定格式与 schema 的可解析输出——即结构化输出合规。多数 benchmark 却把它折进更宽泛的推理/抽取得分中，而非单独测量。本文给出一个完全公开、低成本的配方：用 Group Relative Policy Optimization（GRPO）和 TRL 库对 LFM2.5-350M 微调，把 IFStruct 上结构化合规从 **22.6% → 29.7%**。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md]

## 核心配方

- **模型**：LiquidAI LFM2.5-350M（350M 参数小模型）。
- **方法**：GRPO（Group Relative Policy Optimization），RL 训练。
- **库**：TRL。
- **成本**：约 500 个样本 + 100 训练步，小到免费档 Colab/Kaggle GPU 可跑。
- **基准**：IFStruct（IFStruct-v1.0）。
- **可复现**：notebook 已开源（GitHub cookbook），规范对齐 GRPO + TRL 的完整管线。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md]

> 注意：该训练管线并非训练 IFStruct RL 模型的原始管线，本 notebook 目的是展示任务特定微调能让小模型追平远大于它的模型，而非复现原文 benchmark 分数。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md]

## 意义

- 结构化输出是"能否接入下游系统"的前提，独立于通用推理能力测量。
- 小模型（350M）+ 轻量 RL（GRPO）即可显著提升格式合规，说明任务特定微调对小模型的性价比极高。
- 配方完全公开且可复现，是 post-training 的入门级 RL 实践样本。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md]

## 深度分析

### 基线错误分布：形式失败而非能力失败

在 MacBook（Apple M5 Max、36GB 统一内存）上用 llama.cpp 以 BF16 GGUF 服务基线模型，复现出 452/2000 通过（22.6%），与 IFStruct 官方博客报告的 21.1% 基本一致，作者即以本地结果作为同 serving 栈对比的基线。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:44-56,142] 按格式拆分，JSON 仅 18.0% 而 YAML 有 27.2%；按顶层结构拆分，wrapper key 28.5% 而 bare list 只有 16.6%。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:95-101] 错误榜由形式类错误主导：required field missing 高达 7228 次、wrong item count 738 次、type mismatch 540 次，远超其他类别。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:129-139] 实体类型间差异极大：recipe 低至 4.3%，event_ticket_booking 高达 45.8%。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:103-127] 失败模式是"没按 schema 长出正确形状"，而不是"不会回答"。

### 训练数据：用增广把训练分布拉向评测分布

训练数据用 `nvidia/Nemotron-RL-instruction_following-structured_outputs`，每条 prompt 配一个目标 JSON Schema 和期望字段数，只取约 500 条。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:148-150] 因为 Nemotron 分布与 IFStruct 评测分布不同，作者做了两处定向增广来弥合差距：40% 的样本追加"把输出放进 fenced code block"的指令，让模型学会服从格式指令而非永远输出裸 JSON；互斥的另外 20% 被转成顶层 array 任务（schema 包进 `array` 并要求 item count），专训 bare-list 输出与数量合规。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:152-155]

### LoRA 配置与三个可验证奖励函数

在 LFM2.5-350M 上挂 LoRA 适配器：r=16、alpha=32，target_modules 针对其混合注意力/卷积架构的 LFM 专有模块名（q/k/v/out/in_proj 与 w1/w2/w3），只训练约 6M 参数、占全模型的 1.66%。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:157-174] 奖励由三个 [0,1] 函数加权求和：json_format_reward 按请求形式给满分 1.0、形式错但可解析给 0.2、不可解析给 0.0；field_count_reward 顶层字段数精确匹配得 1.0、随偏差线性衰减；schema_validation_reward 统计每条约束违规并按 required-key 覆盖率给部分分。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:176-183] 组合权重 [1.0, 0.5, 2.0]——schema 验证权重最高，对应最难也最值钱的约束。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:184]

### 训练超参与结果：增益精准落在训练目标上

GRPOConfig 的关键项：max_steps=100、每 prompt 组采 8 个生成、learning_rate 5e-5、warmup 10 步、temperature 1.1（更热的采样保持组内多样性）、beta 0.01 的 KL 惩罚、max_completion_length 1024（给嵌套 JSON 留空间），整体按免费档 16GB GPU 设计。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:186-209] 训练全程三个奖励分量同步上升，KL 在 warmup 后抬离零点，截断补全比例保持近零。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:212] 训练结束后 merge LoRA 权重并存为单一 checkpoint，转成 BF16 GGUF 即可在 llama.cpp 上继续服务评测。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:214-224] 同 serving 栈前后对比：总体 22.6%→29.7%（+7.1），JSON 18.0%→31.9%（+13.9），bare list 16.6%→29.7%（+13.1），而 YAML 仅 +0.3、wrapper key 仅 +1.2。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:322-330] 提升几乎全部落在两个增广方向对应的指标上；最终成绩仍低于 Qwen3.5-2B 的 33.15%，但已接近数倍于它的模型。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:332]

### 结果的另一面：错误并未消失

微调后错误榜的结构值得注意：required field missing 仍是第一位且有 7331 次（基线 7228 次），wrong item count 反而从 738 次升到 890 次，type mismatch 也从 540 升到 555，平均延迟从 1453ms 微升到 1518ms。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:268-320] 也就是说，594/2000（29.7%）的通过数提升主要来自格式与指令遵循轴（JSON 形式、bare-list 包装），而不是消除了 schema 违规本身。按实体类型看同样明显：log_parser_examples 29.2%→45.8%、support_ticket_batch 37.0%→49.3% 大涨，但 camera_review、gpu_review、recipe 仍停在 6-10%。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:283-307] 剩下的差距是内容与语义层面的，纯形式奖励够不着。

## 实践启示

1. **错误分布先于奖励设计**：基线错误榜里 required field missing（7228 次）等纯形式错误占绝对主导，说明大量"失败"不需要更强的推理能力，只需要可验证的形式奖励就能修复。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:129-139]
2. **奖励权重向最难的约束倾斜**：三个奖励中 schema_validation 拿到 2.0 的最高权重，简单格式分（可解析性）只给低权重，避免模型停在"能解析"而不再逼近 schema 合规。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:176-184]
3. **数据增广是对齐评测分布的杠杆**：40% fenced-block 指令与 20% 顶层 array 的定向增广，几乎一对一解释了 JSON +13.9 与 bare list +13.1 的提升——训练分布不覆盖的输出形状不会自己长出来。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:152-155,324-330]
4. **小模型 RL 的成本结构极低**：约 500 样本、100 步、LoRA 只训 1.66% 参数，免费档 16GB GPU 可跑训练；评测侧 llama.cpp + GGUF 在笔记本上即可完成，训练与评测完全解耦。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:32-35,172-174,188]
5. **格式收益不自动泛化**：YAML 与 wrapper key 几乎不动，增益集中在被训练的格式上；要让某种输出形状合规，就得把它显式造进训练数据，而不是指望 RL 自己泛化。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:326-330]
6. **复现路径完整可走**：notebook 开源在 Liquid4All cookbook，benchmark 代码与数据集（LiquidAI/ifstruct-v1.0）均公开，是 RLVR 式任务特定微调的低门槛入门样本，可与 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 可验证推理]] 和 [[concepts/reinforcement-fine-tuning-rft|强化微调 RFT]] 的框架互相印证；LFM2.5 家族另见 [[entities/liquid-ai-lfm2-5-230m|LFM2.5-230M]] 与 [[entities/liquid-ai-lfm2-5-2-6b-agentic-on-device|LFM2.5-2.6B]]。^[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct.md:146,336-338]

## 相关

- [[concepts/grpo-policy-optimization-2026|GRPO 策略优化]]
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演化]]
- [[concepts/reinforcement-fine-tuning-rft|强化微调 RFT]]
- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 可验证推理]]
- [[entities/aws-grpo-rlvr-sagemaker-math-reasoning|AWS GRPO-RLVR 数学推理]]
- [[entities/5-useful-things-post-training-textbook-nathan-lambert|Post-training 手册要点]]

→ [[raw/articles/fine-tuning-350m-model-structured-outputs-grpo-trl-ifstruct|原文存档]]