---
title: "从 BERT、GPT 到 Jev：大模型架构演进与输出范式的分化"
created: 2026-10-20
updated: 2026-10-24
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

## 深度分析

### 不变量：一次前向，把序列压成每个位置的隐状态

块内算式从 2017 年至今没动过：一次注意力做位置间交换，一次前馈做逐位置变换。真正的常量在更抽象一层——**给定序列，模型用固定次数的前向把它压成每个位置的隐状态；能力由压缩质量决定，任务由「从哪个位置、读几次」决定**。注意力取代循环后，「第 n 个词要等前 n−1 个」的约束消失，同一套参数才能被任意次、任意形式地查询。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md:27-29] 创新因此只剩两类落点：让隐状态更便宜（常数因子），或改读出的位置与次数（复杂度阶）。

### 分水岭在读出侧：串行采样 vs 并行读出

判据只有两条：是否因果掩码（能看多远）、是否逐 token 生成（答案要跑几遍网络）。BERT 与 GPT 差在第一条，Jev 与所有 LLM 差在第二条，两条正交：因果却不生成、双向却仍生成，都存在。

串行采样的成本随输出长度线性涨，第 t 步依赖第 t−1 步。更关键的是**概率长在哪个空间**：LLM 的概率在词表上，是「下一个 token」而非「陈述为真」，要从前一个得到后一个须先把判断外化成 CoT 再解析回标签，语义编解码一个来回，误差与延迟都随链长增长。Jev 把分布直接定义在**任务局部选项空间**上：255 个选项与 2 个选项耗时相同，因为选项是并行分支而非生成长度。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md:144-152] JSON schema 不解决问题——它只约束字符串形状，逐 token 生成一步没少。

### 为什么删掉生成比让生成更快便宜

decode 访存受限：每 token 要搬全部权重加整份 KV cache，算术强度个位数 FLOP/byte，而加速器设计点在数百。成本正比于输出 token 数，kernel 优化、投机解码、大 batch 只压低斜率，不改 O(L)；prefill 相反，计算密集、完全并行、可摊薄。去掉 decode 等于把「访存受限的 L 步串行」换成「算力受限的 1 步并行」，两个数量级由此而来，不是来自更聪明的算子。组件与范式因此必须一起读：

- **KV cache 是 decode 的产物**：前提是「后面还要复用」，单次前向零收益——Gated DeltaNet 的固定状态、V4.1 后 20 层 KV 投影都只为 decode 服务。反向则是双向可见与缓存复用冲突——Decoder-only 赢下主流靠它，decision model 甘愿付它。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md:57-57]
- **MoE 收益不对称**：它解耦总参数与每 token 算力，对 decode 收益最大，但路由要求 batch 内 token 散到不同专家，单请求摊不平 all-to-all；判断模型的 batch 被「选项 × 问题」填满，却更可能选全激活密集结构（Laya 400M 密集），瓶颈不在权重搬运。见 [[concepts/moe-mixture-of-experts-2025|MoE]]。
- **注意力变体看瓶颈**：线性/稀疏注意力压 O(S²) 与 KV 增长，服务长 prefill 与长 decode；判断模型输入短、S 小，预算该投读出头与校准。

### Laya 排除了什么，又把什么留在黑箱里

Laya 是一次**单变量替换**：输出方式固定（选项前 `[MASK]` 读分、选项内 softmax、以校准为目标），只把底座从推断中的 LLM 级因果 Transformer 换成确定的 4 亿双向 encoder。由此排除三件事：读出方式不稀缺（33 ms 可复现）、架构新颖性不必要（ModernBERT 加小头即够）、把选项写进序列而非参数 400M 也走得通。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md:158-164]

没排除的恰是结论依赖的那一项：Laya 最好成绩来自**针对评测集微调之后**，Jev 是零样本，两者不同实验条件，故「差距在底层网络」是推断而非受控结论；把参数量与「预训练目标 + RLCD 校准数据」打包成「LLM 级理解能力」，是全文最弱一环。Laya 概率整体偏高亦然，反证校准依赖对任务的把握。局限由此显形：Jev 结构未公开、全靠行为反推；n=1 定位不了阈值（4 亿不够，40 亿呢？）；三原语覆盖不了数值与多步推理；六个 `noul` 不增延迟，state 却重复计六次 token。

## 实践启示

1. **先量输出长度分布再选范式。** P99 输出超几十且答案可枚举 → O(1) 前向优于任何 decode 加速；答案须是未见字符串 → 仍走 Decoder-only。
2. **把任务指定从参数搬到请求。** 双向 encoder + 每选项读出头 + 选项内 softmax + 校准目标：换判断不必重训，代价是零样本能力被底座锁死。
3. **别在判断模型上照搬生成模型的优化。** KV 压缩、线性注意力、投机解码对单次前向零收益；预算改投读出头、选项位置偏置与校准数据。
4. **校准单独度量、训练、验收。** RLHF 会破坏校准（GPT-4 报告：预训练校准良好，后训练后明显变差），按阈值分流就用 ECE / Brier；采样投票不算替代品，那是把 decode 成本乘上采样数。^[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026.md:68-68]
5. **拆分粒度是成本问题。** 原子问题并行摊薄延迟，但 state 重复计 token；先用「问题数 × state 长度 × 单价」算边界，再定拆几层，组合逻辑放代码。
6. **两种范式共享序列化与阈值层。** 判断模型不能写、不能思考、不能进 agent 循环；state 编解码、置信度分流、人工兜底做成共用中间件，免得维护两套链路。

## 关联

- [[concepts/transformer-architecture|Transformer 架构]]
- [[concepts/attention-mechanism|注意力机制]]
- [[concepts/inference-optimization|推理优化]]
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]
- [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026|Jev 语义决策组件（京东技术）]]
- [[entities/jev-rlcd-machine-heart-report-2026|Jev RLCD 解读（机器之心）]]

→ [[raw/articles/llm-architecture-evolution-bert-gpt-jev-tencent-2026|原文存档]]
