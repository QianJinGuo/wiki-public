---
title: Transformer Architecture
created: 2026-04-28
updated: 2026-08-01
type: concept
tags: [transformer, architecture, deep-learning, llm, attention, rope, flashattention, kv-cache, moe]
related:
  - [[entities/agent框架owl原理详解|OWL]]
  - [[entities/claude-code-core-internals|Claude Code]]
  - [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw]]
sources: []
confidence: high
---
# Transformer Architecture
## Overview

Transformer 是一种基于**自注意力机制（Self-Attention）**的神经网络架构，于 2017 年由 Vaswani 等人在《Attention Is All You Need》中提出。凭借其并行化能力强、可扩展性高的优势，Transformer 已成为大语言模型（LLM）的基石，并从 NLP 领域扩展到多模态（视觉、语音、代码生成）等多个方向。

现代 AI Agent 框架（如 [[entities/agent框架owl原理详解|OWL]]、 [[entities/claude-code-core-internals|Claude Code]]、 [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw]]）的底层推理能力，本质上都是 Transformer 模型通过自注意力对上下文进行建模，再配合工具调用、记忆管理、子 Agent 调度等上层架构实现的。理解 Transformer 的内部机制，是理解现代 Agent 系统能力边界与性能优化空间的必要前提。

## Transformer Fundamentals

### Embedding：Token 到向量的映射

Transformer 的输入是离散的 token 序列（文本、代码、或任意离散符号），通过一个可学习的 Embedding 层映射为连续向量空间中的稠密向量。Embedding 矩阵的维度记为 $d_{model}$，通常为 768、1024、1280、2048 等。输入 token 经过 Embedding 后获得固定维度的向量表示，后续由注意力层和前馈网络在此向量空间中进行运算。

### Self-Attention：核心创新

Self-Attention（自注意力）是 Transformer 的核心创新，它允许序列中任意两个位置之间直接建立依赖关系，无需像 RNN 那样逐 token 顺序处理。

**数学形式**（Scaled Dot-Product Attention）：

给定输入序列的向量表示 $\mathbf{X} \in \mathbb{R}^{n \times d_{model}}$，通过三个线性投影得到查询（Query）、键（Key）、值（Value）：
$$\mathbf{Q} = \mathbf{X}\mathbf{W}_Q,\quad \mathbf{K} = \mathbf{X}\mathbf{W}_K,\quad \mathbf{V} = \mathbf{X}\mathbf{W}_V$$

注意力分数和输出为：
$$\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}\!\left(\frac{\mathbf{Q}\mathbf{K}^\top}{\sqrt{d_k}}\right)\mathbf{V}$$

其中 $\sqrt{d_k}$ 为缩放因子（Scaling），用于缓解高维向量点积后数值过大导致的 softmax 梯度消失问题。

**多头注意力（Multi-Head Attention, MHA）** 将上述过程在多个"头"上并行执行：
$$\text{MHA}(\mathbf{X}) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h)\mathbf{W}_O$$
$$\text{head}_i = \text{Attention}(\mathbf{X}\mathbf{W}_{Q_i}, \mathbf{X}\mathbf{W}_{K_i}, \mathbf{X}\mathbf{W}_{V_i})$$

每个头在不同的子空间中学习不同的关联模式（如语法关系、语义相似性、指代消解等），最终拼接后投影回 $d_{model}$ 维度。多头注意力的设计让模型能够同时捕获多种不同类型的依赖关系。

在 Agent 场景下，自注意力是实现**上下文理解**的基础：模型能够"看到" User Agent 和 Assistant Agent 的完整交互历史（参见 [[entities/agent框架owl原理详解|OWL 多 Agent 架构]]），理解工具调用的返回值对后续决策的影响，以及在多轮对话中保持一致的角色认知。

### Feed-Forward Network（FFN）

每个 Transformer 层除了注意力机制外，还包含一个逐位置（position-wise）的全连接前馈网络，结构为两层线性变换，中间插入非线性激活函数（通常为 ReLU 或 GELU）：

