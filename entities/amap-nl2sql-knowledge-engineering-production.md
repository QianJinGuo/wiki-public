---
title: "高德 NL2SQL 知识工程：确定性路由 + 6 节知识卡片 + 352 表生产实践"
created: 2026-07-22
updated: 2026-10-15
type: entity
tags: [nl2sql, knowledge-engineering, amap, alibaba, data-warehouse, deterministic-routing, agent-skill, qoderwork, semantic-layer, smq]
confidence: 0.85
provenance_state: extracted
sources: [raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22, raw/articles/semantic-layer-smq-data-agent-sql-compiler-datafun-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 高德 NL2SQL 知识工程：确定性路由 + 6 节知识卡片 + 352 表生产实践

> 高德（Amap）基于 QoderWork Agent Skill 构建的生产级 NL2SQL 智能取数系统。覆盖 9 个业务域、352 张 ODPS 表、30,000 行结构化知识文档。NL2SQL 准确率 >95%，取数效率提升 30 倍以上。^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

## 架构演进：V1 → V2

### V1：分域独立 Skill（瓶颈暴露）

按数仓业务域拆分为 9 个独立 QoderWork Skill（8 取数 + 1 工具）。快速验证了 LLM 可行性，但暴露出 5 个结构性问题：用户选择成本高、路由天花板低、表扩充加剧域冲突、维护成本指数膨胀、扩展不便。^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

根因：路由决策发生在控制范围之外，无法调试也无法增强。^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

### V2：统一 Skill + 确定性两级路由

核心决策：不是合并 9 个 SKILL.md，而是在一个统一 Skill 内部建立 4 层分层架构。^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

| 层 | 载体 | 职责 |
|----|------|------|
| L1 域路由层 | SKILL.md 内置 | 从用户问题提取意图，查确定性域路由表（~200 行），歧义时主动澄清 |
| L2 知识加载层 | references/ 目录 | 路由后只读取该域文件；9 域 30,000 行 → 单次 200-400 行 |
| L3 标准工作流层 | SKILL.md 内置 | 6 步标准流程统一所有域：前置检查→域路由→知识加载→口径确认→选表+SQL 生成→执行+结果解读 |
| L4 公共服务层 | references/common/ | 日期计算、SQL 安全约束等公共逻辑只维护一份 |

## 关键设计决策：确定性路由而非 RAG

选择规则路由而非向量检索的三个理由：^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

1. **业务术语语义重叠**（如"收入""供给"在不同域中 embedding 几乎一样但口径不同）
2. **352 表按域分组后可枚举**（9 个域，每域 3-96 张，有限分类问题可用规则穷举）
3. **可调试可追溯**（每次路由能回答"为什么选了这个域/表"）

### 两级路由

- **一级路由**（SKILL.md ~200 行）：基于意图判断的域路由 + 易混淆术语消歧规则 + 主动澄清机制
- **二级路由**（DOMAIN.md ~100 行/域）：域内按 ADS > DWS > DWD 优先级链选表

## 知识工程方法论

### 6 节知识卡片（每张表一个 Markdown）

352 张表各有 50-130 行的 6 节标准化 Markdown 文档（"知识卡片"）：^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

1. **S1 表元数据**：全表名、数仓层级、粒度、分区、更新频率
2. **S2 字段列表**：KEY/DIM/IDX 前缀标注 + 中文释义
3. **S3 场景映射**：适用/不适用什么问题
4. **S4 关联方式**：JOIN 维度表和关联键
5. **S5 指标口径定义**：**最关键的**，计算公式和口径说明，口径来自知识库不由模型编造
6. **S6 SQL 模板**：高频查询参考 SQL

### 元数据产出 Pipeline

自动化（技术元数据拉取）→ 半自动（LLM 初版 60%）→ 人工（BI 同学注入业务口径）→ 自动校验（字段覆盖率、6 节完整性）^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

### AI-Friendly 标准分

知识质量评分体系：字段覆盖率、口径完整度、SQL 模板覆盖率、场景标注率、路由规则完备度。驱动评测→定位→补强→再评测的迭代循环。^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

## 质量保障体系

- **900+ 测试题自动化回归**：错误归因分类（选表/语法/口径/语义）
- **DDL 一致性监控**（cici_monitoring）：检测表结构与知识文档的漂移
- **用户反馈闭环**：纠正信号 + 重复提问 + 放弃对话三个沉默信号自动检测，问题解决率 >70%

## 设计原则

五条核心原则：^[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22.md]

1. **确定性优于模糊性** — 路由/选表/关键约束走规则而非概率
2. **按需加载优于全量灌入** — 200 行路由表换 200-400 行按需加载
3. **知识分层对齐变更频率** — 静态层/规则层/公共层
4. **标准化优于自由发挥** — 统一 6 节文档、6 步工作流
5. **评测驱动优于经验判断** — 标准分 + 回归测试

## 落地成效

- NL2SQL 准确率 >95%
- 智能问数场景覆盖率 >90%
- 手动取数（~8h）→ 分钟级，效率提升 30 倍以上
- 同一架构可低成本复制到不同业务方向

## 与已有实体的关系

- [[entities/alibaba-data-rd-harness-engineering-nl2sql]] — 阿里数据研发 NL2SQL 的宏观 Harness Engineering 方法论（7 Agent + NL2DSL2SQL + 幻觉防控）。本文聚焦高德具体生产系统的**架构设计**和**知识工程方法论**，角度互补。

## 补充维度：SMQ 中间表示与确定性编译路线（2026-07-31）

2026-06-30 提交的 arXiv 预印本（QUVI-3 + Gemini 3 Pro Preview）提出与知识卡片路由不同的第三条路线：在大模型与数据库之间加入受治理的 Semantic Layer，并以 **Semantic Model Query（SMQ）** 作为业务意图与物理 SQL 之间的中间表示。^[raw/articles/semantic-layer-smq-data-agent-sql-compiler-datafun-2026.md]

### SMQ 三要素与确定性编译

SMQ 由 metrics / filters / group_by 三类元素组成，表达"计算什么、筛选什么、按什么维度分组"，而不是最终物理 SQL。确定性的 SMQ-to-SQL 引擎解析元素背后的字段表达式，从预先维护的 Join Graph 注入连接条件，生成目标方言 SQL。抽象业务名称与物理实现（真实表名、字段名、计算表达式）被明确分开。^[raw/articles/semantic-layer-smq-data-agent-sql-compiler-datafun-2026.md]

### 与高德路线的对比

| 维度 | 高德知识卡片路线 | SMQ 语义层编译路线 |
|------|------------------|--------------------|
| 知识载体 | 6 节 Markdown 知识卡片 + 确定性路由 | Semantic Model（Dimension/Measure/Metric）+ Join Graph |
| SQL 生成者 | LLM 在规则约束下生成 | 确定性编译器从 SMQ 编译 |
| 模型职责 | 选域、选表、写 SQL | 生成紧凑 SMQ + 长尾 SQL 组合 |
| Schema Grounding | 知识加载进上下文 | 强制经过语义层，禁止查 INFORMATION_SCHEMA |

两条路线的共同点是把易错环节（口径、Join、方言）从纯概率生成中剥离；差异在于高德把知识"喂给"模型让模型更准，SMQ 路线把知识"编译进"确定性系统让模型更少犯错。^[raw/articles/semantic-layer-smq-data-agent-sql-compiler-datafun-2026.md]

### 评测与局限

Spider2-snow 547 项任务完成 515 项（94.15%），但该成绩是"强推理模型 + 受约束 Agent 循环 + 手工语义层 + 确定性编译器"整体能力，且无移除 Semantic Layer 的控制实验。论文未披露 Token 消耗、轮数、延迟与语义层建设人力。SMQ 只覆盖常见分析核心，长尾结构（子查询、窗口函数、递归 CTE）仍需 Agent 完成。^[raw/articles/semantic-layer-smq-data-agent-sql-compiler-datafun-2026.md]

## 未解决问题

1. 取数 Skill 与分析 Skill 之间的跨 Skill 知识共享
2. 知识库从树形目录到图状结构的平台化管理（过时检测 + 冲突发现）
3. 端到端 Agent 评测体系（不只看 SQL 准确率）

→ [[raw/articles/amap-nl2sql-knowledge-engineering-2026-07-22|原文存档]]

---
## 深度分析

### 为什么确定性路由在此场景胜过 RAG

RAG 的隐含假设是"知识条目之间语义可分"——查询向量与目标文档的 embedding 距离能反映相关性。高德的场景恰好打破了这个假设：352 张表按业务域组织后，"收入"在广告域与业财域的口径差异是业务事实层面的差异，而非文本层面的差异，向量空间无法表达这种边界（两者的 embedding 几乎重合）。同时该问题在结构上是**有限分类问题**：9 个域、域内 3-96 张表，完全可以枚举。当一个分类问题的类别集合有限且稳定时，规则表是更优解——它不引入检索误差，且每次决策可回溯（"为什么选了这个域"有明确的一行规则可指）。这与 [[entities/rag-vs-agentic-retrieval-deep-dive-yunduojun-datastudio-2026-09-01|RAG vs Agentic Retrieval 深度对比]] 的一般性结论一致：检索的竞争力在开放语料与长尾问题，而企业数仓恰恰是封闭、可枚举、强口径约束的场景，规则天然占优。

更深一层：RAG 的失败模式是**静默的**（召回了错误域，但置信度看起来不低），而规则路由的失败模式是**显式的**（歧义时触发主动澄清）。生产系统的可运维性取决于失败模式是否可观测——这正是 V1"路由决策发生在控制范围之外"这一教训的正面表述。

### 知识卡片即 Schema：把文档工程当作数据建模

6 节知识卡片常被读作"文档规范"，但更准确的类比是**为 LLM 上下文定义的 schema**：S1/S2 是结构描述（元数据 + 字段），S3/S4 是语义约束（适用性、关联键），S5 是业务不变量（口径公式），S6 是查询先验（SQL 模板）。每一节对应生成 SQL 过程中的一个决策点，缺一节就有一个决策退化为模型猜测。50-130 行的行数上限本身也是 schema 设计的一部分——它把单表上下文压到可枚举、可校验的规模，使"字段覆盖率、6 节完整性"这类自动化校验成为可能。

这条思路与 [[entities/qoder-team-knowledge-engine-compiled-knowledge|Qoder 团队知识引擎：编译型知识]] 的"知识需要编译而非堆积"一脉相承：知识卡片的价值不在内容本身，而在它把隐性口径显式化、把自由文本结构化，从而让"知识质量"变成可测量、可回归的工程对象（AI-Friendly 标准分）。它与 Lyft 的指标语义层治理在动机上同构——[[entities/metric-semantic-layer-how-lyft-governs-and-scales-key-data-definitions|Lyft 指标语义层治理]] 同样是把"口径的唯一权威定义"从使用者侧收回到平台侧，差别只在载体：一个是 Markdown 卡片，一个是中心化语义层。

产出 Pipeline 的三段式（自动拉取技术元数据 → LLM 半自动补 60% 初版 → 人工注入业务口径）同样值得注意：它把"机器擅长的"与"只有人知道的"严格分层，S5 口径定义被刻意保留为纯人工环节——这是对"LLM 会编造口径"这一失败模式的制度性防御，而非技术修补。

### SMQ 编译路线的代价结构：与知识卡片路线互补而非替代

把 SMQ 路线与高德路线放到同一坐标系，差异的本质是**错误校正发生的位置**：

- 知识卡片路线把知识加载进上下文，让 LLM 在规则约束下写 SQL——错误发生在生成时，靠 900+ 测试题回归与错误归因事后拦截。表达力高（长尾 SQL 也能写），但每次生成都是一次独立的失败机会。
- SMQ 路线把知识编译进确定性引擎，LLM 只产出紧凑的 metrics/filters/group_by——错误空间被中间表示压缩，连接、口径、方言由编译器保证。但代价是：语义层建设本身是重人力投入（论文未披露建设成本），且表达力被 SMQ 三要素截断，子查询、窗口函数、递归 CTE 等长尾结构仍需回退到 Agent 生成。

两条路线实际是同一权衡（表达力 vs 确定性）的两个端点，且**并不互斥**：SMQ 覆盖常见分析核心 + Agent 处理长尾，恰好对应高德的"确定性路由处理 352 张可枚举表 + LLM 处理规则未覆盖的查询"。选择取决于场景参数：表规模是否可枚举、口径是否高频复用、语义层建设人力是否可承受。需要强调的是，Spider2-snow 94.15% 的成绩缺少"移除 Semantic Layer"的控制实验，"确定性编译贡献了多少"尚未被隔离证明——引用该数字时必须携带这一限定。同族的"先产中间结构再生成 SQL"思路另见 [[entities/taobao-subsidy-data-analysis-agent-nl2mdl2sql-2026-09-04|淘宝补贴数据分析 NL2MDL2SQL]]。

## 实践启示

1. **先问"问题是开放还是封闭"，再选 RAG 还是规则。** 有限分类问题（可枚举的域、表、路由）用规则穷举；只有开放语料才轮到检索。范式选错，后面所有优化都在错误的方向上。
2. **把路由决策收回自己控制的边界内。** V1 的核心教训不是"9 个 Skill 太多"，而是路由发生在无法观测、无法增强的地方。任何"由用户或上游替你做的选择"都是不可调试的黑盒。
3. **文档当 schema 设计，不当说明书写。** 固定节数、固定行数上限、每节对应一个生成决策点——知识质量才可自动校验（完整性、覆盖率），也才能进入评测→定位→补强的回归循环。
4. **口径定义必须留人工环节，且是唯一权威来源。** LLM 可以写 60% 初版，但业务口径（S5）来自知识库而非模型生成——"口径不由模型编造"是整套系统能到 95% 准确率的前提之一。
5. **中间表示是压缩错误空间的通用手法。** 无论 SMQ（语义层编译）还是 NL2MDL2SQL（中间文档），本质都是让 LLM 产出一个受约束的结构化表示，而非直接生成最终产物——错误在中间层被拦截，比在最终产物上修补便宜得多。
6. **评测先行于规模扩张。** 900+ 测试题 + 错误归因分类 + DDL 漂移监控，让 40 张表到 352 张表的扩张是可控的而非赌博式的；没有这套闭环，知识库越大漂移越快。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

