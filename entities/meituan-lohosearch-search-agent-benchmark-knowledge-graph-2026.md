---

title: "LoHoSearch — 下一代搜索智能体评测基准"
created: 2026-07-24
updated: 2026-09-23
type: entity
tags: [search-agent, benchmark, knowledge-graph, evaluation, ai-agent, meituan, open-source]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LoHoSearch — 下一代搜索智能体评测基准

> 美团 LongCat 团队提出 LoHoSearch，一个利用知识图谱自动化构造搜索智能体评测题目的基准。通过构建覆盖 762 万维基百科实体的知识图谱，系统控制搜索空间与结构复杂度两个难度维度，突破了人工出题的难度上限。最强模型 GPT-5.5 准确率仅 34.74%。

→ [[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026|原文存档]] ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

## 背景：基准饱和与知识图谱出题

在 BrowseComp 等现有评测上，顶尖模型准确率已从最初的 30% 区间迅速攀升至 90% 以上，区分度持续递减。BrowseComp 题目由人工设计，局限在于只能基于标注者已知的实体和关系构思，无法从全局知识网络判断题目的真实难度。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

LoHoSearch 的核心创新是**让机器自己出题**——以大规模知识图谱为基础，自动生成难度可控的搜索智能体评测题目。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

## 构建流程

整个构建流程分为四个环节：

1. **建图**：从完整英文维基百科出发搭建大规模知识图谱，包含 **762 万个实体**、**2.65 亿条有向边**。每个实体类型取自 Wikidata P31 类别，热度用入度衡量。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

2. **双维度难度控制**：
   - **搜索空间**：满足条件的候选实体数量
   - **结构复杂度**：需要同时满足的条件数量
   
   针对这两个维度设计了树结构（放大搜索空间）和图结构（叠加结构复杂度）。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

3. **质量把关**：三层筛选——从子图抽取+改写为自然语言问题 → 自动验证 → 人工复核。自动化流程 75.5% 直接通过，22.3% 微调后接受，仅 2.2% 被丢弃。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

4. **最终数据集**：**544 道**经人工核验的题目，覆盖音乐、地理、影视、体育等 11 个主题领域。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

## 关键发现

**模型表现**：最强模型 GPT-5.5 准确率仅 34.74%；DeepSeek-V4-Pro、Claude-Opus-4.6、Kimi-K2.6 集中在 15.53%–15.99%，与 BrowseComp 上 80%+ 的表现形成鲜明对照。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

**工具调用增长**：解一道 LoHoSearch 题目平均工具调用从 BrowseComp 的 35 次增至 61 次（+74%）。图结构题目准确率 8.01%，远低于树结构的 11.89%。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

**重复采样收益**：pass@N 从 N=1 的 9.3% 升至 N=16 的 38.3%，但仍有六成以上题目无法攻克。best-of-N 仅 24.6%，说明模型置信度校准存在不足。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

**上下文管理策略失效**：最优策略（Discard-all + Verify）仅提升 6.8 个百分点，远低于在 BrowseComp 上的 14.03%，表明 LoHoSearch 需要更长的推理链，简单轨迹压缩无法解决信息丢失。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

## 意义与开源

LoHoSearch 的三项核心贡献：
- **基于知识图谱的自动化构造流程**，突破人工出题难度上限
- **更具区分度的评测标准**，为搜索智能体建立新标尺
- **面向上下文管理研究的挑战性平台**，推动下一代技术研究 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

已全面开源：Paper (arXiv 2606.12837) | [HuggingFace 数据集](https://huggingface.co/datasets/meituan-longcat/LoHoSearch) ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md]

## 相关实体

- [[entities/meituan-longcat-vitabench-20-long-term-dynamic-agent-benchmark|美团 LongCat 开源 VitaBench 2.0：长期动态智能体基准新标杆]] — 美团 LongCat 团队的另一个智能体基准
- [[entities/agent-evaluation-systematic-guide-metrics-to-closed-loop|Agent 评测体系化指南]] — Agent 评测方法论

## 深度分析

### 结构复杂度是独立于搜索空间的难度来源

用 DeepSeek-V4-Flash 作探针对比两个基准：同一模型在 BrowseComp 上准确率 58.84%，在 LoHoSearch 上仅 10.02%；平均工具调用从 35 次增至 61 次（+74%），中位数从 26 升至 59。更关键的是，图结构题目准确率仅 8.01%，低于树结构的 11.89%——两者搜索空间都被放大，唯独图结构额外引入环形依赖与交叉约束，说明长推理链本身（而非候选数量）就是独立的难度维度。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:99-105]