$$\text{FFN}(\mathbf{x}) = \mathbf{W}_2\,\sigma(\mathbf{W}_1\mathbf{x} + \mathbf{b}_1) + \mathbf{b}_2$$

FFN 通常扩展为输入维度的 4 倍（$d_{ff} = 4 \times d_{model}$），是 Transformer 中参数占比最大的组件（约 2/3 的参数量）。FFN 的作用是对注意力层的输出进行非线性语义变换，完成信息的选择性提炼。在推理阶段，FFN 的计算量通常与 KV-cache 的长度无关（仅依赖当前 token 的隐藏状态），这使得它在长上下文场景下成为计算瓶颈的重要来源。

### Positional Encoding：赋予序列顺序

Self-Attention 本身是**排列等价（permutation invariant）**的——对输入序列的任意排列，注意力输出不变。这意味着 Attention 本身不感知 token 的位置信息，必须通过额外机制注入。

**原始 Transformer（绝对位置编码）** 使用正弦和余弦函数：
$$PE_{(pos, 2i)} = \sin\!\left(\frac{pos}{10000^{2i/d_{model}}}\right),\quad PE_{(pos, 2i+1)} = \cos\!\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$

这种编码方式的优势是能够在推理时泛化到训练时未见过的序列长度（通过外推的正弦曲线）。

**旋转位置编码（RoPE）** 是 LLaMA 等主流模型采用的位置编码方案，详见下文"关键架构创新"章节。

## Architecture Evolution

### Encoder-Decoder → Decoder-Only → Prefix LM

| 架构类型 | 代表模型 | 特点 |
|---|---|---|
| **Encoder-Decoder** | 原版 Transformer、T5、BART | 完整编码器+解码器，双向注意力建模输入，单向注意力生成输出 |
| **Decoder-Only** | GPT 系列、LLaMA、Claude | 仅用解码器，自回归生成，训练效率高，是当前 LLM 的主流架构 |
| **Prefix LM** | GLM、PaLM-2 | Prefix 部分双向注意力，生成部分单向注意力，兼顾理解与生成 |

OWL 框架（[[entities/agent框架owl原理详解]]）和 Claude Code（[[entities/claude-code-core-internals]]）均运行在 Decoder-Only 的 LLM（GPT-4o / Claude 系列）之上，通过自回归方式逐 token 生成工具调用指令和对话回复。

## Key Architectural Innovations

### RoPE：旋转位置编码

RoPE（Rotary Position Embedding，苏剑林等 2021 年提出）通过**在 Q 和 K 向量空间中旋转**来实现位置编码，是当前最广泛采用的位置编码方案，被 LLaMA、GLM、Qwen 等主流模型系列采用。

其核心思想：将位置信息编码为旋转矩阵，作用于 Q 和 K 的前 $d$ 维子空间，使得内积 $\mathbf{q}_m^\top \mathbf{k}_n$ 仅依赖于相对位置 $m-n$。

**数学形式**：对于维度 $d$（设为偶数），将向量分为 $d/2$ 个二维子空间，每个子空间旋转角度 $\theta_i = b^{-2i/d}$（$b$ 通常为 10000）：
$$\mathbf{q}_m' = \mathbf{R}(m, \theta)\mathbf{q}_m,\quad \mathbf{k}_n' = \mathbf{R}(n, \theta)\mathbf{k}_n$$
$$\mathbf{R}(m, \theta) = \begin{pmatrix} \cos(m\theta) & -\sin(m\theta) \\ \sin(m\theta) & \cos(m\theta) \end{pmatrix}$$

RoPE 的核心优势：
1. **自然支持相对位置**：注意力内积只依赖 $(m-n)$，无需额外的相对位置偏置
2. **可插拔**：作为位置编码注入，不改变 Attention 的核心结构
3. **长上下文友好**：通过适当的频率基数 $b$ 调优，可以获得较好的上下文长度外推能力

RoPE 的引入使得 [[entities/claude-code-core-internals|Claude Code]] 这类基于长上下文的 Agent 系统能够在有限的 context window 内更有效地利用相对位置信息，进行跨长距离的依赖建模。

### FlashAttention：注意力计算的高效实现

