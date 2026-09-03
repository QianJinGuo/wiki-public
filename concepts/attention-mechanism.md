---
title: Attention Mechanism
created: 2026-04-28
updated: 2026-08-01
type: concept
tags: [transformer, attention, deep-learning, mha, mqa, gqa, flashattention, lsh, linear-attention]
related:
  - [[entities/claude-code-core-internals|Claude Code]]
  - [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes]]
  - [[entities/agent框架owl原理详解|OWL]]
sources: [raw/articles/kimi-attention-residuals-preNorm-dilution-block-attnres]
confidence: high
---
# Attention Mechanism（注意力机制）

## Overview

注意力机制允许模型在处理序列数据时动态关注输入的不同部分，是 Transformer 架构的核心组件。2017 年《Attention Is All You Need》提出标准 Scaled Dot-Product Attention，此后衍生出 Multi-Head Attention（MHA）、Multi-Query Attention（MQA）、Grouped Query Attention（GQA）、Multi-Resolution Attention（MRO）、Locality-Sensitive Hashing Attention（LSH）等众多变体，以及 FlashAttention、PagedAttention 等高效实现。理解注意力机制的数学本质与工程权衡，是理解现代 LLM 乃至 AI Agent 系统能力边界的必要前提。

---

## 一、Attention 数学原理（Q/K/V）

### 1.1 Scaled Dot-Product Attention

标准注意力（亦称 **Softmax Attention**）的数学形式：

给定输入序列向量表示 $\mathbf{X} \in \mathbb{R}^{n \times d_{model}}$，通过三个线性投影得到查询（Query）、键（Key）、值（Value）：
$$\mathbf{Q} = \mathbf{X}\mathbf{W}_Q,\quad \mathbf{K} = \mathbf{X}\mathbf{W}_K,\quad \mathbf{V} = \mathbf{X}\mathbf{W}_V$$

其中 $\mathbf{W}_Q, \mathbf{W}_K, \mathbf{W}_V \in \mathbb{R}^{d_{model} \times d_k}$ 为可学习参数矩阵（通常 $d_k = d_{model} / h$，$h$ 为注意力头数）。

**注意力分数计算**：
$$\mathbf{S} = \frac{\mathbf{Q}\mathbf{K}^\top}{\sqrt{d_k}} \in \mathbb{R}^{n \times n}$$

**Softmax 归一化**：
$$\mathbf{A} = \text{softmax}(\mathbf{S}) \in \mathbb{R}^{n \times n}, \quad A_{ij} = \frac{\exp(S_{ij})}{\sum_{k=1}^{n}\exp(S_{ik})}$$

**加权求和输出**：
$$\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \mathbf{A}\mathbf{V} \in \mathbb{R}^{n \times d_k}$$

### 1.2 缩放因子 $\sqrt{d_k}$ 的物理意义

点积 $\mathbf{q}^\top \mathbf{k}$ 的方差与向量维度 $d_k$ 成正比。当 $d_k$ 较大时，点积值数量级较大，softmax 的输入分布在极值区域，梯度接近于零（梯度消失）。除以 $\sqrt{d_k}$ 使得点积的方差退回 $O(1)$，保证 softmax 在合理梯度区间工作。

### 1.3 Multi-Head Attention（MHA）

单头注意力的表达能力有限，**多头注意力（MHA）** 在多个并行"头"上分别执行注意力，每个头有独立的 $\mathbf{W}_Q, \mathbf{W}_K, \mathbf{W}_V$：

$$\text{MHA}(\mathbf{X}) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h)\mathbf{W}_O$$
$$\text{head}_i = \text{Attention}(\mathbf{X}\mathbf{W}_{Q_i}, \mathbf{X}\mathbf{W}_{K_i}, \mathbf{X}\mathbf{W}_{V_i})$$

其中 $\mathbf{W}_O \in \mathbb{R}^{hd_k \times d_{model}}$ 将拼接后的输出投影回 $d_{model}$。

每个头在不同的子空间中学习不同的关联模式（如语法关系、语义相似性、指代消解等）。MHA 是当前大多数 Transformer 模型的标准配置，如 GPT 系列、LLaMA、Claude 等均采用 MHA。