### 人工出题存在系统性偏差，知识图谱出题是对症下药

对比两个基准的"隐藏实体"特征可发现两层偏差：其一，BrowseComp 隐藏实体流行度明显更高，人工标注者倾向围绕自己熟知的实体构思，导致题目偏易；其二，即便将流行度控制在同一水平，LoHoSearch 的关系搜索空间仍显著更大，实体推断难度远高于 BrowseComp。这说明人工出题的局限不是个别标注者的能力问题，而是"人只能基于已知实体关系出题"这一机制的结构性缺陷，知识图谱因此成为系统化构造高难度题目的不可或缺的基础。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:125-134]

### pass@N 与 best-of-N 之间的落差暴露置信度校准缺口

对 DeepSeek-V4-Flash 采样 16 个独立回答，pass@N 从 N=1 的 9.3% 升至 N=16 的 38.3%，收益可观；但 best-of-N 聚合仅 24.6%，与 pass@16 上界相差近 14 个百分点。模型"曾经答对过"却"选不出正确答案"，指向答案置信度校准的明显不足——采样扩展策略的天花板不在于模型多样性，而在于聚合时能否可靠识别正确轨迹。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:109-113]

### 上下文管理策略在长程搜索下失效，指向信息丢失而非容量不足

以标准 ReAct 为基线，表现最佳的组合（Discard-all + Verify）仅将成绩从 10.02% 提至 16.82%，绝对提升 6.8 个百分点，而同一套策略在 BrowseComp 上可带来 14 个百分点的增益。收益近乎腰斩的原因是 LoHoSearch 需要更长的推理链：简单的轨迹压缩或重启无法解决长程搜索中的信息丢失问题。这表明当前上下文管理技术的问题不在"存不下"，而在"存下了但长程依赖断裂"——这类基准因此成为下一代上下文管理技术更有价值的试验场。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:117-121]

## 实践启示

- **评测选型**：BrowseComp 上 90%+ 的饱和分数已无区分度，评估搜索智能体时应换用 LoHoSearch——最强模型也仅 34.74%，且其 544 道题覆盖 11 个领域，可按领域切片定位模型的短板方向。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:97-97]
- **Agent 工程预算**：解一道 LoHoSearch 题平均需要 61 次工具调用（BrowseComp 仅 35 次），工程实践中为搜索智能体配置工具调用预算、超时与重试上限时，应按长程任务（60+ 步）规划，而非按 BrowseComp 量级（35 步）估算。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:99-105]
- **上下文管理迭代方向**：若在改进 Summary/Discard 类策略，应选 LoHoSearch 而非 BrowseComp 作为迭代基准——后者上 14 个百分点的增益在前者上缩水到 6.8，只有在更难的基准上仍站得住的策略才是真改进。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:144-144]
- **答案聚合策略**：在推理预算允许时优先用 pass@N + 人工/规则复核，而非直接信任 best-of-N 的自选结果（24.6% vs 38.3%）；在模型侧则应投资置信度校准，让"多数聚合"能真正选出正确轨迹。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:109-113]
- **基准构造方法论迁移**：建图（762 万实体 + 2.65 亿边）→ 双维度难度控制（搜索空间 × 结构复杂度）→ 三层质量把关的流程是可复用的范式，可迁移到代码检索、多跳问答等其他需要可控难度的智能体评测构造上。 ^[raw/articles/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026.md:41-41]

## 关联

- 同题异语种孪生页：[[entities/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知]]（归并候选，提案卡 #11 批1）
