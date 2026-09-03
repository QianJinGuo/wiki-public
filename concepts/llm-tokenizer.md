---
title: "LLM Tokenizer — 分词器训练与词表构建"
type: concept
tags: [llm-tokenizer]
related:
  - [[entities/llm-post-training-full-guide|LLM Post-Training 全景指南]]
  - [[entities/minimax-token-degradation-jiqia|MiniMax Token 退化问题]]
  - [[entities/lightseek-token-speed-inference|LightSeek Token 加速推理方案]]
  - llm
  - tokenizer
  - vocabulary
  - pretraining
  - data-preprocessing
created: 2026-05-09
updated: 2026-08-01
---
# LLM Tokenizer — 分词器训练与词表构建

分词器（Tokenizer）决定了模型的词表粒度和 token 序列长度。词表构建是 LLM 预训练的第一步，直接影响模型能力、推理效率和多语言处理效果。分词算法的选择和词表大小的设定是工程实践中最重要的超参数决策之一。

## 主流分词算法

### BPE（Byte Pair Encoding）

BPE 是一种基于频率的贪心合并算法，最初用于数据压缩，后被 GPT-2 等模型采用作为分词器核心算法。

**训练流程：**
1. 将文本切分为 Unicode 字符序列（byte-level 或 character-level 起始）
2. 统计所有相邻 token 对的共现频率
3. 迭代合并频率最高的 pair，生成新的 subword token
4. 重复直到达到预设词表大小

**优点：** 实现简单，训练效率高，词表覆盖可控
**缺点：** 贪心合并无法全局最优，相同文本可能产生不同分词结果

BPE 的核心问题在于其贪心合并策略——每次只选择局部最优的 pair 进行合并，这可能导致在全局视角下次优的合并被提前固定。GPT-2 使用的 BPE 实现基于 byte-level UTF-8 编码，可以处理任意 Unicode 字符，避免了 UNK 问题。

### SentencePiece

SentencePiece 是 Google 开发的开源分词库，实现了**Unigram Language Model**（ULM）分词，同时支持 BPE 和 Unigram 两种算法模式。它是当前最广泛使用的分词器训练框架，Mistral、LLaMA、Qwen、DeepSeek 等主流开源模型均采用 SentencePiece 训练词表。

**与标准 BPE 的关键区别：**
- SentencePiece 将所有输入（包括空格）视为原始 byte 序列，不依赖语言特定的预处理
- 内置 UNK token 处理，所有未登录词都会被映射到 [UNK] 而非报错
- 支持词汇表外（out-of-vocabulary）字符的动态处理

SentencePiece 的 Unigram 模式使用基于语言模型的训练目标，可以在给定词表大小约束下学习最优的 subword 分割。与 BPE 的贪心合并不同，Unigram 在训练过程中会评估每个候选 token 的语言模型似然度，从而选择全局更优的词表结构。

### Unigram Language Model（ULM）

ULM 是 SentencePiece 默认使用的训练算法，其核心思想是：**给定一个词表，寻找使训练语料似然度最大化的 subword 分割概率分布**。

**训练流程：**
1. 初始化一个大词表（通常基于字符级分词 + 高频字符 n-gram）
2. 使用 EM 算法迭代优化：
   - E步：根据当前词表计算每个词的最佳分词方式
   - M步：重新计算每个 subword token 的出现概率，更新词表
3. 迭代直到词表大小收敛或达到目标大小

**优势：**
- 训练目标直接优化似然度，词表质量更高
- 支持动态添加字符级别的 token 处理低频词
- 可以输出多个候选分词结果及其概率

**实践中**，ULM 通常产生比 BPE 更平衡的词表——高频词保留为完整 token，低频词被更细粒度地分解。这使得 ULM 在处理稀有字符组合（如专有名词、代码标识符）时通常优于 BPE。

## 词表大小权衡

词表大小（vocab size）是分词器设计中最关键的超参数，直接影响模型训练和推理的多个维度。

### 词表过大的问题

