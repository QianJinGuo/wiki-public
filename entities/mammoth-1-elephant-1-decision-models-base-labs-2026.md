---
title: "Mammoth-1 与 Elephant-1：Base Labs 的决策模型技术报告"
created: 2026-10-22
updated: 2026-11-16
type: entity
tags: [model, post-training, decision-model, uncertainty, attention, benchmark, qwen]
sources: [raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026]
confidence: 0.75
provenance_state: extracted
---

# Mammoth-1 与 Elephant-1：Base Labs 的决策模型技术报告

Base Labs（Baseten）2026-10-20 发布技术报告，推出两个决策模型 **elephant-1（9B）** 与 **mammoth-1（27B）**，由 Qwen3.5-9B / Qwen3.5-27B **后训练**而来。mammoth-1 在 Jev Decision Index 0.4 的 208 个模型中排名第一（71.24），elephant-1 是 9B 档最强（超过 Jev 1.14），并在 Artificial Analysis Decision Index 上与上一代最强 d1-ultra 打平（65.2 vs 65.0，mammoth-1 为 65.9）。作者自述两个方法要点：**(a) 全注意力层使用 block-bidirectional attention mask**（在数据固定时优于 causal attention），**(b) 高度可扩展的数据配方**。^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

## 什么是决策模型（decision model）

决策模型接收一个 **state**（JSON 值，如工单或文档，图片/视频以内联引用）与一组 **questions**，对每个问题直接返回答案上的概率分布，不生成自由文本。问题有三种指令类型：^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

- **Choice**：1–255 个选项（可带描述），每题返回各选项概率（和为 1）与 confidence，通常把最高概率重标定使 0 = 均匀猜测、1 = 绝对确定——这是决策模型的规范输出形态。
- **Yes/no**：给出真/假判定说明，返回单个 P(true)。
- **Score**：2–10 档量表（如"客户有多生气"，0 平静 → 4 暴怒），返回每档概率、期望档位（概率加权平均）与 confidence（全概率落在单一档为 1，均匀分布为 0）。

作者给出的四点相对普通 LM 的优势：a) 答案即给定选项之一，**无需解析生成文本**；b) **不生成任何 token**，因此便宜；c) **易于阈值化**——概率足够高才自动执行，否则升级给人；d) 可以把模型**训练成不确定性校准（uncertainty calibrated）**的。普通语言模型虽然也能做同样的判断，但需要解码文本再解析答案，且除非显式提示否则不提供可靠的自身确定性度量。^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

## 数据配方与不确定性感知数据构造

报告方法学章节以"data, data, data"定调：数据来源、数据处理、**对数据处理过程做 looping**、**数据演化（data evolution）**，以及 **uncertainty-aware data construction**（不确定性感知的数据构造）构成方法主体；评测基准为 Jev Decision Index（37 个公开 benchmark + 未披露数量的私有测试）与 Artificial Analysis Decision Index（知识、推理、工具调用与安全评测）。附录 A1 给出每个 Jev Decision Index benchmark 的分项得分。^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

## 谱系位置

decision model 这一 **格式** 由 TypeSafe 于 2026 年 9 月随 Jev 与 System One API 引入（命名取自 Daniel Kahneman 关于大脑"直觉系统"快速决策的描述），此后多家训练并托管决策模型。mammoth-1/elephant-1 是同一族谱上向后训练与数据侧加码的迭代，与 [[entities/liquid-ai-d1-edge-decision-models]] 属同一"决策模型"家族的不同团队路线（d1 系列此前是 Artificial Analysis Decision Index 上的最强档，本报告的 elephant-1 与之打平）。^[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026.md]

## 工程要点摘要

| 项 | 值 / 做法 |
|----|-----------|
| 基座 | Qwen3.5-9B（elephant-1）、Qwen3.5-27B（mammoth-1），后训练 |
| 关键架构改动 | 全注意力层 block-bidirectional attention mask（数据固定时优于 causal） |
| 输出形态 | 每问题一个概率分布 + confidence；无 token 生成 |
| 主要基准 | Jev Decision Index 0.4（37 公开 + 私有）、Artificial Analysis Decision Index |
| 高分 | mammoth-1 71.24（#1/208）、65.9 AA；elephant-1 65.2 AA |
| 对手 | d1-ultra、Jev 1.14、Microsoft-Decision-1、Perplexity Decider v1.1 27B |

## 深度分析

### 读出层设计：把"回答"变成一次分类读出

从方法学细节看，决策模型并不是一个新架构，而是把语言模型改造成一个**结构化分类器**：每个请求由包含系统提示与 state 的公共前缀（x_P），加上每个问题各自的后缀（x_S，含指令与带字母标号的选项）拼成。后缀结尾处设一个**答案位置** t*，最终答案就是该位置上各选项字母 logit 经 LM head 投影后的 softmax 分布。这个设计有两个值得注意的推论：其一，所谓"不生成 token"在实现层面就是"只读一个位置的 logits"；其二，前缀只需前向一次并复用其 KV cache，之后每加一个问题只需对后缀做一次前向——多问题请求的成本随问题数线性增长且斜率很小。这使得"一次 state、几十个问题"的评测式用法在部署上真正可行。

### block-bidirectional mask：部署经济性被编码进注意力掩码

