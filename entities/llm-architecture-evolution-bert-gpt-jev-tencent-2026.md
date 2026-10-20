---
title: "从 BERT、GPT 到 Jev：大模型架构演进与输出范式的分化"
created: 2026-10-20
updated: 2026-10-20
type: entity
tags: [transformer, architecture, bert, gpt, jev, laya, decision-model, moe, kv-cache]
sources: [raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026]
confidence: 0.8
---

# 从 BERT、GPT 到 Jev：大模型架构演进与输出范式的分化

腾讯技术工程这篇长文把 2017 年 Transformer 到 2026 年 Jev 的演进拆成**「怎么读输入」**与**「怎么产生输出」**两条线，并给出一个可复用的判断框架：主流模型的骨架从未变过——ChatGPT 与推理模型改变的是**训练方式**，DeepSeek 与 Qwen 改变的是**内部组件**，而 Jev 改变的是**输出方式**。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md]

## 骨架：三次变化，一个不变

- **2017 原始结构**：机器翻译任务的 Encoder-Decoder 两段式；贡献不在两段式（seq2seq 已如此），而在用注意力完全取代循环结构，使所有位置可并行计算——这是后来扩展到千亿参数、GPU 得以充分利用的工程前提。Encoder 双向可见，Decoder 因果可见。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md]
- **2018–2020 三条路线分化**：Encoder-only（BERT，填空预训练 + 分类头）、Decoder-only（GPT，逐词生成）、Encoder-Decoder（T5 系）。
- **2020 后 Decoder-only 成为主流**：统一的「读入 + 续写」接口让同一模型可做问答、代码、工具调用，扩展性压过架构优雅性。
- **2022–2025 后训练与推理模型**：能力提升从架构转向训练方式（RLHF/RLVR、长 CoT、推理时计算）。

## 2026 前沿模型的两个改动点

1. **前馈网络改 MoE**：每层 FFN 拆成数百个子网络，每 token 只激活少数几个，总参数与每 token 算力解耦。DeepSeek-V4.1-Flash 总参数 552B、每 token 激活 16B；Qwen3.8-27B 是 27B 全激活的密集模型。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md]
2. **KV cache 压缩**：长上下文与 Agent 场景下缓存占用超过权重本身。DeepSeek 走低维潜向量压缩 + 稀疏选择，并在 V4.1-Flash 中把前 20 层作为因果 encoder、后 20 层的 KV 由其投影得到，**prefill 只需跑一半网络**；Qwen 自 Qwen3-Next 起把每 4 层中的 3 层换成线性注意力（Gated DeltaNet），以固定大小状态替代逐 token 缓存。详见 [[concepts/inference-optimization|推理优化]]。

## Jev：去掉「生成」的那一半

Jev（TypeSafe AI，2026-09）不生成文本，而是针对给定材料回答三类限定问题——**choice**（从若干选项选一个）、**score**（在若干等级上评分）、**noul**（判断陈述是否成立），并直接返回各答案的概率。常见 LLM 完成一次回答需数秒到数分钟，Jev 完成一组判断在**百毫秒**量级；多数查询约 100 ms，同一请求内增加问题几乎不增加延迟；每百万输入 token $0.042、输出免费；仅文本、以英语为主、不开源。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md]

返回结构上，除了答案还给出完整概率分布与 `confidence`（由分布形状算出：均匀为 0、完全集中为 1），官方建议按置信度分流——高置信度自动执行，低置信度交人。

方法论主张是**代码掌握控制流，只在需要常识判断处调用模型，且每个问题拆到原子**：垃圾邮件识别不问「是不是垃圾邮件」，而是同一封邮件并行问六个 noul（是否索要凭据、是否声称意外奖励、是否制造时间压力、发件人与域名是否冲突、链接域名是否冲突、链接文字是否掩盖目标），返回六个概率由代码加权合成风险分数并设阈值。这与 [[concepts/agent-orchestration-patterns|Agent 编排模式]] 中「确定性控制流 + 局部模型判断」的分工一致。

## 对照实验：Laya 说明难的不是读出方式

Jev 发布三天后，Convai Innovations 以 Apache 2.0 开源 **Laya**（「33 ms 的开源 System 1 决策引擎」），被视为开源版 Jev。其结构可直接从代码读出：底层是 ModernBERT-large（4 亿参数**双向 encoder**，即 Encoder-only 路线的当代版本）+ 小决策头；每个问题拼成一条序列，各选项前放 `[MASK]`，在每个 `[MASK]` 位置取隐向量打分，对同一问题的选项做 softmax 得到概率分布——三种原语与 Jev 相同，训练目标同为概率校准，单次最多 512 token。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md]

结论因此很有价值：**「一次前向按选项读概率」这个输出方式并不难，用开源 encoder 加读出头即可复现；难的是其下的网络**。同样输出方式装在 4 亿参数 BERT 上，零样本判断远不及 Jev（Laya 最好成绩来自针对评测集微调后）；Jev 与开源实现的差距不在读出，而在「不训练就能读懂任意问题」的 LLM 级理解能力。

## 判断框架

- 需要**自由文本生成** → 用 LLM（Decoder-only）。
- 需要在**有限选项间做判断**、要求低延迟与可校准概率 → 决策型模型（Jev 类）；有固定判断场景且可微调，Laya 类开源方案可用。
- 需要**零样本处理任意判断问题** → 仍要 LLM 级底层网络。

一句话：Jev 不是 LLM 的替代，而是同一底层网络在另一类任务上的另一种用法。相关实体见 [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026|Jev 语义决策组件]] 与 [[entities/jev-rlcd-machine-heart-report-2026|Jev RLCD 机器之心解读]]。

## 局限

对 Jev 内部结构的描述属作者基于零样本表现的**推断**（参数量与结构未公开），文中已明示；Laya 的公开概率整体偏高（声称把握高于实际正确率），需自行校准后使用。

## 关联

- [[concepts/transformer-architecture|Transformer 架构]]
- [[concepts/attention-mechanism|注意力机制]]
- [[concepts/inference-optimization|推理优化]]
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]
- [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026|Jev 语义决策组件（京东技术）]]
- [[entities/jev-rlcd-machine-heart-report-2026|Jev RLCD 解读（机器之心）]]

→ [[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026|原文存档]]
