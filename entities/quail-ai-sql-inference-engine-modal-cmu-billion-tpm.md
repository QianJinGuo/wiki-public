---

title: "Hitting a billion tokens per minute on one GPU by combining a query planner and an inference engine"
created: 2026-09-29
updated: 2026-11-26
type: entity
tags: ["inference-optimization", "ai-sql", "kv-cache", "query-planner", "vllm", "modal", "benchmark"]
sources: [raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm]
confidence: 0.7
---

# Quail：AI-SQL 推理引擎（Modal × CMU）

## 定位与问题

Quail 是 Modal 与 CMU 数据库研究者合作开源的 **AI-SQL 推理引擎**：不是"用自然语言生成 SQL"（Text-to-SQL），而是方向相反——用 SQL 扩展**程序化地构造（并消费）发给 AI 系统的 prompt**，如 `AI.IF(PROMPT("{customers.profile} might buy this: {products.description}"))`。这类负载主要出现在 BI 平台对半结构化数据（文档、自由文本字段）的模糊查询场景。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md]

关键洞察：AI-SQL 负载与 chatbot/coding agent 负载的推理特征完全不同——单个查询可能产生数百万条数千 token 的序列，且所需智能远低于前沿模型（小开源模型即可胜任）。朴素地将其送入为 agentic inference 优化的引擎是大规模低效的。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md]

## 核心技术：查询计划器 + KV cache 主动管理

核心胜利点：**拿到结构化查询后，可以预先排序请求以更好地 cache（和驱逐）KV**——这需要对推理引擎的调度层做轻度改造。同时，大量针对小模型的小请求会产生可观开销，而预先知道请求结构可以规避这些开销。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md]

## 实测性能

- H100 单卡处理超过 **10 亿 tokens/分钟（TPM/GPU）**，查询计划收益最大的 multi-join 查询上比同硬件 vLLM baseline **快 >10x**
- 全任务几何平均 **1.84x faster than vLLM**（含两个刻意设计用于暴露改进空间的查询）
- 在 Modal 上折合 **每十亿 tokens 不到 6¢** 的成本
- 部署形态：`modal.Image.from_registry("nvidia/cuda:13.0.1-devel-ubuntu24.04")` + `@app.function(gpu="H100!")`，文档数据经 `quail.DocumentProvider.from_table()` 接入

## 与已有 wiki 实体的关系

与 [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026]]（Jev 快判断层）互补：Jev 解决"Agent 判断题从大模型里拆出来"（模型侧），Quail 解决"结构化 prompt 负载的推理调度"（推理侧）。两者都指向同一趋势——**大量低智能密度、高吞吐的推理负载需要专门的系统工程**，而非通用 frontier 推理引擎。作者团队自称这是"推理与数据库交叉的开源性能工程的开端"，鼓励"expert-parallel"协作（数据库视角另有专文）。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md]

## 深度分析

### 为什么查询计划器对 LLM 推理至关重要

传统推理引擎面对的是客户端任意发起的请求，KV cache 的命中与驱逐只能被动应对。AI-SQL 把控制权翻转了——客户端只提交 SQL，查询展开成海量请求（单个 multi-join 查询可达数百万条数千 token 序列），而"查询计划器 + 推理引擎"联合体从一开始就拿到了整个查询计划的完整请求清单。于是缓存需求可以被预测：某个 cache 条目何时不再被需要完全可知，可以"无畏地驱逐"；未来访问的时间线可预排，prefetch 有了着落。加上负载以吞吐为纲、对 KV 取回延迟不敏感，多级 KV cache 也从"奢侈品"变成"可行项"。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:57-61]

计划器侧的技术大多相当经典：谓词下推穿过 join、按选择性排序 filter、深树上动态规划做 join ordering——问题被限制在 filter 与 join 之内，成熟数据库技术得以直接复用。真正的创新是把推理成本注入决策：join ordering 搜索额外跟踪先前计划留下的 KV 状态，防止某个"初看很差"的选项其实能被后续 join 复用其 KV；候选计划只有在 tokens、attention pairs、cached tokens 三项同时被占优（Pareto 意义的 dominated）时才淘汰，最后由 speed-of-light 成本估计定夺。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:84-92]

### KV cache 的 prefill 与驱逐策略：吞吐杠杆所在

序列推理分 prefill（生成绝大部分 KV）与 decode（生成绝大部分输出 token）两阶段，而 agentic 负载严重偏向 decode——decode 算术强度低，常被迫上跨节点 prefill-decode 分离架构（参见 [[entities/vllm-push-based-kv-transfer-disaggregated-inference-2026]]）。Quail 的结构性优势是干脆消灭 decode：AI.IF/AI_FILTER 类查询只需输出 truthy/falsy 的概率，最后一个 prefill forward pass 即给出答案，于是采样、CUDA Graph capture、speculative decoding 等为 decode 服务的机器都可省去。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:62-74]