- **lm_head 参数量增大**：输出层权重矩阵为 `vocab_size × hidden_size`，32K 词表 vs 200K 词表在 hidden_size=4096 时，lm_head 参数量差异达约 670M 参数
- **Embedding 稀疏化**：大词表中大量 token 对应训练数据稀少，导致对应 embedding 欠训练
- **Positional Embedding 扩展困难**：某些架构的绝对位置编码需要与词表大小协调

### 词表过小的问题

- **序列长度膨胀**：同样的文本被切分为更多 token，序列长度增加带来计算量二次增长（Attention 复杂度 O(n²)）
- **上下文利用率降低**：模型需要在更长的上下文中才能访问相同的信息密度
- **跨语言不平衡加剧**：中文等字符密集语言的 token 序列往往比英文长 2-4 倍

### 业界主流词表配置

| 模型 | 词表大小 | 分词算法 | 特点 |
|------|----------|----------|------|
| GPT-4（传闻） | ~100K | BPE | 多语言优化 |
| LLaMA-3 | 128K | BPE | 主要服务英语 |
| Mistral-7B | 32768 | SentencePiece | 小词表高效率 |
| DeepSeek-V2 | 102K | BPE | 中英双语优化 |
| Qwen-2.5 | 152K | SentencePiece | 多语言覆盖 |

研究表明，在相同的模型参数预算下，词表大小存在一个最优区间——过大或过小都会损害性能。[[entities/llm-post-training-full-guide|LLM Post-Training 全景指南]] 中指出，后训练阶段同样需要关注词表覆盖度问题。

### 词表大小与 Scaling Laws

[[concepts/scaling-laws|Scaling Laws]] 研究表明，词表大小的 scaling 指数通常小于模型参数和训练 tokens 的 scaling 指数。这意味着盲目增大词表而不相应增加训练数据量是低效的。正确的做法是在固定训练 compute budget 下，联合优化模型参数、训练 tokens 和词表大小。

一个实用的经验规则是：词表大小应该与训练 token 数量的 10-20 倍相匹配，即 1B 训练 tokens 对应 10-20B 的词表大小。但这需要根据具体语言分布调整——多语言模型需要更大的词表来覆盖更多语言中的字符组合。

## 多语言分词挑战

### 字符级 vs Subword 级的语言差异

不同语言的 token 粒度分布差异巨大：

| 语言 | 平均 token/词 | 原因 |
|------|---------------|------|
| 英语 | ~1.3 | 拉丁字母 + 高频词完整保留 |
| 中文 | ~2.5-3.5 | 汉字作为独立语义单元 + 多字词 |
| 日语 | ~3.0-4.5 | 三种脚本混合（汉字+平假名+片假名） |
| 韩语 | ~2.5-3.0 | 谚文音节组合 + 汉字词 |
| 俄语 | ~2.0-2.5 | 西里尔字母 + 丰富的词形变化 |

中文分词的挑战在于：汉字本身具有独立语义，词边界不明确，且存在大量多字词（如"巧克力"是一个词而非"巧""克""力"三个独立语义单元）。传统中文 NLP 需要显式分词，但 LLM 的 subword 分词器可以学习最优的字符组合方式。

### 日语的特殊挑战

日语是 tokenization 难度最高的语言之一。[[entities/minimax-token-degradation-jiqia|MiniMax Token 退化问题]] 的研究发现，日语 token 退化比例高达 29.7%，远超其他语言的 3.3%-3.9%。这一现象的根本原因是：

1. **三种脚本混合**：汉字（Hanzi/Kanji）、平假名（Hiragana）、片假名（Katakana）共存，同一语义可能有多种书写形式
2. **字符集巨大**：常用汉字 2136 个，加上平假名 46 个、片假名 46 个，以及大量非常用汉字，总字符数超过数万个
3. **形态变化丰富**：日语动词和形容词的活用变化产生大量词形变体

日语的 token 退化会导致严重的语言混杂问题——模型在日语回答中混入俄文字符或其他语言的 token，这是因为不同语言的 token 在向量空间中的相邻关系被破坏。

### 多语言词表设计策略

**策略一：语言特定的字符集 + 共享 subword**
为每种语言分配独立的字符级 token 集合（如基础汉字、假名字母），在高频词级别使用跨语言共享的 subword tokens。DeepSeek-V2 和 Qwen 系列采用此策略。

