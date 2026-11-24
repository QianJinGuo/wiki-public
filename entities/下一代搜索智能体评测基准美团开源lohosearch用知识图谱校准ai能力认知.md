---
title: "下一代搜索智能体评测基准！美团开源LoHoSearch，用知识图谱校准AI能力认知"
created: 2026-08-13
updated: 2026-11-24
type: entity
tags: [ai, research, agent, ai-agent, multi-agent, evaluation, benchmark, agent-eval, graphrag, kg, rag, search, agent-search]
sources: [raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md]
confidence: 0.6
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 下一代搜索智能体评测基准！美团开源LoHoSearch，用知识图谱校准AI能力认知

> WeChat-美团技术团队 | 发布于 2026-07-23 | 评分入库 v×c≥49

## 核心内容

美团 LongCat 2026-07-23 19:58 北京 从人工出题到知识图谱自动构造，LoHoSearch 给 GPT-5.5 上了一课。 过去一年，我们见证了 Search Agent 能力的显著演进。 在 BrowseComp 等评测上，顶尖模型准确率从最初的30%区间迅速攀升至90%以上。然而，当基准迅速饱和，其区分模型能力的价值也随之递减。 △图1：BrowseComp 准确率进展曲线 BrowseComp 的题目由人工设计，局限在于只能基于标注者已知的实体和关系构思，无法站在全局知识网络视角判断：哪些条件真的难检索？哪些约束的候选空间足够大？ 正是这种局限，让我们开始思考另一种可能性：能不能让机器自己来出题？ 美团 LongCat 团队在最新论文中提出的 LoHoSearch 基准，就是把这种可能性变成了现实。 🚀 LoHoSearch 的核心"硬核"看点： 知识图谱自动化构造。 以覆盖762万维基百科实体的知识图谱为基础自动生成题目，替代人工出题。基准含544道经人工核验的题目，覆盖11个领域。 双维度难度控制。 在生成中系统控制搜索空间与结构复杂度。构建出难度显著高于现有基准的挑战性问题。 当前模型性能表现。 已评测模型中，GPT-5.5准确率为34.74%；现有上下文管理策略在LoHoSearch上提升幅度为6.8%，低于在BrowseComp上的14.03%。 01 设计原理：让机器出题，需要几部？ 机器出题的前提，是让机器拥有全局视野。整个构建流程可以分为四个环节：建图 → 控制难度 → 质量把关 → 数据概览 。下面逐一展开。 1.1 建图：搭建知识。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md]

## 关键要点

- 原文完整记录：[[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md|原文存档]]
- 关联主题："Agent 架构"、[[concepts/agent-orchestration-patterns]]、"Agent 评估基准体系"

## 相关实体

"Agent 架构" [[concepts/agent-orchestration-patterns]] "Agent 评估基准体系" [[concepts/evaluation-harness-design]] "Agentic RAG 模式"

## 深度分析

### 基准饱和催生的"机器出题"范式

BrowseComp 等评测上顶尖模型准确率一年内从 30% 区间冲到 90% 以上，基准迅速饱和后区分度随之递减。更根本的瓶颈在出题端：人工出题只能基于标注者已知的实体和关系构思，无法站在全局知识网络视角判断哪些条件真的难检索、哪些约束的候选空间足够大。LoHoSearch 的回答是把出题权交给知识图谱——让"难"变成可采样、可参数化控制的变量，而不是依赖出题人的手感。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:18-24]

### 难度的两轴分解：搜索空间 × 结构复杂度

这项工作最有方法论价值的抽象，是把搜索难度拆成两个正交维度：搜索空间（满足单一条件的候选实体规模，决定排除成本）与结构复杂度（须同时满足的条件数量与咬合程度，决定求解链条长度）。两者对应两种子图结构：树结构主要放大搜索空间，图结构则在其上叠加环形依赖与交叉约束。后验数据支持这个分解——图结构题准确率仅 8.01%，显著低于树结构的 11.89%，印证结构复杂度是独立于搜索空间之外的额外难度来源。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:66-79] ^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:111-117]