驱逐侧同样被"全知"改写：join 中 suffix 文档的 KV 根本不写入 cache（零 decode、不做 N>2-way join），HBM 只留 anchor 这类真会被复用的条目。作者同时指出当前实现的边界——只缓存请求生命周期内的 KV 到 GPU HBM，请求结束即丢弃；而 Bloom filter、zone map 这类数据库元数据从不会被"用完就扔"，KV cache 在更便宜的高层存储理应跨请求存活，尤其 KV 表示远大于原始存储表示。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:113-116] ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:125-131]

### "单卡十亿 TPM"意味着什么：batching 与内存带宽

这个数字本质是 prefill-only 推理对 H100 的极限压榨。speed-of-light 成本模型只用两条 roofline 公式刻画瓶颈：`latency_bound = bytes / memory_bandwidth` 与 `compute_bound = flops / compute_bandwidth`——对 per-token attention 投影、MLP/MoE 层和跨 token attention 这三类"帐篷柱"操作，prefill 是典型的带宽受限负载，throughput 只能靠大 batch 饱和 HBM 带宽来获得。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:93-99]

为此 Quail 在两头做减法。数据侧：数百万条序列 + 小模型超出常规 tokenizer 后端的设计空间，token ID 预先算好存进 memory-mapped Arrow 文件，执行期零分词开销。模型侧：只做 filter 和 join，词表被砍到 truthy/falsy 等 8 个选项（boot 时已知），unembedding 矩阵从 vocab_size × latent 缩到 8 × latent——被极端结构化输出解锁的免费加速。内核侧用 Triton 融合 add-RMSNorm+fp8 量化、per-head RMSNorm+rotary，减少小 kernel 的 launch 开销；join 的 attention 用"递归/部分和"方案避免 anchor 在 FlashAttention 里被大量复制，suffix 结果以 LSE-rescaling 合并。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:104] ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:108] ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:112-118]

### 查询计划与缓存状态失配时的失败模式

计划器假设"请求结构提前可知"，一旦失配就从优势变负担。最直接的案例：处理 agent traces 时 Quail 反而输给 vLLM——这条基准被刻意加入以证明系统在做 trade-off 而非全面碾压。把 agent 会话历史存进数据库再查询会产生大量跨文档 prefix 共享，而 Quail 的计划模型无法建模这种共享；可能的补救是建 radix tree 索引挖掘 KV 复用。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:132-135]

第二类失配来自计划与执行的时序错位：join ordering 在搜索期跟踪 KV 状态，tokenization 虽发生在计划器、却在执行期被复用，两阶段的缓存假设必须一致，否则"为复用而保留"的计划反占 HBM。第三类是成本模型的系统性偏差：speed-of-light 估计刻意高估峰值性能（"到不了北极星，但能帮你朝北"），用它选出的计划在 kernel 融合不足、重叠不够时会低于预期——作者承认现实现距 speed-of-light 仍有缺口。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:90-96] ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:136-137]

## 实践启示

1. **拿到结构就拿到了缓存主动权**：只要请求清单在执行前可知（批处理、ETL 等），就应把 KV 驱逐从 LRU 启发式改成"精确知道何时不被需要"的确定性驱逐，并做 prefetch——这是 Quail >10x 收益的最大来源。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:57-61]
2. **先判断负载是 prefill 主导还是 decode 主导**：判断/分类类任务可以像 Quail 一样把输出压到极小词表甚至单 token，砍掉 decode 及其配套机器（采样、speculative decoding、prefill-decode 分离）；做不到时再考虑 [[concepts/inference-optimization]] 中的分解架构。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:62-74]
3. **成本模型糙一点没关系，方向要对**：两条 roofline 公式就够做计划决策与迭代期 sanity check，关键是系统性高估而非随机误差；给"北极星"留恒定偏差比追求精确更重要。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:93-96]
4. **别把"异常基准"当噪音**：Quail 在 agent traces 上输给 vLLM 是刻意设计的对照——新引擎必须展示自己的 trade-off 边界（这里是与通用 prefix-sharing 推理的边界），否则基准就是自嗨。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:132-135]
5. **KV 是资产不是垃圾**：请求结束就丢弃 KV 等于把 Bloom filter 用完就扔；在低成本存储层让 KV 跨请求存活，对 KV 表示远大于原始数据的负载收益尤其大。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:125-131]
6. **专用引擎应建立在开源"可复用核心"上**：Quail fork vLLM 的 forward pass、复用 Arrow/DataFusion 生态再定制内核，不从零造轮子；与 [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026]] 一起看，专用推理引擎时代的正确姿势是窄负载 + 宽生态。 ^[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm.md:106-108]

## 关联

- → [[raw/articles/quail-ai-sql-inference-engine-modal-cmu-billion-tpm|原文存档]]
- [[entities/jev-fast-compaction-judgment-layer-tencent-daryl-2026]] — 同为"低智能密度推理负载"工程化趋势的模型侧案例
- [[concepts/inference-optimization]] — KV cache 调度与吞吐优化方法面
- [[entities/vllm-v0-to-v1-correctness-before-corrections]] — 对照 baseline 引擎

