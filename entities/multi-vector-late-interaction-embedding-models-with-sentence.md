---
title: "Multi-Vector (Late Interaction) Embedding Models with Sentence Transformers"
created: 2026-08-19
updated: 2026-11-21
type: entity
tags: [embedding, retrieval, colbert, late-interaction, sentence-transformers, rag, multimodal]
sources: [raw/articles/multi-vector-late-interaction-embedding-models-with-sentence]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Multi-Vector (Late Interaction) Embedding Models with Sentence Transformers

## 核心思想：把「压缩损失」留到打分阶段

普通 dense embedding 把一段文本压成一个定长向量，罕见实体、精确 ID、长段落里的关键从句都要挤进同一个向量里竞争空间；多条件查询（如"绿色沙发+木腿+圆坐垫"）也会被折成一个点。多向量模型（late-interaction / ColBERT 家族）避开这层压缩：同样跑 transformer，但把每个 token embedding 投影到小维度（经典 128）并全部保留，一篇 9-token 文档变成 9×128 矩阵，而非 1×128 向量。^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md]

cross-encoder 交互最早（query 与 doc 一起过模型，最准但 doc 无法离线预计算）；bi-encoder 交互最少（一个点积，可离线索引、查询快）；late-interaction 介于两者之间——文档仍独立编码、可离线建索引，但打分时让每个 query token 与每个 document token 交互，保留 token 级匹配信息。^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md]

## MaxSim 算子

打分用 MaxSim：对每个 query token 取它与任一 document token 的最高相似度，再对 query 求和。token embedding 经 L2 归一化，每个点积都是 [-1,1] 的余弦相似度，总和落在 [-num_query_tokens, num_query_tokens]。它把单向量模型必须平均掉的 token 级匹配信息保留下来，通常换来更强的召回，代价是更大的索引。^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md]

## 多向量模型的形态与 SOTA 场景

多向量模型是**视觉文档检索**的当前 SOTA：文本 query 直接与页面图像匹配、无需中间 OCR 步骤。同一套 `sentence-transformers`（`pip install -U sentence-transformers`）即可加载各类 checkpoint、编码打分、接入检索栈、在页面图像上运行并控制索引成本；也覆盖音频与视频检索、token pooling、推理加速与评估。^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md]

## 互补定位

- 与 [[entities/instemb-instruction-following-embeddings-2026|InstEmB]]、[[entities/douyin-dme-multimodal-embedding-multimodal-retrieval|Douyin DME]] 等单向量/多模态 embedding 不同，本文聚焦 late-interaction 的工程使用（加载、编码、MaxSim 打分、视觉无 OCR 检索、索引降本）。
- 适用：RAG 检索、视觉文档检索、需要 token 级精确匹配的场景；代价是索引体积随 token 数线性增长。

## 深度分析

### 质量收益的真实边界：同 backbone 对照实验

原文最有说服力的证据是 LightOn 的控制变量对照：LateOn（多向量）与 DenseOn（单向量）使用同一个 ModernBERT backbone、同一份训练数据、同为 149M 参数，唯一区别是是否把 token 向量池化成每文档一个。在 13 个 NanoBEIR 数据集上，多向量在 9 个上占优，均值 0.6868 对 0.6764，约一个 NDCG 点的差距；在完整 BEIR 上差距也接近（57.22 对 56.20）。但它在 ArguAna、FiQA2018、SCIDOCS、SciFact 四个数据集上反而落败——这说明 late-interaction 的收益不是普适的"全面更强"，而是"同尺寸模型换取约一个点的召回，代价全部体现在索引体积上"。评估时应预期它是分布敏感的增益而非默认胜出。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:844-863]

### 索引成本：原始量级与压缩后的真实位置

成本数字要看两个层面。原始层面：4,874 篇 Natural Questions 段落编码出 608,414 个 token 向量（平均每段约 125 个），float32 下 311.5 MB，是 MiniLM dense 索引（7.5 MB）的约 42 倍。压缩层面：PLAID 类索引（如 fast-plaid）每个向量只存 centroid id 加量化残差，同样 60.8 万向量只占 92 MB——已经与 4096 维 dense 模型（Qwen3-Embedding-8B 约 80 MB）处在同一量级。再叠加 token pooling：按余弦距离做 Ward 层次聚类、以簇均值替代簇成员，pool_factor=2 时向量数减半（1.99x）且原文引述的 BEIR 实验保住 100.6% 检索性能，factor=3 时仍有 99.0%。"多向量索引必然爆炸"的直觉在压缩 + 池化之后并不完全成立，但代价确实随文档 token 数线性增长。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:90-98] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:788-795]

### 三种检索栈形态及其适用规模