标准 Attention 的计算复杂度为 $O(n^2)$（$n$ 为序列长度），且需要将完整的 $n \times n$ 注意力矩阵存储在 HBM（高带宽内存）中。FlashAttention（Dao et al., 2022/2023）通过**分块计算（tiling）+ 算子融合（kernel fusion）** 将注意力计算重新组织，在不改变模型输出数值的前提下，大幅降低内存占用并提升计算速度。

**核心思想**：
- 将 $N \times N$ 注意力矩阵切分为 $T \times T$ 的小块（block），每次只将一个 block 加载到 SRAM
- 利用在线 softmax 技巧，在分块计算过程中累积 softmax 的分子（最大值）和分母（指数和），最终得到正确结果
- 无需实例化完整注意力矩阵，内存占用从 $O(n^2)$ 降低到 $O(n)$

FlashAttention 的实际效果：
- 端到端加速约 2-4 倍（具体取决于硬件和序列长度）
- 内存占用降低至原来的 $1/10 \sim 1/20$
- 使得单卡可处理的序列长度大幅提升

在 Claude Code（[[entities/claude-code-core-internals]]）的五层 Context 压缩机制中，microCompact 利用了 Anthropic API 的 `cache_edits` 参数在服务端屏蔽旧 token——这一优化的物理基础正是 FlashAttention 带来的内存效率提升，使得在有限的 HBM 空间内维持更长的 KV 序列成为可能。

### KV-Cache：推理阶段的核心优化

自回归生成的标准实现需要对每个新 token 重新计算与所有历史 token 的注意力，复杂度为 $O(n^2)$。KV-Cache 通过**缓存已计算的 Key-Value 状态**，将计算复杂度降低到 $O(1)$（每步只计算当前 token 的 QKV 并追加到缓存）。

**工作原理**：
1. 首次预填充（Prefill）阶段：对输入 prompt 完整计算一次注意力，缓存所有 K、V
2. 每步解码（Decode）阶段：仅计算新 token 的 Q（查询），从缓存中读取历史 K、V 参与注意力计算

KV-Cache 的代价是内存占用随序列长度线性增长。对于 $n$ 个 token、$l$ 层、$h$ 个注意力头、$d$ 个头维度的模型，KV-cache 大小约为 $2 \times n \times l \times h \times d \times 2\text{bytes}$（FP16）。一个 70B 模型在 4096 tokens 上下文时，KV-cache 可达数十 GB。

**PagedAttention**（vLLM 提出）是 KV-cache 的生产级优化方案，借鉴操作系统虚拟内存的分页思想，将 KV-cache 划分为固定大小的块（block），通过块级管理和共享实现内存的高效利用：

- 多个序列可以共享相同的 KV block（前缀复用）
- 按需分配和释放，避免预分配导致的内存碎片
- 支持 continuation、beam search、swap 等复杂解码策略

在 OpenClaw 架构（[[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw]]）中，AgentLoop 的历史消息（history）管理本质上是应用层的"Context Cache"——通过限制 history 长度（主 Agent 10 轮、子 Agent 15 轮迭代）来控制不断增长的 token 序列长度，其背后的工程逻辑与 KV-cache 的内存压力控制同源。

### MoE：混合专家架构

MoE（Mixture of Experts）通过**稀疏激活**机制突破模型参数量与计算量的线性绑定，在不增加推理计算成本的前提下大幅扩展模型容量。

**核心架构**：
- 将 FFN 替换为 $N$ 个独立的"专家" FFN（$N$ 通常为 8、16、32、64 等）
- 引入一个轻量的**路由机制（Router）**，对每个 token 预测性地选择 Top-K 个专家（通常 K=1 或 K=2）
- 最终输出为被选中专家输出的加权和

数学上：$y = \sum_{i \in S} G_i \cdot \text{FFN}_i(x)$，其中 $S$ 为路由选择的 Top-K 专家集合，$G_i$ 为路由分配的稀疏门控权重。

**代表模型**：Mixtral 8x7B（8 个专家，Top-2 激活）、DBRX、Grok-1、Qwen1.5-MoE 等。