**策略二：统一的 byte-level BPE**
将所有语言统一映射到 UTF-8 byte 序列，使用统一的 BPE 训练。这种方法语言无关，但会导致多语言 tokenization 效率较低。

**策略三：层次化词表**
底层为共享的 byte-level tokens，中间层为跨语言 subword，顶层为语言特定的完整词。这种设计在多语言效率和语言特异性之间取得平衡。

## Special Tokens

Special tokens 是非文本内容的控制标记，用于表示序列边界、任务指令、填充等特殊功能。

### 常见 Special Tokens

| Token | 名称 | 用途 |
|-------|------|------|
| [PAD] | Padding | 批处理时填充序列到相同长度 |
| [UNK] | Unknown | 未登录词替换标记 |
| [CLS] | Classification | 分类任务的序列表示token |
| [SEP] | Separator | 多段文本分隔标记 |
| [MASK] | Mask | MLM 预训练中的掩码标记 |
| [BOS] / \<s\> | Begin of Sequence | 序列起始标记 |
| [EOS] / \</s\> | End of Sequence | 序列结束标记 |

### Special Tokens 的训练问题

Special tokens 在预训练阶段的行为需要仔细管理：

1. **位置分布**：某些 token（如 [CLS]）总是出现在序列特定位置，模型可能学会利用位置偏差而非学习语义
2. **频率失衡**：部分 special tokens 在训练数据中出现频率极低，可能导致对应 embedding 欠训练
3. **跨任务迁移**：后训练阶段新引入的 special tokens（如工具调用标记）可能与预训练词表冲突

[[entities/minimax-token-degradation-jiqia|MiniMax 研究]] 揭示了一个关键问题：当 SFT 数据中特定 special tokens 出现频率极低时，对应的 lm_head 输出向量会发生显著漂移，导致模型无法正确生成这些 token。

### Chat Template Tokens

现代对话模型使用特殊的 template tokens 来结构化输入：

```
[INST] <<SYS>>
系统提示
<</SYS>>

用户问题 [/INST] 模型回答 [/INST]
```

这些 template tokens 的设计直接影响模型的指令遵循能力。错误的 template 设计可能导致模型忽略系统指令或产生格式混乱的输出。

## Tokenization 对推理成本的影响

### 序列长度与计算量

Tokenization 质量直接影响推理计算量：

- **Context 长度**：更短的 token 序列意味着在相同的上下文窗口中可以容纳更多语义信息
- **生成长度**：相同语义的回答，token 序列越短，首 token 延迟（TTFT）越低
- **总生成时间**：总 token 输出数直接决定生成任务的端到端延迟

### KV Cache 效率

在自回归生成过程中，KV Cache 的存储与 token 数量线性相关。更短的 token 序列意味着：

- 相同的 KV Cache 容量可以缓存更多上下文 token
- 内存带宽压力降低，推理吞吐量提升
- 长上下文场景下的 cache 命中率提高

### 词表大小与内存

推理时 lm_head 的权重矩阵完全加载在显存中。以 hidden_size=4096 为例：

- 32K 词表：约 512MB（FP16）
- 128K 词表：约 2GB（FP16）
- 200K 词表：约 3.2GB（FP16）

对于多卡推理，词表过大会增加 tensor parallel 通信开销，因为每个 token 的 logits 计算需要 all-reduce 汇总所有卡的局部计算结果。

### Tokenization 效率的实测差异

以"今天天气真好"（5个中文字符）为例：

| 分词器 | Token 数 | 编码效率 |
|--------|----------|----------|
| 中文 BPE（32K） | ~8-10 | 低 |
| 中文 BPE（200K） | ~5-6 | 高 |
| 英文 BPE（32K） | ~3-4 | 较高 |

相同的语义在中文和英文下的 token 效率差异可达 2-3 倍，这对多语言模型的计算资源分配提出挑战。

## 分词器训练数据

### 训练数据配比原则

分词器训练数据的选择和配比直接影响词表质量：

1. **语言分布代表性**：词表中各语言的 token 覆盖度应与模型预期使用场景一致
2. **领域覆盖**：代码、科学文献、数学公式等垂直领域需要专门的数据补充
3. **数据质量过滤**：去除低质量、重复、有毒内容，避免这些模式被编码进词表