---

## 二、Attention 变体详解

### 2.1 Multi-Query Attention（MQA）

**MQA**（Fast Transformer Decoding: One Write-Head is All You Need，Shazeer 2019）让所有注意力头共享同一套 $\mathbf{K}$ 和 $\mathbf{V}$，仅保留独立的 $\mathbf{Q}$：

$$\mathbf{K}_{shared} = \mathbf{X}\mathbf{W}_K^{shared}, \quad \mathbf{V}_{shared} = \mathbf{X}\mathbf{W}_V^{shared}$$
$$\text{head}_i = \text{Attention}(\mathbf{X}\mathbf{W}_{Q_i}, \mathbf{K}_{shared}, \mathbf{V}_{shared})$$

**优势**：KV 缓存体积大幅减少（从 $h \times n \times d_k$ 降至 $1 \times n \times d_k$），显著降低显存占用，提升推理吞吐量。

**代价**：所有头关注相同的内容，多样性降低，效果通常略低于 MHA。

代表模型：PaLM、Google's T5 早期版本。

### 2.2 Grouped Query Attention（GQA）

**GQA**（GQA: Training Generalized Multi-Query Transformer，2023）是 MHA 与 MQA 的折中：将注意力头分为 $g$ 个组，同组内共享 KV，不同组独立：

$$g = \frac{h}{\text{num_key_value_heads}}$$

当 $g = 1$ 时退化为 MQA；当 $g = h$ 时等价于 MHA。

**优势**：在 KV-cache 效率和模型质量之间取得良好平衡，是当前主流开源模型的首选方案。

代表模型：**LLaMA 2/3**、LLaMA 3.1、Qwen 2、DeepSeek-V2 均采用 GQA（通常 $g = 4 \sim 8$）。

### 2.3 Multi-Resolution Attention（MRO）

**MRO** 是对标准注意力在**分辨率维度**的扩展，最初源于 Vision Transformer（ViT）处理高分辨率图像的需求，以及 LLM 处理超长上下文时的效率优化。

**核心思想**：在不同分辨率（不同粒度）的子序列上分别执行注意力，最后融合。

**典型实现方式**：
- **层次化注意力**（Hierarchical Attention）：如 Swin Transformer，在局部窗口内做注意力（局部注意），跨窗口信息通过上层token传递
- **金字塔注意力**（Pyramid Attention）：对 KV 进行下采样（如 stride > 1），在压缩后的低分辨率序列上做全连接注意力，再将结果插值回原分辨率
- **Longformer/BigBird 模式**：对短序列用全注意力，对长序列用滑动窗口 + 全局 token 的稀疏注意力组合

**数学形式**（层次化变体）：
$$\mathbf{Y}_{low} = \text{Attention}(\mathbf{Q}_{low}, \mathbf{K}_{low}, \mathbf{V}_{low})$$
$$\mathbf{Y} = \text{Interpolate}(\mathbf{Y}_{low}) \quad \text{或} \quad \mathbf{Y} = \text{Concat}(\mathbf{Y}_{low}, \mathbf{Y}_{high})$$

MRO 对 Agent 的意义：处理长篇文档、多文件代码库时，模型需要在局部精确性和全局上下文感知之间切换——MRO 提供了一种在单一前向传播中平衡这两种需求的机制。

### 2.4 Locality-Sensitive Hashing Attention（LSH）

**LSH Attention**（原版：Self-Endoder Attends to the Most Informative Tokens，2020；更著名的是 **Reformer** 2020）使用局部敏感哈希（LSH）将 token 聚类到哈希桶，在同桶内执行注意力，大幅降低计算复杂度。

**LSH 函数**：对位置 $i$ 的 query $\mathbf{q}_i$，计算其哈希值：
$$h_i = \text{sign}(\mathbf{q}_i \mathbf{R})$$

其中 $\mathbf{R} \in \mathbb{R}^{d_k \times b}$ 为随机投影矩阵，$b$ 为哈希桶数。相似的 $\mathbf{q}$（高内积）会以高概率落入相同桶。