### 三层验证管线的质量经济学

机器出题不等于可用题目，LoHoSearch 用三层漏斗兜底：题目生成（从子图抽取维基描述改写为自然语言）→ 自动验证（检查子图关系覆盖度，并由搜索智能体验算标准答案满足全部条件）→ 筛选与人工复核（排除多答案题与易答题）。最终 75.5% 的题目直接通过人工复核，22.3% 经标注员微调后接受，仅 2.2% 因严重问题被丢弃。这组数字的含义是：只要图谱底座与自动验证器设计得当，人工只需承担最后一段质检，出题成本即可较纯人工方式下降多个数量级。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:88-97]

### 评测结果反推出的三个短板信号

其一，长程推理的真实成本：同一探针模型（DeepSeek-V4-Flash）在 BrowseComp 上 58.84%、在 LoHoSearch 上仅 10.02%，正确轨迹的平均工具调用从 35 次增至 61 次（+74%），中位数从 26 升至 59。其二，置信度校准缺失：采样 16 个回答后 pass@16 达 38.3%，但 best-of-N 聚合只有 24.6%，远低于上界——模型并不知道自己哪个答案更可信。其三，现有上下文管理策略失真：表现最佳的 Discard-all + Verify 组合仅提升 6.8 个百分点，而同一套策略在 BrowseComp 上可带来 14 个百分点增益，说明简单的轨迹压缩或重启解决不了长程搜索中的信息丢失问题。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:113-117] ^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:121-125] ^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:129-133]

## 实践启示

1. 评测自家搜索/检索类 Agent 时，饱和基准（如 BrowseComp 类人工题库）得分高不等于能力强，应补测 LoHoSearch 这类低饱和度基准：GPT-5.5 也只有 34.74%，头部模型集中在 15%-16%，区分度远高于 90%+ 的旧基准。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:109-110]
2. 自建评测集时引入"难度两轴"参数化设计：显式控制每题的候选实体规模（搜索空间）与条件咬合度（结构复杂度），而不是靠出题人直觉定难度；知识图谱（实体节点 + 页面超链接边）是系统化构造难例的可复用底座。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:137-145]
3. 上下文管理策略的收益要用长链任务重新校准：Summary/Discard 等压缩策略在短链基准上的增益会严重高估其在长程搜索中的实际价值（6.8% vs 14.03%），选型测试必须包含工具调用 50+ 次量级的任务。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:129-133]
4. 用重复采样提升命中率之前，先解决置信度校准：best-of-N 聚合（24.6%）远低于 pass@16 上限（38.3%），意味着"多采几次"的收益大部分被"不会挑答案"吃掉，优先投入答案打分/校准模块而非单纯堆采样次数。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:121-125]
5. 把工具调用次数分布（均值 + 中位数）当作 Search Agent 的健康指标持续追踪：LoHoSearch 上均值 61 次、中位数 59 次的高消耗既是难度的信号，也是成本优化的直接靶点。^[raw/articles/下一代搜索智能体评测基准美团开源lohosearch用知识图谱校准ai能力认知.md:115-117]
6. 该基准与 [[entities/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026]]（孪生页）可对照阅读；同属美团 LongCat 的长程 Agent 评测还有 [[entities/meituan-longcat-vitabench-20-long-term-dynamic-agent-benchmark]]，搜索方向的同类基准可参考 [[entities/atlas-agentic-search-benchmark-exa]]；上下文管理策略失真的问题域详见 [[concepts/context-management-agent-systems]] 与 [[concepts/agent-evaluation-benchmark-frameworks]]。

## 关联

- 同题异语种孪生页：[[entities/meituan-lohosearch-search-agent-benchmark-knowledge-graph-2026]]（归并候选，提案卡 #11 批1）