MoE 对 Agent 系统的意义在于：稀疏激活使得可以在固定推理预算下运行更大容量的模型。对于需要同时处理复杂推理（[[entities/claude-code-core-internals|Claude Code]] 的多轮规划）和工具调用（[[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw]] 的 SpawnTool/CronTool）的 Agent 场景，更大的模型容量意味着更强的指令理解、多步规划和工具选择能力，而稀疏激活使得这一切不需要按比例增加推理算力。

## Norm Variants：PreNorm vs PostNorm vs RMSNorm

| 归一化方案 | 结构特点 | 代表模型 | 备注 |
|---|---|---|---|
| **PostNorm** | 残差连接在归一化之后：$x_{out} = \text{LayerNorm}(x + \text{Sublayer}(x))$ | 原版 Transformer | 训练不稳定，但最终模型质量高 |
| **PreNorm**（也称 Pre-LN） | 残差连接在归一化之前：$x_{out} = x + \text{Sublayer}(\text{LayerNorm}(x))$ | GPT、LLaMA | 训练更稳定，是当前主流 |
| **RMSNorm** | 仅用均方根归一化，去掉均值 centering | DeepSeek、Qwen2 | 计算量更小，效果与 PreNorm 持平 |
| **DeepNorm** | 在 PostNorm 基础上引入缩放因子 | DeepSeek | 专为极深网络设计 |

[[entities/agent框架owl原理详解|OWL 框架]] 在底层使用 GPT-4o，属于 PreNorm 架构。DeepSeek 系列的 DeepNorm 是对 PostNorm 范式的重要改进，使得极深网络（DeepSeek-V2 的 MoE 层数超过 130 层）的稳定训练成为可能。

## How Transformers Underpin Modern Agent Architectures

Transformer 是现代 AI Agent 系统的**底层推理引擎**，上层 Agent 框架通过以下方式与 Transformer 交互：

### 1. 自注意力实现上下文感知

自注意力允许模型在每个 token 的表示中融合任意历史位置的信息，这是 Agent 实现多轮对话、记忆召回、工具调用链追踪的基础。

以 [[entities/agent框架owl原理详解|OWL]] 的 User Agent ↔ Assistant Agent 对话为例：每一轮交互的历史消息（含工具调用结果）都被完整地拼接为新的输入序列，Transformer 的自注意力在此序列上建立跨轮次的依赖关系。Assistant Agent 在第 $N$ 轮的决策能够直接"关注""第 $N-1$ 轮的 Assistant 工具调用失败"这一事实——这是自注意力机制赋予的能力，而非 Agent 框架额外编写的逻辑。

### 2. 工具调用（Function Calling）本质上是 Attention 触发的行为

现代 Agent 框架的工具调用并非"规则匹配"——模型的工具选择决策是由 Transformer 的前向传播（FFN 层）通过隐式语义推理做出的。

以 [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw]] 为例：LLM 在生成工具调用时，注意力机制会根据当前任务描述（task）、对话历史（history）和工具描述（name、description、input_schema）三者的语义关联程度来决定选择哪个工具。`ToolRegistry.exclude()` 机制则通过将特定工具的 schema 从输入中移除，让模型在注意力层面"看不到"这些工具，实现能力隔离——这是一种在**注意力输入空间**而非代码逻辑层面的约束。

### 3. 子 Agent 管理依赖 Transformer 的并行推理能力

[[entities/claude-code-core-internals|Claude Code]] 的 7 种子 Agent 模式和 [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw]] 的 Promise 并发子 Agent 模型，都依赖于 Transformer 模型在子 Agent 粒度上的**独立前向传播**。子 Agent 获得独立的 context（Prompt/ReAct 历史），模型在其内部完成推理后通过 MessageBus（OpenClaw）或 `agent.run()` 主循环（Claude Code）回传结果。这一架构能够成立，根本原因是 Transformer 的并行计算特性使得多个子 Agent 的推理可以在共享硬件资源上高效交错执行。

### 4. System Prompt 动态组装与注意力分配