**LSH Attention 操作**：
1. 对所有 query 计算哈希桶分配
2. 仅在同桶或邻近桶的 token 之间执行注意力
3. 复杂度从 $O(n^2)$ 降至约 $O(n \log n)$ 或 $O(n^{1.5})$

**代表模型**：Reformer、KyBot、CRAM（Cross-Recipient Attention Mechanism）。

**局限**：哈希碰撞导致近似误差，对于强语义依赖的长距离关系可能遗漏。

### 2.5 Linear Attention

**Linear Attention**（Linear Transformers are Secretly Fast Weight Processors，2020）将注意力矩阵的 softmax 分解为线性形式，将 $O(n^2)$ 复杂度降至 $O(n)$：

$$\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V})_i = \frac{\sum_j \exp(\mathbf{q}_i^\top \mathbf{k}_j) \mathbf{v}_j}{\sum_j \exp(\mathbf{q}_i^\top \mathbf{k}_j)} \approx \frac{\phi(\mathbf{q}_i)^\top (\exp(\mathbf{k}_j^\top) \mathbf{v}_j^\top)}{\phi(\mathbf{q}_i)^\top \exp(\mathbf{k}_j^\top)}$$

引入特征映射 $\phi(\cdot)$（如随机傅里叶特征）来近似核函数。Linear Attention 无法精确还原标准注意力，但在长序列场景下具有明显效率优势。

**代表模型**：Linear Transformer、Performer、FLOP。

---

## 三、FlashAttention 算法详解

### 3.1 问题背景

标准 Attention 面临两个低效问题：

1. **内存瓶颈**：需要将完整的 $n \times n$ 注意力矩阵存储在 HBM（高带宽内存）中，$n = 4096$ 时即为 $4096^2 \times 4\text{bytes} \approx 64\text{MB}$；实际模型 $n$ 可达 32K-128K，内存需求急剧膨胀
2. **内存访问瓶颈**：每个 attention 计算步骤都需要多次读写 HBM，访存开销远大于计算开销

### 3.2 核心思想：分块计算（Tiling）+ 在线 softmax

FlashAttention（Dao et al., NeurIPS 2022 / ICML 2023）通过**分块计算**和**算子融合**解决上述问题，在不改变输出数值（逐位等价）的前提下，将内存降至 $O(n)$。

**关键技巧：在线 softmax**

标准 softmax 需要遍历全部输入才能得到最终结果：
$$\text{softmax}(x_i) = \frac{\exp(x_i)}{\sum_j \exp(x_j)}$$

FlashAttention 采用**在线softmax技巧**，将 softmax 分解为两个累积量：分子（指数加权和）和分母（指数和）。在分块计算过程中，维护两个累积统计量：

$$m_i = \max(x_1, \ldots, x_i) \quad \text{（运行最大值）}$$
$$d_i = \sum_{j=1}^{i} \exp(x_j - m_i) \quad \text{（缩放后的指数和）}$$

最终 $\text{softmax}(x_i) = \frac{\exp(x_i - m_n) \cdot d_{i-1}}{d_n} + \frac{\exp(x_i - m_n)}{d_n}$，其中 $m_n$ 和 $d_n$ 是全局运行结束后的最终值。

### 3.3 分块执行流程

```
输入: Q (n × d_k), K (n × d_k), V (n × d_k)
输出: O (n × d_k)

1. 将 Q/K/V 分块为 T × T blocks（如 T = 64 或 128）
2. 初始化 O = 0, l = 0（行累计分母）, m = -∞（行累计最大值）
3. 对每个行块 i:
   a. 将 Q[i] 块加载到 SRAM
   b. 对每个列块 j:
      - 将 K[j], V[j] 块加载到 SRAM
      - 计算 S[i,j] = Q[i] · K[j]^T / √d_k
      - 更新 m_row = max(m_row, max(S[i,j]))
      - 更新 l_row += sum(exp(S[i,j] - m_row))
      - 更新 O[i] += exp(S[i,j] - m_row) · V[j]
   c. 最终 O[i] = O[i] / l_row
```

