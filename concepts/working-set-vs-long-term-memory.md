---
title: "Agent working set vs long-term memory"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, working-set, long-term-memory, context-window, agent, trade-off]
sources: [entities/agent-harness-context-management-working-set, entities/存之有序治之有矩agent-记忆系统的工程实践与演进]
---

## 定义

working set 是 agent 当前 turn 必须看到的最小信息集（在 context window 内），long-term memory 是按需召回的外部知识库。两者的本质 trade-off：working set 多 → 推理质量高但 token 贵，long-term 多 → token 省但召回延迟和质量风险。

## 核心范式

- **working set 必含**：system prompt / 当前任务 / 最近 N turns / 必要 tool 描述
- **long-term 按需召回**：vector search / kg query / file search，结果回填 working set
- **滑动窗口**：超过 N turns 的对话自动压缩为摘要，原文进 long-term
- **调优杠杆**：N 大→质量↑成本↑，N 小→成本↓但易丢上下文

### 背景与提出

working set vs long-term memory 的区分最初来自操作系统和 CPU 架构的概念。CPU 的 register 文件相当于 working set（极快但容量极小），内存相当于 long-term（容量大但延迟高）。现代 CPU 通过 cache hierarchy 试图用较快的小存储承接热点数据，这个思路直接被 agent 系统借用——working set 就是 LLM context 这个「快速寄存器」，long-term memory 就是 vector DB / knowledge graph 这个「慢速内存」。

这个区分在 LLM 时代变得格外尖锐，原因在于 token 的经济学：2024 年 GPT-4o 的输入 token 成本是 $3-15/1M tokens，折合每 1K token 约 0.003-0.015 美元。一个 128K context 的单次调用成本轻松超过 1 美元，而同等信息塞进 vector DB 检索只需 0.001 美元以下。成本差异超过 1000 倍，使得 working set 的边界管理不再是工程细节，而是直接影响产品利润的核心决策 ^[entities/agent-harness-context-management-working-set]。

这个框架提出的背景是 2023 年中期的 context window 军备竞赛。Anthropic 将 Claude 的 context 扩展到 200K，OpenAI 将 GPT-4 Turbo 扩展到 128K，各家都在宣传「可以喂一整本书」。但实际使用中发现，往 context 里堆 10 万 token 的记忆，模型表现反而下降——原因被后来者研究清楚：模型对 context 中间位置的内容存在「lost in the middle」问题，记忆越多，有价值的上下文被稀释得越严重。

### 范式细节

**working set 必含**的组成有明确的优先级排序：第一层是 system prompt（通常 1-4K token），第二层是当前任务的 instruction（0.5-2K），第三层是必要 tool 描述（约 2-4K/token，取决于 tool 数量），第四层是最近 N turns 的对话历史（约 1K/token/turn）。前四层是固定的，加起来通常已经占用 5-15K token，第五层才是「按需填充」的弹性空间。这个优先级意味着，tool 描述是 working set 的主要成本来源——一个 50 tool 的 agent，光 tool 描述就占掉 100-200K context ^[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]。

**long-term 按需召回**的关键设计是召回粒度。早期实现是整文档召回——把和当前任务相关的整篇文档从 vector DB 取出塞进 context，结果是 context 膨胀严重但利用率低（整篇文档可能只有 3 段相关）。后来演进到「块级召回」（chunk-level retrieval），每个知识块 200-500 字，召回后只取 top-5 最相关的块，总 token 量控制在 2-5K。这个设计把召回精度从「文档级」提升到「段落级」，但也引入了块边界割裂语义的问题——一个跨越块边界的核心概念可能被错误拆分。

**滑动窗口**的实现有两种策略：固定窗口（最近 N turns 保留，之外的压缩为摘要）和动态窗口（按 token 总量硬限制，超出时优先丢弃低 relevance 片段）。固定窗口实现简单但不够智能——用户刚说完的关键约束可能因为窗口滑动就丢失了。动态窗口需要实时 relevance scoring，计算开销大，但在关键信息保留上更可靠。实践中多数系统选择「固定窗口 + 显式重要信息标记」作为折中 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]。

**调优杠杆**的量化经验：N=5 turns（约 1-2K token）时成本最优但质量有限；N=20 turns（约 4-8K）时大多数任务足够；N=50+ turns 时开始出现 diminishing return，模型在冗余上下文里找信号的能力反而下降。这个规律在不同模型上表现一致，但具体阈值因模型 context attention 机制而异——Anthropic 的 Claude 系列在 long context 上的表现普遍优于 GPT 系列，所以 Claude 的最优 N 可以更大。

### 局限与反对声音

working set 的最大局限在于「 relevance scoring 的不透明性」。用 vector similarity 作为 relevance 代理是最常见的做法，但相似度≠有用性——一个和当前任务在向量空间相邻的段落，可能因为时间久远（描述的是过时事实）或者语气不搭（不是用户想要的分析角度）而完全无用。如何在召回时融合时效性、来源可信度、用户偏好等多维度信号，目前没有标准解法。

另一个系统性问题是「记忆的召回精度 vs 召回召回率」的 trade-off：严格召回（只返回高度相关的记忆）可能遗漏重要信息（false negative），宽松召回（返回更多候选项）则让 working set 膨胀、引入干扰（false positive）。这个 trade-off 和信息检索领域的 precision/recall dilemma完全同构，只是 agent 场景下的评估更困难——precision 的损失可能不会立刻显现（模型用自己的知识补充了空缺），但累积下来会造成系统性幻觉。

滑动窗口的压缩策略本身也可能是信息瓶颈。把 50 turns 压缩成 200 字的摘要，压缩比 50:1，这个过程必然丢失细粒度信息。如果压缩算法不够好（比如用 naive 的 GPT-4 摘要），关键约束（比如「不要用 Python，要用 Go」）可能在压缩时被改写或忽略。改进方向是结构化压缩——保留「用户约束」「任务目标」「中间结论」等结构化 slot，而非自由文本摘要。

### 现实案例

[[entities/agent-harness-context-management-working-set|harness working set]] 是这个概念最直接的工具化实现。Claude Code 的 context management 设计里，working set 由三部分组成：system prompt（写死）、当前会话历史（滑动窗口）、以及通过 @file 显式注入的文件内容（不经过滑动窗口，直接追加）。这个设计的精妙之处在于把「隐式上下文」和「显式上下文」分开管理——模型自己学到的隐式知识通过 working set 的滑动窗口自动管理，用户显式提供的知识通过 @file 显式控制。

[[concepts/harness-context-window-management|harness 上下文窗口]] 里对这个问题的处理更进一步：它把 working set 分成了「必须常驻」（system prompt、核心 tool）和「按需加载」（知识库召回结果）两层，并给每层设置了独立的 token 预算。知识库召回结果的预算占总 working set 的 30-50%，超出时按 LTR（last time visited）排序淘汰。这个实现在 [[entities/hermes-wiki-9-step-auto-growing-knowledge-network|Hermes Wiki]] 的 AI team knowledge harness 中被采用，线上数据显示平均 working set size 稳定在 12K token 左右，相比无限制注入时的 45K token，cost per task 降低了 67%。

## 在 wiki 中的关联

- [[entities/agent-harness-context-management-working-set|harness working set]]
- [[concepts/agent-memory-substrate-three-layer|memory substrate 三层]]
- [[concepts/harness-context-window-management|harness 上下文窗口]]
- long context techniques
- [[concepts/context-engineering|context engineering]]

## 进一步阅读

- [[entities/agent-harness-context-management-working-set]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]

## 所属 MOC

- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
