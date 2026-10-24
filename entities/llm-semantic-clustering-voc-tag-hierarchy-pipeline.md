---
title: "LLM + 语义聚类 VOC 四级标签体系 Pipeline"
created: 2026-07-13
updated: 2026-10-25
type: entity
tags: [llm, nlp, semantic-clustering, voc, user-feedback, pipeline, tag-hierarchy]
sources: [raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline]
confidence: 0.80
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# LLM + 语义聚类 VOC 四级标签体系 Pipeline

本文详细阐述了一套将非结构化用户评论提炼为层级标签体系（VOC，Voice of Customer）的完整 pipeline 设计。在每个设计点交代了对应的可调参数及其对输出的影响。示例数据使用公开的中文电商评论数据集（online_shopping_10_cats 手机类，2323 条）。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]

## 背景

任何有规模的产品都会积累海量用户声音——电商评论、应用商店评价、客服会话、问卷反馈。这些文本量大且非结构化。传统做法要么靠人工逐条打标（不可行），要么直接把评论丢给 LLM 让其"总结一下"（结果很快失控）。直接让 LLM 自由生成标签会遇到三个问题：标签乱（同一意思不同表述）、标签重（几千近义标签不收敛）、跨批次不可比。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]

## 四级结构

系统采用四级标签体系（L1-L4），每层有明确的语义分工：^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]


- L1：用户旅程阶段（框架固定，跨批次可对比）
- L2：任务维度（如产品质量、客服体验、物流）
- L3：具体评价指标
- L4：细粒度标签

L1 框架固定，同层互斥，可复用，可量化（每个标签带频次）。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]

## 三阶段 Pipeline

整套流程分三个阶段：

**Stage1 — 建 L1-L2 标签体系**：先从数据中提炼出一套稳定的两级标签框架（用户旅程 → 任务维度）。使用 Map-Reduce 模式分片提取、多轮归并、评估优化。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]


**Stage2 — 逐条打标 + 生成 L3-L4**：用框架给每条评论打上 L1-L2，再为每条评论生成更细的 L3-L4 标签。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]


**Stage3 — 精炼收敛**：将 Stage2 产出的成千上万个细标签归并、去重、过滤、对齐，收敛成一棵干净的标签树。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]


共 11 个步骤，每个步骤都有可调参数。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]

## 技术核心

- **LLM 与向量模型的分工**：LLM 负责语义理解、标签生成和归并；Embedding + 聚类负责语义聚合，将相似标签自动归簇
- **Map-Reduce 式标签生成与归并**：分片提取 → 多轮归并 → 评估优化
- **低频过滤与对齐**：通过频率阈值过滤噪音标签，人工对齐确保质量
- **结构化输出（Structured Output）**：确保 LLM 输出格式一致，便于下游处理
- **Prompt Cache**：大量 LLM 调用使用相同 prompt 模板，使用 Prompt Cache 优化成本^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]

## 成果与边界

文章附有完整代码、配置和参数速查表。适用边界包括：需要有足够样本量（建议千条以上）才能收敛出稳定标签树；对高度同质化的评论（如全是短评"好用"）效果受限。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md]

→ [[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline|原文存档]]

---
## 深度分析

**一、固定 L1 的本质是"量具校准"，不是分类偏好。** 扁平标签失效的根因不在分类精度，而在标签空间无界：同一批数据里"续航差 / 电池不耐用 / 待机时间短"会被反复重新发明，跨批次连顶层类目都对不上，趋势分析直接作废。四级结构把"框架"和"内容"拆成两种资产——L1-L2 是跨批次不变、长期复用的量具，L3-L4 是每批数据重新填充的读数；量具随数据漂移，读数就不可比。所以文章用 JSON Schema 的 enum 把 L1 锁死：模型不是"被请求别乱来"，而是"在输出协议上无法乱来"。这也是 L1 必须在看数据之前、从业务旅程定下来而非从语料归纳的原因——归纳出的顶层类目一定带着当批数据的口音。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md:42-55]

**二、LLM 与聚类的分工是"边界判断"与"密度估计"的分工，不可互相顶替。** 判断几千个细标签里哪些语义重复，是一个 O(n²) 的两两相似度问题：向量算一次矩阵就结束，让 LLM 在上下文里互相比对既慢又不稳；反过来，向量没有生成能力，既决定不了一条评论该打什么标签，也给不出人类可读的簇名。分界线：单条评论走向无界生成（温度 0.5），标签集合走向有界比较（确定性温度 0.2）。画错的后果很典型——只聚类不命名，得到一堆无法解释的簇；只让 LLM 归并，则遭遇上下文截断与命名不一致。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md:212-222]

