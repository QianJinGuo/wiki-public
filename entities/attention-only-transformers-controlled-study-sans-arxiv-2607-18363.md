---
title: "Attention-Only Transformers 受控研究（SANs vs 标准 Transformer）"
created: 2026-08-11
updated: 2026-09-11
type: entity
tags: [transformer, attention, architecture, arxiv, ml-research, feed-forward]
sources: [raw/articles/controlled-study-attention-only-transformers-arxiv-2607-18363]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Attention-Only Transformers 受控研究（SANs vs 标准 Transformer）

## 摘要

这篇 arXiv 论文（2607.18363，Henry Ndubuaku 等，2026-07-20 提交）做了 transformer 架构史上一次罕见的「必要性检验」：前馈层占了非 embedding 参数的三分之二，却从没有人同时控制参数量、算力和深度来问一句「它到底是不是必需的」。作者用预训练的纯注意力 decoder（Simple Attention Networks, SANs）与标准 transformer 逐项配对对照，发现原位删除 FF 层的代价看似很大，但只要把释放出的预算重分配进「注意力深度」，差距就几乎消失——由此把架构优劣之争还原为「训练配方是否公平」。

## 核心要点

- **检验对象**：前馈（feed-forward）层占 transformer 非 embedding 参数的 2/3，但此前从未有过同时控制参数、算力、深度的必要性检验。
- **实验设置**：attention-only decoder transformers（SANs）对标准 transformer 做三组独立配对——匹配参数量、匹配训练 FLOPs、匹配深度（2 到 48 层），训练最高 105B tokens，模型规模 6M 到 87M 参数。
- **原位删除代价高**：matched-depth 下标准 transformer 领先 0.47 nats，matched-FLOPs 下领先 0.26 nats——直接砍掉 FF 层确实吃亏。
- **预算重分配即闭合**：把省下的参数/算力投入注意力深度后，matched-params 下差距降到 0.006 nats（仅占损失的 0.27%），跨 seed 对可复现到万分之一（one part in ten thousand）。
- **规模越大差距越小**：在 5B / 30B / 105B token 预算上差距持续收缩，且在 29 倍模型尺寸范围内稳定在 ~0.02 nats 附近。
- **剩余差距定位到参数化回忆（parametric recall）**：attention-only 模型在上下文接地（context-grounded）回答上更好，在必须从权重里取知识的场景更差。
- **权重谱解释**：路由矩阵（Q/K）早期结晶，内容矩阵秩积累缓慢；删掉 FF 层会把这种秩积累「搬家」到注意力输出投影上。
- **可训练性关键**：撑住 48 层 attention-only 栈的是 QK-normalization，而不是 FF 层或残差门控。
- **预注册验证**：作者事先预测知识密集型 web 文本上存在 0.02–0.05 nat 差距，fineweb-edu 匹配对实测 0.040，落在预测区间内。

## 深度分析

### 一、为什么这场「必要性检验」此前一直缺席

FF 层占了非 embedding 参数的三分之二，这个数字足以让人默认它不可替代；但「占比大」只说明它被大量使用，不说明它不可替代。真正的难点在实验设计：参数、算力、深度三者必须同时公平匹配，任何一项失控都会让比较变成「苹果比橘子」。多数消融只能控制其中一到两项，于是「删掉 FF 就崩」里混杂了大量训练配方差异。本文的价值首先不在结论，而在于它同时拉起了这三条控制线。^[raw/articles/controlled-study-attention-only-transformers-arxiv-2607-18363.md]

### 二、差距的真正来源是训练配方混淆，不是注意力的能力上限

「原位删除代价高」与「重分配预算后差距消失」这两个结果必须成对阅读。前者给出 0.47 / 0.26 nats 的落差，看起来像是 FF 层在承担不可替代的计算；但后者证明，只要把释放出的参数量与算力重新投给注意力深度，matched-params 差距就掉到 0.006 nats，而且跨 seed 稳定到万分之一。也就是说，标准 transformer 的领先很大程度上来自「更深/更宽的注意力栈」这一间接红利，而不是 FF 层本身提供了注意力做不到的运算。差距还随训练预算扩大而收缩、在 29 倍尺寸范围内稳定在 ~0.02 nats，更像残余效应而非结构性缺陷。对架构之争的含义很直接：拿不同层数、不同深度配比的模型互比「谁更强」，本质上是在比训练配方，而非比架构的表达力上限。^[raw/articles/controlled-study-attention-only-transformers-arxiv-2607-18363.md]