### 预训练语料 vs 推理语料的分布差异

[[concepts/llm-pretraining-vs-sft|预训练与后训练数据分布差异]] 是导致 token 退化的根本原因。分词器基于预训练语料的统计特征构建词表，但模型在后训练阶段接触的数据分布往往与预训练显著不同：

- **预训练数据**：网页文本、书籍、代码库，主要为叙述性和知识性内容
- **SFT数据**：对话数据、指令遵循、工具调用，交互性内容占比高
- **RLHF数据**：人类偏好反馈，分布更加窄化和特定

这种分布差异导致预训练阶段高频的 token（如某些领域的专有名词）在后训练中变为低频或缺失，而预训练中低频的 token（如 tool_call 标记、特殊格式符号）在后训练中反而成为高频。

### 领域适配词表

针对特定领域（如代码、医学、金融）训练专用词表可以显著提升 tokenization 效率：

- **代码分词器**：保留完整的编程语言关键字和操作符，避免常见代码结构被过度分解
- **多语言混合分词器**：在通用词表基础上增加特定语言的字符组合优化

### 词表训练的数据预处理

训练 SentencePiece 前的数据预处理关键步骤：

1. **Unicode 标准化**：NFKC 规范化确保同一语义的不同 Unicode 表示被统一处理
2. **去重**：移除重复内容，避免高频模式过度编码
3. **采样**：控制不同来源数据的采样比例，确保语言和领域分布均衡
4. **长度过滤**：移除过长或过短的序列，减少噪声

## Token 退化问题

Token 退化（Token Degradation）是 [[entities/minimax-token-degradation-jiqia|MiniMax 2026年研究发现]] 的一种后训练问题，本质是预训练-后训练数据分布失配在 token 粒度上的体现。

**核心机制：**
- 分词器基于预训练语料的统计频次决定 token 合并策略
- 后训练对话数据的分布与预训练差异巨大——大量在预训练中常见的 token 在 SFT 数据中出现频次极低
- 高频出现的 SFT token 持续更新周围向量空间，把低频 token 挤压到错误方向
- lm_head 权重向量偏移，余弦相似度大幅下降，Norm 变化显著

**全词表"复读任务"** 是当前最有效的工程解法：通过合成数据为全词表建立一个生成频率的"下限保障"，防止任何 token 因为完全缺失而退化。

## 相关概念

- [[concepts/llm-pretraining-vs-sft]] — 预训练与后训练的数据分布差异
- [[concepts/scaling-laws]] — 词表大小与模型性能的 scaling 关系
- [[concepts/catastrophic-forgetting]] — 稀疏 token 退化的理论基础
- [[concepts/inference-optimization]] — Tokenization 对推理效率的影响
- [[entities/llm-post-training-full-guide|LLM Post-Training 全景指南：从RLHF到GRPO再到AgenticRL]]
- [[entities/minimax-token-degradation-jiqia|MiniMax Token 退化问题研究]]
- [[entities/lightseek-token-speed-inference|LightSeek Token 加速推理方案]]

## 来源

词表构建是 LLM 基础设施的核心环节，参考 GPT-4、Mistral、DeepSeek、Qwen、MiniMax 等模型的分词器设计实践。

## 关联实体

**上游依赖**:
- [[entities/llm-post-training-full-guide]] — 提供基础理论/方法
- [[entities/minimax-token-degradation-jiqia]] — 提供基础理论/方法
- [[entities/lightseek-token-speed-inference]] — 提供基础理论/方法

**下游应用**:
- [[entities/llm-post-training-full-guide]] — 具体应用场景
- [[entities/minimax-token-degradation-jiqia]] — 具体应用场景
- [[entities/minimax-token-degradation-jiqia]] — 具体应用场景

**平行协作**:
- [[entities/minimax-token-degradation-jiqia]] — 替代/补充方案
- [[entities/llm-post-training-full-guide]] — 替代/补充方案
- [[entities/minimax-token-degradation-jiqia]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-1-llm-principles|Layer 1 Llm Principles]]
