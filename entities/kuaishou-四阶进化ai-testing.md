---
title: "快手AI测试四阶进化实践 — Prompt→Multi-Agent→知识工程→Agentic自进化"
created: 2026-07-09
updated: 2026-09-11
type: entity
tags: [ai-testing, ui-testing, test-case-generation, agentic-workflow, knowledge-engineering, multi-agent, rag, badcase-driven, self-evolving, kuaishou, harness]
sources:
  - raw/articles/智能ui用例生成与执行的四阶进化实践
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 快手AI测试四阶进化实践 — Prompt→Multi-Agent→知识工程→Agentic自进化

快手研发Agent负责人苗星在 AiDD 大会分享的 AI+测试演进路径：过去两年，团队从 Prompt 工程一路走到 Agentic 自进化，用例生成率从 8% 提升到 70%，覆盖全公司 120W+ 用例。这不是一份模型能力报告，而是一份"把 AI 嵌进既有 QA 工作流"的工程演进记录。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]

## 摘要

快手把 AI 测试的能力增长拆成四个阶段：Prompt 工程（8% 生成率）、Multi-Agent 协作（15%）、知识工程（35%）、Agentic 自进化（70%），每一步的瓶颈都不在模型，而在知识供给与人机分工的设计。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]贯穿全程的核心假设是：用例不是"一次生成就完成"的产物，而应像代码一样经历设计、评审、迭代、沉淀的全生命周期。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]

## 核心要点

- **瓶颈诊断先行：** 用例编写只占约 13% 的工作量，执行却消耗约 38% 的资源。因此"自动化执行"并没有打中真正的约束；稀缺的是用例的来源、质量保证和规模复用。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]
- **用例生命周期假设：** 用例应像代码一样被设计、评审、迭代、沉淀——这解释了为什么单次生成永远不够，只有把评审与沉淀做成结构，质量才能持续爬升。
- **四阶演进与量化曲线：**

| 阶段 | 方法 | 核心能力 | 生成率 | 采纳率 |
|------|------|----------|--------|--------|
| V1.0 | Prompt工程 | Few-shot + 场景化Prompt + 输出强约束 | 8% | 40% |
| V2.0 | Multi-Agent协作 | PRD解析→用例生成→用例评审 Agent | 15% | 55% |
| V3.0 | 知识工程 | RAG + 多模态 + 历史缺陷 + 业务私域知识 | 35% | 65% |
| V4.0 | Agentic自进化 | ReAct Loop + Review-Critique + BadCase自进化 | 70% | — |

- **V1 的三个根本失败：** 长文档理解衰减（PRD 超 30K tokens 后遗忘前文）、业务知识缺失（隐含规则不会被自动推导）、生成过程黑盒（用户无法干预中间步骤）。
- **V2 复制工作流：** 把 QA 流程克隆成 PRD解析 → 用例生成 → 用例评审三段 Agent，AI 负责生成与框架，人负责 Review 决策；生成率 +87%，采纳率 +37%，但新天花板是"正确但浅显"。
- **V3 注入四类知识资产：** 文本测试物料、多模态物料、历史用例 RAG、业务私域模板——历史缺陷覆盖率从 12% 跃升到 76%，代价是 170+ 私域模板需专人维护且更新滞后。
- **V4 自进化闭环：** 三层资产（170+ Skills、Memory、Knowledge 图谱）+ ReAct 循环 + 双级 Review-Critique + BadCase 规则库；模板更新从天级压缩到 5 分钟，维护成本下降 99%。
- **执行侧走第三条路：** 既非纯脚本也非纯自由探索，而是"测试意图驱动"，前提是 AI-Friendly 用例（原子化步骤、可观测预期、环境依赖显性化）与执行引擎的感知→决策→执行→反馈四能力。

## 深度分析

### 瓶颈诊断决定了整个技术路线的走向

这个案例最有价值的地方是它的起点不是"用哪个模型"，而是一次成本结构诊断：用例编写仅占 13%，执行环节却吃掉 38% 的资源；同时资深 QA 与新手写出的用例质量天差地别，历史缺陷难以复用，PRD 频繁变更又推高维护成本。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]如果只盯执行自动化，投入产出比会被这个错误的目标函数稀释。快手把资源押注在"用例从何而来、质量能否保证、怎样规模复用"上，本质是把问题重新定义成一个**知识供给问题**，而不是一个推理能力问题。这也解释了后续四阶演进里，每一阶增加的主要不是模型调用技巧，而是知识资产与流程结构的密度。

### 从 Prompt 到知识工程：质量上限由知识丰富度而非 Prompt 精妙度决定

V1 的失败很有代表性：3-5 个 few-shot 样本加上场景化模板和输出强约束，看似把 Prompt 打磨到位，生成率却只有 8%，用户反馈"生成得太泛了，没法直接用"。三个根因——超 30K tokens 的长文档衰减、隐含业务规则缺失、生成过程黑盒——都不是靠再优化措辞能解的。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]V2 用 Multi-Agent 把 QA 工作流显式复制成 PRD解析、用例生成、用例评审三段，借"分阶段审查"结构把生成率推到 15%、采纳率推到 55%，但它暴露的新天花板是"正确但浅显"——结构对了，知识还是空的。V3 才真正打中要害：注入文本物料、多模态物料、历史用例 RAG 与业务私域模板四类资产。一个具体例子是生成"直播送礼"用例时，系统会通过 RAG 自动召回历史缺陷"并发送礼导致重复扣款"，把曾经的线上事故直接变成新用例的覆盖点，历史缺陷覆盖率因此从 12% 升到 76%。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]这条曲线给出的结论是硬的：**知识丰富度决定质量上限，Prompt 精妙度只决定你能多快撞到那个上限。**