**三、两个阈值反向配合，调的是树的"形状"而非"质量"。** L3 阈值偏低（合得狠，消除同层碎片），L4 阈值偏高（留得松，保住细分），二者共同锁定 L3:L4 比例；若 L4 被聚得太狠，每个 L3 只剩一个 L4，L3 就退化成别名层，失去"归类多个指标"的意义。但阈值不是万能开关：L1 粒度只影响顶层精简度（15 类 vs 12 类，L2 从 76 降到 53），树的深度上限由数据丰富度决定——用户若只零星提一句"低光拍摄"，阈值调到多松也只挂得住一个 L4。分清"参数压塌"与"数据没话说"，是最需要经验、也最难自动化的一步。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md:120-157]

**四、Embedding 的选择决定 L2/L3 边界能否稳定，而 0.80 不可迁移。** 选 bge-large-zh-v1.5 的前提是待编码对象为中文短标签；余弦相似度分布在模型间差异极大（各向异性程度不同），换模型、换语言、换领域语料，同一个"0.80"对应的合并力度就变了，只能重标定而不能照抄。更隐蔽的是：标签多为四到十个字的短语，缺少上下文，正是 embedding 最不擅长的区间——"屏幕清晰"与"屏幕显示不清"向量距离可能极近，语义却相反。文章把 L4 定义成"标签名: 一句话解释"，表面是给人看的文档化，实质是一次 embedding 质量干预：解释句把裸短语拉回有语境的句子，相似度才有判别力。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md:262-268]

**五、min_samples=1 把"取舍"的决定权从聚类移交给了规则。** 该设置意味着每个点都是核心点、不存在噪声类，聚类退化为 eps 图上的连通分量：好处是顺序无关、可复现，坏处是"只出现一次的孤立标签"不会被当噪声丢掉，而是各自成簇活下来。于是"要不要保留长尾"算法回答不了，只能由 filter_freq 这个显式规则回答——保留 freq>1 得 75 个 L3 / 80 个 L4，全保留则涨到 145 / 185，含多个 L4 的桶占比从 6% 翻到 15%。这是个值得记住的范式：让算法只管合并，把价值判断（保留还是丢弃）留给可审计、可回滚的规则；同理，evaluate_l4 允许改三四级却用代码层白名单锁住 L1-L2，也是把不可逆权限收回到代码里。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md:241-247]

## 实践启示

1. **先定 L1，再碰数据。** 顶层类目从业务旅程定、不从语料归纳，写进 schema 的 enum；prompt 里再强调一遍只算冗余保险（同 [[concepts/prompt-engineering-fundamentals|Prompt Engineering]] 里"硬约束优先于软请求"）。L1 一旦投产，改动成本约等于重建全部历史报表。

2. **每步落盘，按 stage 单步重跑。** 候选 L2、聚合后 L3、过滤后 L4 各留中间产物；否则一次跑完就无法区分"树太扁"是阈值问题还是数据问题，这是最高频的返工来源。

3. **调参前先做数据丰富度体检。** 抽几个 L2 看其下 L4 的数量分布；若 L3≈L4 是普遍现象而非个别现象，先回原始评论确认用户是否多角度评价过该维度，再决定是否下调 AGGREGATE_L4_SIMILARITY_THRESHOLD。把参数问题和数据问题混在一起调，是最常见的徒劳。

4. **用四个指标验收 taxonomy，而不是"看起来合理"。** 覆盖率（多少评论能落到 L4）、纯度（抽样人工标注一致率）、跨版本稳定性（同数据两次跑、升级模型后叶子 path 集合的 Jaccard）、下游可用性（能否直接出"哪个维度被提及最多"的报表）。覆盖率与纯度天然对立，低频过滤就是那个滑块，量化代价是 80 vs 185 个 L4。这套口径可与 [[concepts/data-quality-framework|数据质量框架]] 对齐，但多了一条后者常不强调的：跨版本稳定性。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md:352-367]

5. **review 预算全押在"合并"上，不押在"命名"上。** 命名错了改一个字，合并错了是不可逆的信息丢失——两个语义不同的 L4 被并成一个，原始评论再也追不回来。评审清单应是 merge_l4 与 L3 归一化的合并对（代表名、成员数、示例原文）。

6. **迁移到别的域，换的是 L1 与切分方式，不换的是分工。** 客服工单、代码评审、事故复盘都能套这副骨架，但工单是多轮长文本，须先按会话聚成"议题"再打标，不能按消息切；代码评审的 L1 应是提交生命周期（理解 / 设计 / 正确性 / 风格）而非用户旅程。不能迁移的是"短、观点密集"这个前提——文章自承换成客服会话，分片、prompt、few-shot 全要重做。同类落地可参照 [[entities/gaode-voc-hermes-multi-agent-auto-triage-2026|高德交易 VOC 自动排查]]，它把 VOC 用于问题定位而非标签统计。^[raw/articles/llm-semantic-clustering-voc-tag-hierarchy-pipeline.md:400-414]

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