对 Qwen3.5-27B 这类混合架构（每十六个重复单元中三个 Gated DeltaNet 层配一个全注意力层），作者只改动了那十六个全注意力层的掩码：允许 token i 关注 token j 当且仅当 b(j) ≤ b(i)，即前后缀两块划分下的 block-bidirectional mask。它是 prefix-LM 掩码向两块的推广，同时带来两个结构性后果：问题在编码时能看到自己的全部选项（读出需要的上下文），而前缀永远不看到任何后缀，因此 cache 在同一请求的所有问题间完全一致、可共享。在固定数据的消融实验中，该掩码把公开 Jev 子集得分从 44.8 提到 48.6——值得注意的是，这个改动首先是被"cache 必须可复用"的推理成本约束逼出来的，精度提升反而是掩码形状的副产品。Perplexity 的 Decider v1.1 也采用了类似做法，说明这正在成为决策模型的默认架构选择。相关背景见 [[concepts/transformer-architecture]] 与 [[concepts/attention-mechanism]]。

### 数据配方的两段曲线：先堆量、后修补

八版数据混合的得分轨迹清晰地分成两段：Mix 1→4 靠把规模从 25.4 万扩到 419 万（约 16 倍）把 9B 模型的公开 Jev 得分从 45.5 推到 66.0；而 Mix 4→8 只增加了约 48% 的行数，却再涨 8 分，其中 Mix 5 仅多 4% 行数就 +5.5 分，Mix 7 对全部 253 个数据源重审错误标签、答案泄露与格式问题后又 +2.2 分。也就是说，**规模红利用尽之后，数据质量审计成为主导杠杆**。数据构成同样按认知顺序递进：先公开决策数据，再按知识推理、语言、工具自动化、检索分类、艺术品味五个技能维度合成核心数据，之后才补上安全弃权（早期实验暴露决策模型"不会拒绝回答"的短板）与裁判/专家领域数据——这暗示"通用决策模型"所需的能力是逐轮试错中被发现的，而非预先设计的。这一模式与 [[concepts/scaling-laws]] 及 [[concepts/data-quality-framework]] 的讨论相互印证。

### 校准与弃权：用模型自身的置信信号反向构造训练目标

报告中最巧的一处是对"我不知道"能力的构造。标准训练流程下模型几乎从不选择弃权项；作者在消融中发现，最可靠的内部不确定性信号反而最简单——最高单选项概率低于 τ（略小于 0.5）。于是训练时对不确定性数据集随机加入"I don't know"选项、随机移除正确答案：若正确答案被移除或其概率低于 τ，就把 100% 概率分给弃权项，否则保持原分布并给弃权项 0%。配合在答案位置上以交叉熵加 Brier 项（λ=1）作为损失——Brier 项按绝对量惩罚落在错误选项上的概率，从而抑制过度自信——模型把"自己原本不校准的置信度"转化成了显式可训练的弃权行为。这是一种低成本的自我校准回路：不需要额外的不确定性估计器，只需要读出层概率本身。

## 实践启示

1. **判别类业务优先考虑决策模型格式，而非让 LLM 自由生成再解析。** 结构化输出省去文本解析与格式校验层，且概率天然支持阈值化：置信度够高自动执行，不够则升级人工。客服分派、内容审核、风控分级等高频判定场景是直接受益者。
2. **多问题共享同一 state 时，把前缀 cache 复用作为成本设计的第一原则。** Base Labs 把这一经济性约束直接编码进注意力掩码（block-bidirectional），精度反而更高。设计面向评测或批量判定的系统时，应先问"哪些 token 是公共前缀"，再决定架构。
3. **数据扩量出现收益递减后，转向对全部数据源的轮次化审计修复。** 报告中 Mix 7 对 253 个源重审标签错误、答案泄露与格式问题即可再涨分。可操作的做法是：每轮训练后按评测弱项定位数据源，重审而非重造，并区分"缺数据"与"数据坏了"两类问题。
4. **合成数据必须叠加多重独立校验，单一过滤器不可信。** 该管线用五裁判打标（至少 4/5 一致才保留）、拒绝歧义与多解、对评测集去污染、独立 agent 复推标签并排查选项泄露捷径、最后以 240 步小试训验证新数据源——五道检查各抓不同的坏例，缺一不可。
5. **显式弃权能力要从模型自身的置信信号反向构造，而不是指望 SFT 直接学会。** 用"最高概率 < τ"作为弱信号、以随机移除正确答案的方式制造弃权监督，是可迁移的配方：凡是需要"知道自己不知道"的分类系统，都可以照此构造训练目标。
6. **用小模型迭代数据配方，定型后再放大。** Base Labs 全部数据实验都在 9B 的 elephant-1 上完成，Mix 8 定型后才训练 27B 的 mammoth-1。数据配方的相对收益在模型规模间大体可迁移，先用小模型把迭代周期压短、成本压低，是预算有限团队的现实路径。

## 关联

- [[entities/liquid-ai-d1-edge-decision-models]] — 同族（decision model）另一团队路线，AA Decision Index 上的直接对手
- [[concepts/attention-mechanism]] — block-bidirectional mask 是注意力掩码层面的变体
- [[concepts/inference-optimization]] — "不生成 token 因此便宜"是决策模型格式的部署侧动因

→ [[raw/articles/mammoth-1-elephant-1-decision-models-base-labs-2026|原文存档]]