### BadCase 自进化：把低质量用例变成系统性燃料

V3 的隐痛也随之浮现——Review 成本依旧高，170+ 业务私域模板需要专人维护，知识更新有滞后。V4 的答案是把"沉淀"本身自动化，用三条主线闭合飞轮：一是自主决策与 ReAct Loop，靠 170+ 定制 Skills、Memory 与 Knowledge 图谱三层资产支撑；二是自主评审，Review-Critique 分两级——模块级看覆盖完整性、结构合理性、层级清晰度、命名规范性，用例级看 PRD 覆盖度、场景完整性、步骤清晰度、预期明确性；三是 BadCase 规则库，把低质量用例反向抽取成模式，进入规则库后自动召回并纠正下一次生成。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]结果是场景规则模板更新从天级降到 5 分钟、维护成本下降 99%，生成率从 35% 冲到 70%。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]这里的工程含义很关键：**BadCase 的系统化收集是自进化唯一的可靠燃料**——没有结构化的失败样本，再精巧的 Agent 循环也只是在原地打转。

### 从"能生成"到"能可靠执行"：测试意图驱动的第三条路

生成率上去之后，问题自然移到执行端。快手没有在"纯脚本"与"纯自由探索"之间二选一，而是选择测试意图驱动：用例声明意图，执行引擎负责把它落地。这条路有两个硬前提：用例必须 AI-Friendly——精简全面、步骤原子化、预期可观测、环境依赖显性化；执行引擎必须具备"四觉"能力——感知、决策、执行、反馈。^[raw/articles/智能ui用例生成与执行的四阶进化实践.md]换个角度看，AI-Friendly 用例规范同时约束了人怎么写和机器怎么读，它把过去散落在 QA 脑中的隐性约定变成了显式契约。这也呼应了人工与 AI 的分工边界：高频、可枚举的工作交给 AI，而涉及业务判断与关键纠偏的决定权始终留在人手里。

## 实践启示

1. **先做瓶颈诊断再选技术路线。** 别默认"自动化执行"就是答案；先量一量各环节的真实工作量占比（如 13% 编写 vs 38% 执行），把资源压到真正的约束上。
2. **为任何工作流复制"分阶段审查"结构。** V2 的收益主要来自把流程显式拆成生成与评审两段，这个手法几乎可以平移到任何内容生成场景。
3. **从 V1 就要设计反馈闭环。** 不要让 BadCase 在聊天记录里流失；把失败样本的收集通道当成第一等公民来建设。
4. **投资顺序是：历史缺陷库 > 业务规则模板 > Prompt 调优。** 三者的质量杠杆依次递减，把预算优先花在知识资产而非提示词技巧上。
5. **从高价值低复杂度场景切入。** 先打覆盖类与质量类的通用问题，再攻关垂类场景，用早期成功换取组织信任与数据积累。
6. **明确人机分工边界并把它写下来。** 高频工作可自动化，关键决策不可替代；把每一阶段的"谁做决定"固化成规范，而不是靠默契。
7. **把知识资产当成有生命周期的软件来运营。** 170+ 私域模板的维护税是真实成本，需配套更新机制与过期策略，否则知识会随时间腐化。

## 相关实体

- [[concepts/harness-engineering-framework|Harness Engineering]] — 本案例是"把工作流分阶段审查结构显式化"的典型样本
- [[concepts/rag-retrieval-augmented-generation|RAG 检索增强生成]] — V3 用历史缺陷 RAG 召回把覆盖率从 12% 推到 76%
- [[concepts/multi-agent-systems|多 Agent 系统]] — V2 将 QA 流程克隆为三段 Agent 协作
- [[concepts/agent-self-improvement-loops|Agent 自进化闭环]] — V4 的 BadCase 规则库 + Review-Critique 飞轮
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测框架]] — 双级评审（模块级/用例级）是一种领域评测设计
- [[entities/nl2test-agent-natural-language-test-case-generation-bytedance-2026|字节 NL2Test 自然语言测试用例生成]] — 同为 NL→测试用例生成，可横向对比
- [[entities/agent-self-planning-ui-testing-capability-system-aliexpress-2026|阿里 Agent 自主规划 UI 测试能力体系]] — UI 测试 Agent 化的平行实践
- [[entities/pagepilot-pc-ai-test-skill-design-practice|PagePilot PC AI 测试 Skill 设计]] — 测试 Skill 化的工程经验
- [[entities/rca-agent-kuaishou-guo-yongliang-qcon-2026|快手 RCA Agent（郭永良 QCon）]] — 同一公司的 Agent 工程实践
- [[entities/mobileforge-annotation-free-gui-agent-kuaishou-zju-2026|快手 MobileForge GUI Agent]] — 快手在 GUI Agent 方向的研究
- [[entities/agent-self-evolution-evaluator-bottleneck|自进化的评测瓶颈]] — 与 V4 自进化闭环的瓶颈讨论互补
- [[entities/agent-eval-counterintuitive-insights-langfuse|Agent 评测反直觉洞察]] — 评测指标定义层面的补充

→ [[raw/articles/智能ui用例生成与执行的四阶进化实践|原文存档]]