Claude Code 的动态 System Prompt（`buildEffectiveSystemPrompt`）是一个极具 Transformer 特色的设计：6 层优先级（基础行为契约 → 工具描述 → MCP 指令 → Skill 索引 → 环境信息 → ToolSearch 提示）的运行时组装，本质上是在控制 Transformer 注意力分配的信息来源。当工具集因 MCP 连接状态变化时，工具描述的动态注入直接改变了注意力计算的信息源，让模型能够感知"当前有哪些可用工具"，进而做出正确的工具选择决策。

## Relation to Inference Optimization

Transformer 的推理优化与 Agent 系统性能直接相关：

### KV-Cache 与 Context Window 管理

在 Claude Code 的五层 Context 压缩机制中，microCompact 利用 `cache_edits` 参数在**服务端**对旧工具结果做注意力屏蔽（attention mask = 0），而非删除本地消息——这一设计精准地利用了 KV-cache 的物理特性：如果物理 token 序列不变，KV 矩阵的物理存储位置也不变，prompt cache 的 KV 引用就不会失效。这使得 microCompact 能在**不破坏缓存命中**的前提下减少有效注意力长度。

### FlashAttention 与长上下文 Agent 场景

FlashAttention 带来的内存效率提升，使得单次前向传播可以处理更长的序列，直接支持了 Claude Code 对 git status 上下文、仓库文件内容、工具返回结果等大量信息的并发注入。当 FlashAttention 将可处理的序列长度从 4K 提升到 32K-128K 时，Agent 在单次 forward 中能够"看到"的上下文量大幅增加，多步规划、跨文件代码理解、长篇文档分析等场景才成为可能。

### 推理服务层的优化对 Agent 吞吐的影响

生产级 Agent 系统（如 Claude Code、OpenClaw 接入的生产 Bot）的并发能力，不仅取决于 Agent 框架的调度设计，还取决于底层 Transformer 推理服务的吞吐：

- **Continuous Batching / Iteration-Level Scheduling**：允许多个请求在 token 级别交织执行，提高 GPU 利用率
- **Prefix Caching**：多个请求共享 prompt 前缀（常见于 System Prompt 固定的 Agent 场景），KV-cache 可复用
- **Speculative Decoding**：用小模型预测后续 token，大模型验证，在保持输出质量的同时提升解码速度

这些优化对 Agent 开发者和框架设计者通常是透明的，但对 Agent 系统的实际响应延迟和并发吞吐有决定性影响。

## Related

- [[concepts/attention-mechanism]] — 注意力机制的多种变体（Softmax Attention、Linear Attention、FlashAttention 等）
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes]] — 残差连接的工程优化方案，注意力选择前驱层机制
- [[entities/agent框架owl原理详解|Agent框架OWL原理详解]] — 多 Agent 协作框架，Transformer 作为底层推理引擎
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 动态 System Prompt、子 Agent 模式、Context 压缩的完整实现
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw 架构]] — 薄抽象 Tool 系统、MessageBus、SubagentManager 的最小实现
- [[entities/design-patterns-for-ai-agents-2026|Design Patterns for AI Agents 2026]] — 2026 年 AI Agent 设计模式全景
- [[entities/17-agent-architectures-evolution|17种Agent架构演进]] — 控制流设计的完整演化史
- [[entities/agent-harness-architecture|Agent Harness 架构]] — Harness 与 Transformer 推理引擎的交互模式

## 关联实体

**上游依赖**:
- [[entities/agent框架owl原理详解]] — 提供基础理论/方法
- [[entities/claude-code-core-internals]] — 提供基础理论/方法
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构]] — 提供基础理论/方法

**下游应用**:
- [[entities/claude-code-core-internals]] — 具体应用场景
- [[entities/claude-code-core-internals]] — 具体应用场景
- [[entities/claude-code-core-internals]] — 具体应用场景

**平行协作**:
- [[entities/claude-code-core-internals]] — 替代/补充方案
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构]] — 替代/补充方案
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-1-llm-principles|Layer 1 Llm Principles]]
- [[moc/wiki-master-map|Wiki Master Map]]