原文给出三条路径，边界清晰。其一，穷举 MaxSim：4,874 篇在 RTX 3090 上 20 秒编码完，每次查询约 120ms，精确但时间和内存都随全库 token 总量线性增长，几千篇以内是最简单正确的选择。其二，专用索引：fast-plaid（无服务器、Rust 实现的 PLAID）5 秒建索引、11ms 查询；Qdrant 26.3 秒摄入、18ms 查询——在合理规模下（全量打分可行时）三个数据库复现了穷举 MaxSim 的分数到小数点后四位，fast-plaid 因近似设计略有漂移但排序不变。其三，retrieve-and-rerank：多向量模型退居重排器，dense bi-encoder 先取 top-50，MaxSim 只对候选重打分，token 向量全程是瞬态的；相比 cross-encoder 逐对前向，多向量重排是批量编码 + 一次矩阵乘法，候选多时明显更便宜。这与 [[entities/rag-chunk-embedding-rerank-pipeline|检索-重排流水线]] 的经典结构天然兼容，也和 [[entities/ettin-reranker-family|Ettin reranker 家族]] 一类的专用重排器形成替代关系。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:296-313] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:360] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:385-453] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:317-350]

### 分数语义：地板效应、不可比性与精确分解

MaxSim 打分有三个容易被误解的性质。一是分数聚集：最大值算子保证每个 query token 都能找到某个"还过得去"的最好匹配，加上 contextualized embedding 的各向异性（聚在窄锥内），候选分数普遍贴得很近——Red Planet 例子里 Saturn 含有字面短语"the Red Planet"，分数 11.0811 与冠军 Mars 的 11.1104 只差 0.03。二是跨模型不可比：分数对 query token 数求和，LateOn 把该查询编成 12 个 token，ColBERTv2 固定 padding 到 32 个，同一组文档分数完全落在不同区间；需要有界分数时应切到 MeanMaxSim（除以 query token 数）。三是精确可分解：每个分数点唯一归属于一个 query token 和一个 document token，可解释性实验里 `rich`、`mond`、`preliminary`、`final` 自匹配，`when` 匹配到 `since`、`play` 匹配到 `game`——而非lexical匹配（`live` 与 `inhabit` 相似度 0.94）说明这种对齐不依赖字面重叠；同时三个 special token 贡献了 22.7% 的分数却不携带任何查询内容，说明 interpretability 读数时要留意 marker 的份额。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:243-245] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:257-276] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:84] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:735-751]

## 实践启示

1. **长文档场景是 late-interaction 的最强入场理由。** dense 压缩按训练分布学习"保留什么"，文档越长被平均掉的关键信息越多；同 backbone 对照在长文档基准 MLDR 上多向量 77.92 对 dense 51.59。落地时先检查 checkpoint 的 `document_length` 截断上限（LateOn 为 300：662 token 的段落只返回 273 个向量，超出部分直接消失），可用 `encode_document(..., processing_kwargs={"text": {"max_length": 512}})` 单次调高，但索引会按比例膨胀。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:181]
2. **按语料规模选栈，不要一开始就建专用索引。** 几千篇以内直接穷举 MaxSim（约百毫秒级查询）即可；过了这个量级再在 fast-plaid（要 pip 装的 PLAID）、Qdrant（v1.10+）、Weaviate（v1.29+）、Vespa 之间选。注意 Elasticsearch/OpenSearch 目前只能用 MaxSim 重排候选、不能用它做召回。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:352-358] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:280-313]
3. **索引体积先看压缩后数字，再用 token pooling 砍前段。** 报 311.5 MB 之前先算 PLAID 压缩（本例 92 MB）；编码侧起手 pool_factor=2（减半向量、BEIR 上近乎无损），并用 `MultiVectorNanoBEIREvaluator` 在自己的语料上实测后再定更激进的因子。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:92-98] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:786-795]
4. **最低成本的接入路径是把多向量模型当 reranker。** 保持现有 [[concepts/rag-retrieval-augmented-generation|RAG]] dense 索引不动，bi-encoder 取 top-50 后用 MaxSim 重排，token 向量即用即弃、不落盘；与 [[entities/meituan-search-3-llm-semantic-embedding-ranking-2026|美团搜索的语义排序体系]] 中"召回-重排"分层思路一致，可用它替换或补充 cross-encoder 重排档。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:317-350]
5. **编码与打分的 API 纪律。** 多向量模型是非对称的：必须用 `encode_query`/`encode_document`（各自套用 marker 前缀、长度上限、skiplist 配方），不能用通用 `encode()` 混用两侧；MaxSim 分数跨 checkpoint 不可直接比较（query token 数不同），要阈值或跨模型对比时先切 MeanMaxSim。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:183-209] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:255-276]
6. **多模态下提早做池化与降采样。** 视觉文档一页图像产生约 755 个向量（文本段落约 125），token pooling 的收益点应前移；视频必须降帧率降分辨率（0.5 fps 低分辨率把 VRAM 峰值从 20.8 GB 压到 12.5 GB，排序不变），长音频按 30 秒分块。^^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:641] ^[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence.md:714]

→ [[raw/articles/multi-vector-late-interaction-embedding-models-with-sentence|原文存档]]