SRAM（共享内存）是 GPU 上比 HBM 更快但更小的内存，每次只将必要的小块数据加载进来，避免了 HBM 的 $O(n^2)$ 存储。

### 3.4 实际效果

| 指标 | 标准 Attention | FlashAttention |
|------|----------------|----------------|
| 内存复杂度 | $O(n^2)$ | $O(n)$ |
| 端到端加速 | 1x | 2-4x（与序列长度和硬件相关） |
| 内存占用减少 | 1x | 10-20x |
| 单卡可处理序列长度 | ~4K | ~32K-128K（取决于显存） |

FlashAttention-2（2023）进一步优化了 block 大小和 warp 分组，FA-3（2024）支持动态稀疏注意力模式。

### 3.5 FlashAttention 与 Agent 系统的关联

FlashAttention 是现代长上下文 Agent 的物理基础。Claude Code（[[entities/claude-code-core-internals|Claude Code]]）的五层 Context 压缩机制、git status 上下文注入、多文件代码库分析等能力，都建立在 FlashAttention 将可处理序列长度从 4K 提升到 128K 的基础之上。没有 FlashAttention，单次前向传播中"看到"整个代码仓库所有文件内容是不现实的。

---

## 四、Attention 模式与 Agent 行为

### 4.1 注意力分布作为模型"思维"的代理

Attention 权重 $A_{ij}$ 描述了模型在生成第 $i$ 个 token 时，对第 $j$ 个历史 token 的关注程度。分析注意力分布可以直接揭示模型的推理路径和决策依据。

**Agent 场景下的关键注意力模式**：

1. **工具选择（Tool Selection）注意力**
   当 Agent 决定调用某个工具时，注意力分布会在工具描述（tool schema）和任务描述之间形成强依赖路径。通过查看 $\mathbf{A}$ 中 tool schema token 对 tool name/description token 的注意力权重，可以判断模型是否真正"理解了"工具的用途，还是在随机选择。

2. **跨轮次依赖（Cross-turn Dependency）**
   在多轮对话中，Agent 的回复是否关注了前几轮的关键信息（如之前工具调用的失败原因），可以通过分析跨轮边界的注意力权重来量化。当 Agent 频繁"遗忘"早期上下文时，通常表现为远距离注意力权重衰减过快。

3. **规划链（Planning Chain）注意力**
   在复杂任务分解中（如 ReAct 循环），Agent 的"思考"token（thought token）与之前执行步骤的注意力连接强度，反映了模型的规划连贯性。注意力分散（所有 thought 均匀低权重）往往对应模型缺乏明确计划。

### 4.2 KV-Cache 与 Agent 上下文管理

自回归生成的标准实现对每个新 token 重新计算与所有历史 token 的注意力（$O(n^2)$）。**KV-Cache** 通过缓存已计算的 Key-Value 状态，将每步解码的计算复杂度降至 $O(1)$：

1. **Prefill 阶段**：对输入 prompt 完整执行一次注意力，缓存所有 K、V
2. **Decode 阶段**：仅计算新 token 的 Q，从缓存读取历史 K、V

KV-cache 的代价是内存随序列长度线性增长。一个 70B 模型在 4096 tokens 上下文时，KV-cache 可达数十 GB。

**Agent 中的 KV-cache 行为**：Claude Code 的 history 消息管理（主 Agent 10 轮、子 Agent 15 轮迭代上限）本质上是在控制 token 序列长度——超过上限后旧消息被截断，KV-cache 中对应内容被释放。这是应用层对注意力机制物理约束的一种妥协。

### 4.3 PagedAttention 与 Agent 并发

**PagedAttention**（vLLM，2023）借鉴操作系统虚拟内存的分页思想，将 KV-cache 划分为固定大小的 block（通常 16 个 token 一个块），通过块级管理和共享实现内存高效利用：

- **前缀复用**：多个共享相同 system prompt 的请求可以共享 KV block
- **按需分配**：避免预分配导致的内存碎片
- **复杂解码策略支持**：continuation、beam search、swap 等

