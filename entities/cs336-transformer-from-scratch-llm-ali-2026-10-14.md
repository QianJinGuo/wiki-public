---
title: "CS336 从零手搓 LLM（Transformer 篇）：面向 Agent 工程师的底层实现路径"
created: 2026-10-14
updated: 2026-10-15
type: entity
tags: [cs336, llm-from-scratch, transformer, rope, swiglu, rmsnorm, bpe, pytorch, stanford, agent-engineer-education]
value: 7
confidence: 0.85
sources: [raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14]
provenance_state: extracted
---

# CS336 从零手搓 LLM（Transformer 篇）

阿里技术（★★★★★ 第一方）工程师学习系列首篇：跟随 Stanford CS336《Language Modeling from Scratch》Assignment1（stanford-cs336/assignment1-basics）从零手搓 Transformer 的完整实现教程（57,204 字全文代码）。CS336 的「from Scratch」= 从零手搓而非从零教学——禁止使用 PyTorch 现有模块（仅豁免 Parameter/Module/Optimizer 基类），作业采用 RoPE/SwiGLU 等现代变体而非 Attention Is All You Need 原始版；作业附带 AGENTS.md 防 coding agent 一键代写。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md]

## 面向 Agent 工程师的动机

不把 LLM 当魔法黑盒才能回答：什么 prompt 更好、model card 参数什么意思、technical report 架构变动代表什么——例如 Jev 与传统小模型的架构本质区别、为什么快；AI 冲击传统软件架构后，Agent 工程同学同时要为运行效果负责，必须了解模型原理/推理/训练。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md]

## 教学路径与实现要点

- **MLP 体感先行**：线性回归 → loss（MSE）→ 梯度下降/学习率直觉 → **多层 Linear 等价单层**（仿射变换复合仍是仿射）→ 必须加非线性激活；完整训练循环（mini-batch / zero_grad→backward→step / no_grad 验证防误入计算图）。全程 Java 工程师类比（unsqueeze 如 double[n][1]、nn.Module 如框架回调约定）。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md]
- **Transformer 组件（CS336 变体版）**：Embedding / causal Attention / **RoPE 旋转位置编码** / FFN 用 **SwiGLU** / **RMSNorm**（对比 LayerNorm）/ scaled softmax / 残差 / Multihead 分头 / **BPE tokenizer** 词表训练与编解码。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md]
- **定位**：系列第一篇，面向无系统 ML 背景的工程同学即用即学；CS336 作业驱动禁止调库，是"了解 LLM 底层原理"最系统的公开路径之一。后续篇目（optimizer/训练/数据等 Assignment1 其余部分）可按同档追加。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md]

## 深度分析

这篇教程的价值不在「又一篇 Transformer 讲解」，而在三点交叉：Stanford 正式课程作业的系统性、从零手搓对理解深度的强制检验、以及一位纯工程背景作者转做 Agent 开发的第一视角转译。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:11,15]

### 从零手搓是理解深度的可验证标准

CS336 的「from Scratch」= 从零手搓而非从零教学：禁止使用 PyTorch 现有模块，仅豁免 Parameter/Module/Optimizer 基类。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:15] 这把「懂了没有」从主观感觉变成可判定状态——能不用现成组件把前向传播和训练循环写出来，才算真懂。对自学者，这比再读十篇解析文章更能暴露理解空洞；对团队，这也是一种可复制的 LLM 原理培训验收方式。

### 组件选型刻意避开 2017 原始版

作业采用 RoPE 旋转位置编码、SwiGLU FFN、RMSNorm（对比 LayerNorm）等现代变体，而非 Attention Is All You Need 原始版。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:15] 全文代码出现频次（Embedding 36 处、SwiGLU 20 处、RMSNorm 17 处、RoPE 15 处、BPE tokenizer 11 处）说明教程重心放在当今开源模型实际使用的组件组合上，学完即可对齐主流 model card 里的架构描述。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:19-21]

### 教学顺序：先建立梯度体感，再堆 Transformer 组件

路径设计为线性回归 → MSE loss → 梯度下降/学习率直觉 → 多层 Linear 等价单层（仿射复合仍是仿射）→ 必须加非线性激活 → 完整训练循环（mini-batch 抽样、zero_grad→backward→step、no_grad 验证防误入计算图）。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:19] 即先在 MLP 尺度把「训练是什么」体感化，再进入 [[concepts/attention-mechanism|causal Attention]]、scaled softmax、残差、Multihead 分头等组件堆叠，^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:20] 避免一上来就面对完整 [[concepts/transformer-architecture|架构]] 的黑盒感。

### AGENTS.md：作业设计对 coding agent 时代的回应

作业附带 AGENTS.md 防 coding agent 一键代写。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:15] 这是一个值得注意的信号：连课程方都默认学习者手边有能完成作业的 agent，「亲自手搓」正在从默认状态变成需要制度保护的稀缺行为。对教育者，作业设计需要显式考虑 agent 可达性；对学习者，用 agent 生成代码再逐行验证是可接受的替代路径，但验证深度决定学习深度。

## 实践启示

1. 学完本篇能直接回答三类日常问题：什么 prompt 更好、model card 参数什么意思、technical report 架构变动代表什么——这是「懂底层」的即时回报，不必等到成为训练专家。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:15]
2. 自评标准用「能否不调库写出前向传播」：跑通 CS336 作业（或等价地从零实现 RoPE/SwiGLU/RMSNorm）比再读三篇解析更有效。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:15]
3. 团队内可用「从零手搓 + AGENTS.md 式约束」做 LLM 原理培训验收：禁止调库的作业天然防 agent 代写，保证被验收者本人真正理解。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:15]
4. Agent 工程同学要为运行效果负责，模型原理/推理/训练是职责范围内知识而非选修——Jev 与传统小模型的架构本质区别这类问题（详见 [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026|Jev 判断层]]），只有懂组件才能答。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:15]
5. 阅读顺序沿教程路径：先 MLP 训练循环体感，再 Transformer 组件，最后 [[concepts/llm-tokenizer|BPE tokenizer]] 词表训练与编解码；跳读组件部分会失去梯度层面的直觉地基。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:19-21]
6. 本篇是学习系列首篇（2026 年第 68 篇），后续 Assignment1 其余部分（optimizer/训练/数据）出现时可按同档追加，避免系列覆盖断裂。^[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14.md:11,25]

## 相关实体

- [[entities/cs336-transformer-from-scratch-llm-ali-2026-10-14]]（本页）
- [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026|Jev 判断层]]（文中作为"为什么要懂底层"的例子）

→ [[raw/articles/alibaba-cs336-transformer-from-scratch-2026-10-14|原文存档]]