### 三、残余差距的定位：参数化回忆与权重谱动力学

作者用三组测量把剩余差距钉到具体机制上：参数化回忆。attention-only 模型在需要从上下文取答案时反而更好，在需要把知识固化进权重、再靠权重作答时更差；差距集中在低上下文（low-context）查询预测上，并在最大训练预算下完全局限于此。权重谱分析给出动力学解释——路由类矩阵（Q/K）很早就「结晶」定型，内容类矩阵的秩需要慢慢积累，而 FF 层正是原本承担这部分缓慢积累的地方；删除 FF 层后，这种秩积累被重新安置到注意力输出投影上。这解释了为什么差距「可弥合但不为零」：结构上可以迁移，但迁移效率和规模有关。预注册测试进一步把解释变成可证伪的预测，并在 fineweb-edu 上得到 0.040 的实测值，落在事先声明的 0.02–0.05 nat 区间里。^[raw/articles/controlled-study-attention-only-transformers-arxiv-2607-18363.md]

### 四、局限与适用边界

必须强调作者自己的边界措辞：「在受测范围内，注意力做了其余一切（within the tested regime, attention does the rest）」。受测范围是 6M–87M 参数的 decoder-only、最高 105B tokens 的小规模预训练，这是典型的受控实验尺度而非前沿 LLM 尺度；把小规模下「差距可闭合」外推到大模型，尤其是知识密集、长上下文、需要强参数化回忆的任务，需要额外证据。其次，可训练性前提不能忽略：48 层 attention-only 栈能训起来靠的是 QK-normalization，说明结论对归一化选择敏感；而「更好的上下文接地」与「更差的权重知识」是一体两面，SANs 更适合检索/上下文驱动型负载，而非纯知识记忆任务。最后，本次检验只针对 FF 层必要性，不构成对 SSM/卷积/混合架构的判决。^[raw/articles/controlled-study-attention-only-transformers-arxiv-2607-18363.md]

## 实践启示

1. **做架构消融时先把控制线拉满**：参数、算力、深度三者必须同时匹配，否则「删掉 X 就变差」很可能只是训练配方不公平的产物。本文的实验设计可直接当作消融模板。
2. **把「组件必要性」和「预算分配」分开问**：一个组件看起来不可替代，可能只是因为它顺带带来了更多深度/宽度；先做重分配实验，再下必要性结论。
3. **用预注册预测验证机制解释**：本文把「残余差距源于参数化回忆」变成可证伪的数值预测（0.02–0.05 nat），再用 fineweb-edu 实测 0.040 对照，这种「先声明再验证」的做法值得在内部研究里复制。
4. **按工作负载类型选架构**：若任务以上下文接地（RAG、长文档问答、检索增强）为主，纯注意力结构在受控对比下并无劣势；若任务高度依赖权重内知识记忆，应预期 attention-only 存在 ~0.02–0.05 nat 量级的持续劣势。
5. **归一化是 attention-only 栈的隐藏前提**：部署或复现深层 attention-only 模型时，QK-normalization 不可省，不能用 FF 层或残差门控替代。
6. **审慎对待 scaling-law 比较**：不同架构间的 loss 对比，如果不报告参数量/算力/深度匹配方式，几乎无法解读；引用架构优劣结论前先确认这三项是否被控制。

## 相关实体

- [[concepts/attention-mechanism|Attention Mechanism]]
- [[concepts/scaling-laws|Scaling Laws]]
- [[concepts/transformer-architecture|Transformer Architecture]]
- [[entities/discoformer-density-score-transformer-allen-ai|DiscoFormer]]
- [[entities/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu|SSM Attention 睡眠巩固研究]]
- [[entities/attention-collapse-context-management|Attention Collapse 与上下文管理]]

→ [[raw/articles/controlled-study-attention-only-transformers-arxiv-2607-18363|原文存档]]