对于 Agent 系统，PagedAttention 使得多个并发请求（多个用户同时使用同一个 Agent 服务）能够更高效地共享底层 KV 缓存，减少显存碎片，提升系统整体吞吐量。

### 4.4 Attention 模式异常与 Agent 可靠性

Attention 分布的异常模式可以预测 Agent 的潜在失败：

| 异常模式 | 表现 | 可能原因 |
|----------|------|----------|
| **注意力分散** | 所有历史 token 权重均匀分布 | 上下文过长，缓存失效，模型"迷失" |
| **过度聚焦最近 token** | 最近 1-2 个 token 占据 >80% 注意力 | 早期 token 的 KV 被截断或损坏 |
| **工具注意力偏移** | 模型关注的是工具名而非工具描述 | schema 注入位置不当，prompt 工程问题 |
| **循环注意力** | 两个特定 token 互相持续高度关注 | 模型陷入了某种循环引用/重复生成陷阱 |

通过监控 attention weights（可借助 `torch.nn.functional.scaled_dot_product_attention` 的 `enable_gqa` 或模型内部 hook），Agent 框架可以在运行时检测这些异常并触发干预（如截断 history、切换子 Agent、请求用户确认）。

### 4.5 AttnRes：对层间注意力的新探索

标准注意力是 token 级别（每个 token 关注哪些其他 token），**Attention Residuals（AttnRes）**（Kimi，2025）开创性地将注意力扩展到**层级**（每层关注哪些前驱层的输出）。

详见：[[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes]]

---

## 五、Variants 对比总结

| 变体 | KV 头数 | KV-Cache | 计算复杂度 | 质量 | 代表模型 |
|------|---------|----------|------------|------|----------|
| **MHA** | $h$（与 Q 头数相同） | $O(h \cdot n)$ | $O(n^2)$ | 最高 | GPT-4、Claude 3、LLaMA 1 |
| **MQA** | 1 | $O(n)$ | $O(n^2)$ | 略低 | PaLM、T5 早期 |
| **GQA** | $g$（$1 < g < h$） | $O(g \cdot n)$ | $O(n^2)$ | 接近 MHA | LLaMA 2/3、Qwen 2、DeepSeek-V2 |
| **LSH** | $h$ | $O(n \log n)$ | $O(n \log n)$ | 近似 | Reformer、KyBot |
| **Linear** | $h$ | $O(n)$ | $O(n)$ | 近似 | Performer、FLOP |
| **MRO** | 变体 | 变体 | 变体 | 任务相关 | Swin Transformer、Longformer |

---

## 六、Related

- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes]] — 注意力机制在跨层信息路由方向的新进展，用注意力替代残差连接的等权累加
- [[concepts/transformer-architecture]] — Transformer 整体架构，RoPE、KV-Cache、MoE、Norm 变体
- [[concepts/scaling-laws]] — 扩展定律，注意力效率优化是 scaling 的重要维度
- [[entities/claude-code-core-internals|Claude Code]] — FlashAttention 如何支撑长上下文 Agent 场景
- [[entities/agent框架owl原理详解|OWL]] — 多 Agent 框架，自注意力实现上下文感知
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw]] — Tool 系统的注意力输入空间约束
- [[concepts/inference-optimization]] — PagedAttention、Continuous Batching 等推理优化技术
- [[entities/deepseek-visual-primitives-thinking|DeepSeek Visual Primitives]] — 视觉原语作为注意力锚点，坐标化的指代机制如何提升多模态推理效率

## 关联实体

**上游依赖**:
- [[entities/claude-code-core-internals]] — 提供基础理论/方法
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres]] — 提供基础理论/方法
- [[entities/agent框架owl原理详解]] — 提供基础理论/方法

**下游应用**:
- [[entities/claude-code-core-internals]] — 具体应用场景
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres]] — 具体应用场景
- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres]] — 具体应用场景

**平行协作**:
- [[entities/claude-code-core-internals]] — 替代/补充方案
- [[entities/agent框架owl原理详解]] — 替代/补充方案
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-1-llm-principles|Layer 1 Llm Principles]]
- [[moc/wiki-master-map|Wiki Master Map]]
