---
title: "图智能体：图在 Agent 中的三类功能角色"
created: 2026-10-22
updated: 2026-10-23
type: entity
tags: [agent, graph, knowledge-graph, multi-agent, memory, reasoning, survey, harness]
sources: [raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026]
confidence: 0.7
provenance_state: extracted
---

# 图智能体：图在 Agent 中的三类功能角色

> 国防科技大学赵翔团队与天津大学王鑫团队在《软件学报》发表综述《图智能体概念与研究进展》，把「图」在智能体系统中的角色拆成环境建模 / 依赖建模 / 思维建模三类功能角色，并给出横向比较与评测维度。核心判断：图的嵌入深度不等于任务表现，图的价值取决于它是否对信息获取、行为生成或状态维护产生**结构性约束**。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

## 核心定义：什么叫"图智能体"

综述给出的判定标准不是"系统里用到了图"，而是**至少有一个图的拓扑关系对信息获取、行为生成或状态维护中的某一环节产生直接的结构性约束**。这条标准把「用图做可视化」和「用图做控制」分开：前者图是产物，后者图是约束源。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

动机来自纯文本上下文的三个失效模式：多阶段任务中实体关系与步骤依赖会丢失、工具调用的顺序约束容易被违反、过时状态会被沿用作有效状态。图之所以对症，是因为节点可以承载知识实体 / 工具 / 子任务 / 记忆单元，边可以承载语义关联 / 数据流向 / 执行依赖 / 时间关系，从而让"关系"变成可查询、可更新的对象，而不再只是被写进 prompt 的一段描述。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

## 三类功能角色

| 角色 | 建模对象 | 图约束的是 | 代表方法 | 落地价值 |
|------|---------|-----------|---------|---------|
| 环境建模 | 外部知识、工具关系、环境状态 | 证据从哪里来、哪些动作可行 | RAPTOR、GraphRAG、HippoRAG、ToG、ToolFlow、ControlLLM、GraphAD、Graphormer | 检索证据组织、工具链合法性检查、场景约束参与规划 |
| 依赖建模 | 子任务依赖、多智能体协作拓扑 | 什么必须先做、谁该和谁通信 | CommFormer、AgentPrune、G-Designer、DyLAN、DynTaskMAS、DART-LLM、ChatSOP | 任务图调度、协作拓扑随任务自适应、局部重规划 |
| 思维建模 | 候选假设、中间结论、交互经验 | 推理如何展开与回溯、经验如何复用 | ToT、GoT、EGoT、SSRL、H-MEM、A-MEM、AriGraph、G-Memory、MAGMA | 推理图搜索/聚合、长期记忆的写入与关系更新 |

同一系统可以同时包含多个图实例，分别承担不同角色——这是综述强调的分析单位：**以图实例为单位，而不是以系统为单位**。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

## 环境建模：把外部世界变成可查的拓扑

知识侧有两条路线。一条从原始文本自下而上构建结构：RAPTOR 用递归聚类 + 摘要建立层次语义树，让检索可以在不同粒度间切换；GraphRAG 抽取实体与关系后做社区划分与摘要，把跨文档信息整合**前置到索引阶段**，在线查询只消费已形成的语义结构；GRAG、FG-RAG、KG-Retriever 进一步用子图检索、实体扩展、多层图索引组织多跳证据。另一条依托既有知识图谱：HippoRAG 通过图上关联传播激活跨文档证据，ToG 系列用关系选择 + 实体扩展 + 候选筛选控制搜索路径。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

行动环境侧，ToolFlow / ToolNet 把工具及调用关系表示成图，用输入输出关系约束工具链规划；ControlLLM 把工具节点与资源节点组织成有向二分图，用资源类型匹配限定可行工具链——这实质上是把"接口合法性"变成一次图可达性检查。具身/场景任务里，GraphAD 用场景图表达交通参与者与道路元素，Graphormer 把风险关系纳入规划，SceneCraft 把场景关系转成可计算的空间约束。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

## 依赖建模：任务图 + 协作图

多智能体侧的图优化分两支：CommFormer 把通信拓扑纳入策略优化，AgentPrune 用连接剪枝砍冗余交互，G-Designer 生成与任务匹配的稀疏通信图；ARG-DESIGNER / DyLAN 依据任务语义或执行表现动态配置角色与连接。任务侧把复杂目标写成带依赖边的任务图：DynTaskMAS 把自然语言任务分解成带依赖的任务图，再用任务结构 + 资源条件配置协作关系，执行反馈触发局部重构；DART-LLM 让 LLM 抽取子任务与依赖，再据此建有向无环图并用图遍历规则做分配；ChatSOP 把标准操作流程写成显式约束图。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

这套分工很清晰，也值得直接借用到 harness 设计：**语言模型负责理解任务并产出结构化表示，图搜索与依赖调度负责检查执行条件、选择可行节点、组织步骤衔接**。把"顺序正确性"从模型的自觉性转移到图的可达性/依赖满足上，是这类方案相对纯 prompt 编排的主要收益。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

## 思维建模：显式推理图与记忆图

显式路线把中间思考当可操作节点：ToT 把思考组织成树形状态空间以支持搜索与回退，GoT 升级为有向图允许思路聚合与精炼，EGoT 把生成、评估、问题提取合并进节点处理，并按节点质量与冲突信息调整后续扩展（采样温度用来调节探索范围）。隐式路线不维护完整思维图，而把拓扑约束嵌进训练目标或推理流程：ToG 让逐步推理沿知识图谱可达路径走，SSRL 用可达性信息训练策略网络。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

记忆侧：H-MEM 用层次结构组织不同粒度经验，A-MEM 按语义关联建立/更新记忆链接，TSM 引入时间关系，AriGraph 结合语义记忆与情景记忆，AgentNet 把图结构与策略学习结合、用子图选择组织执行路径并按反馈更新连接权重；跨主体共享时 G-Memory 用分层图记录交互、任务状态与策略，MAGMA 用快慢双路径（快路径把新观测挂到相关节点，慢路径对局部子图做关系推断与筛选）维持记忆结构的持续演化。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

## 横向比较：图越深不等于越好

综述从**作用位置、结构更新代价、运行开销、可解释性、鲁棒性**五个维度做横向比较，并给出一条反直觉结论：**图嵌入程度更深，并不必然带来更好的任务表现**，实际效果取决于图质量、更新机制与任务需求三者是否匹配。这对工程取舍的含义是：图不是免费的结构，它把"模型猜错"换成"图建错/图过期"，成本从推理时转移到了索引与维护时。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

评测也按角色分开：环境建模评知识检索质量与工具链合法性，依赖建模评依赖满足率与执行顺序正确性，思维建模评推理路径有效性与记忆复用效果。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

## 什么时候不该上图

三类角色对应三种失败信号，可反向用作"不上图"的判据：任务单步或线性、没有多跳证据需求时，图上索引只会增加维护成本；工具集合小且调用关系稳定时，硬编码顺序比工具图更便宜；上下文能装下完整状态、且不需要跨会话续接时，记忆图的更新代价收不回来。图的收益出现在**关系数量超出文本可表达范围**的那一侧。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

## 与 wiki 既有图工程内容的边界

同主题但不同切面：[[entities/graph-engineering-survey-system-intelligence-2026|Graph Engineering 系统智能综述]]（吉林大学主导，arXiv）把图用于**任务组织 / 智能体协同 / 运行时状态管理**，落点是"从个体智能到系统智能"的组织层；本页综述的落点是"图实例作为结构性约束"的三类功能角色，且额外覆盖了环境建模这一侧（检索图谱系 GraphRAG / HippoRAG / ToG）与方法级横向比较维度。两者互补，不互相替代。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

[[entities/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026|Procedural Graphs（Google/GaTech/PKU）]] 提供的是 harness 视角下"工具/技能/记忆作为图节点"的函数式定义，可与本页的思维建模（记忆图）与依赖建模（工具图）两节对读。检索侧的工程对照见 [[entities/three-rag-architectures-classic-graph-agentic|三类 RAG 架构]] 与 [[concepts/agent-memory-systematic-framework|Agent Memory 系统性框架]]。

## 深度分析

### 三类角色的失败模式与成本曲线并不同构

把三个角色并置在同一张表里，最容易产生的误读是以为它们只是「图放在哪里」的差别。真正的差别在**失败怎么发生、代价什么时候付清**。

环境图的错误是**沉默的**：索引一旦建错或过期，系统不会报错，只会稳定地检索到错误证据，错误再被后续推理吸收成看起来自洽的答案。它的成本曲线是「一次建、多次用」——索引阶段重投入，查询阶段边际成本很低。收益随查询次数增长，但退化没有人会主动告诉你。GraphRAG 把跨文档整合前置到索引阶段，收益与风险同时前置：查询越频繁越划算，而一旦语料漂移，整张索引图的半衰期就开始计时。^[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026.md]

依赖图的错误是**响亮的**：漏一条边就是死锁或跳过，虚假并行会让两个执行体同时改同一份状态。失败在几秒内可见，代价高但可定位。它的成本是**每次执行都要付**——调度器必须重新检查可达性与依赖满足，图规模小却高频重算。

思维图的成本曲线最陡：搜索类方法的开销大致随分支因子与深度组合增长，而收益随深度递减。它的失败方式也最难看——不是给不出答案，而是给出一个论证完整、前提却错掉的答案，因为被污染的分支会以「已评估节点」的身份进入后续聚合。

三种失败模式要求三套不同的可观测性：环境图要**新鲜度与覆盖率**，依赖图要**可达性与重排次数**，思维图要**分支预算与实际采纳率**。用依赖图那种「任务失败率」去监控环境图，等于指望沉默的错误自己喊出来。

### 表达力与预算的张力：图既是压缩器也是放大器

「图更结构化所以更省 token」只在检索侧成立。环境建模里，图把 O(语料) 的候选面压成 O(子图) 的证据面，是净压缩；依赖建模里，图把顺序约束从 prompt 里的自然语言条款换成一次可达性检查，也是净压缩。到了思维建模，图是**展开**：显式维护中间状态意味着每个候选都要被生成、被评估、被留在上下文里，token 消耗随探索宽度成比例上升。

同一个「图」字，两种相反的 token 经济学。这直接决定预算怎么分：在有限预算下，环境图与依赖图几乎总是优先项（它们消掉的是噪声与非法动作），思维图则是有条件的——只有当**单步判断置信度低、且验证某个候选比生成它便宜**时，花预算去搜索才回本。在验证比生成更贵的场景（跑测试、调外部 API、等人工确认），思维图搜索是很差的投资。

还有一层容易被忽略的成本位置问题：前置索引型方法把成本搬到写入时。对高频查询的服务这是好交易，对一次性分析任务则几乎收不回来。所以「要不要上图」的判据不该是「关系复不复杂」，而是 **查询次数 × 单次节省 > 建图与维护成本**。文本装得下时上图，是拿确定的前置成本换不确定的检索收益——这正是 [[concepts/context-engineering|上下文工程]] 里预算分配问题的图版本。

### 分类学没写出来的轴：图是谁建的

综述按「图约束什么」分类。但工程上更决定命运的是另一个轴——**图的作者是谁**，因为作者决定了图的半衰期，也决定了失败该归因到谁头上。

| 作者 | 典型形态 | 半衰期 | 主要风险 |
|------|---------|--------|---------|
| 工程师 | SOP 约束图、工具二分图、任务模板 | 月级，变更走发版 | 覆盖不到长尾任务，schema 僵化 |
| 模型 | DART-LLM 抽子任务、A-MEM 建链接、GraphRAG 建索引 | 会话级到天级 | 图本身不可信，错误不可审计 |
| 优化循环 | G-Designer 生成通信图、AgentNet 更新连接权重 | 训练周期 | 不可解释，回归难定位 |

三类图的运维方式必须分开：工程师建的图可以当代码管理（评审、单测、灰度）；模型建的图只能当**假设**管理——每条边要能追溯到抽取来源与时间，更新走追加而非原地改写，出错时能整段丢弃重建；优化循环产出的图最好当黑盒指标看，不要试图逐边解释。

这个轴还解释了为什么依赖建模在三类里最先落地：它的图最常由工程师写死（SOP、DAG 模板），因而最容易测试、最容易定责、也最容易灰度。环境图次之（索引可以离线评估），思维图最难——它的 ground truth 在模型内部，没有外部裁判。记忆图处在最尴尬的位置：它由模型写、由模型读，写入与读取的错误会互相放大，这也是 [[concepts/agent-memory-architecture|Agent 记忆架构]] 讨论「记什么」比「怎么存」更重要的原因。

从约束施加的位置看，三分法其实是**输入侧（环境）、编排侧（依赖）、内省侧（思维）**。越往内，验证成本越高，因为外部世界不再提供对照。这条梯度比三分法本身更有指导性：可靠性投入应该优先压在验证便宜的那一侧。

### 评测的真正缺口：没有人测图本身

综述归纳的任务完成效果、长程一致性、图更新成本、工具调用合法性、通信开销、记忆保持六个维度，全是**系统级**指标。四个空白值得单独指出：

**其一，图的质量从未被单独度量。** 边级 precision/recall、节点覆盖率、过期节点比例、更新滞后——这些在知识图谱社区是标准动作，在图智能体评测里几乎缺席。后果是所有失败都被归因到「模型不行」或「图不行」，而两者不可分。

**其二，缺少「同等信息」的扁平基线。** 把图替换成等 token 的扁平文本块或结构化 JSON，性能掉多少？没有这个对照，就无法区分收益来自**拓扑**还是仅仅来自「多给了结构化上下文」。这是最该补、也最容易补的实验：成本极低，杀伤力极大——很多「图有效」的结论很可能只是「结构化有效」的下位陈述。

**其三，成本记账不统一。** 索引 token、图维护延迟、调度器开销很少进同一张表，于是「更贵但更准」无法换算成单位成本，工程上无法决策。至少应报告「每提升一个点的任务成功率，多花多少 token 与多少毫秒」。

**其四，没有图退化的鲁棒性曲线。** 随机删 10% 边、注入 5% 错边、让 20% 节点过期，性能如何崩塌？这条曲线才决定系统能不能上线——线上图一定会坏，问题只是坏多少还能用。

四点合起来说明：当前评测在衡量「用了图的 agent 好不好」，而没有衡量「这张图值不值」。前者适合论文，后者适合上线决策。评测方法侧可对照 [[concepts/evaluation-harness-design|评测 Harness 设计]] 与 [[entities/agent-evaluation-survey-ibm-yale-2026|Agent 评测综述（IBM/Yale）]]。

## 实践启示

1. **一图一角色一指标。** 每个图实例只声明一个角色，只绑一个主 SLI：环境图看证据召回与新鲜度，依赖图看可达性与重排次数，思维图看分支采纳率与预算消耗。角色混装的图无法定责，也无法调参——这三类指标的采集点根本不在一处。

2. **先跑扁平基线，再决定上图。** 用等 token 的 JSON / 大纲方案把任务跑通，量出差距；差距落在噪声内就别建图。这一步几乎零成本，却能挡掉大部分「为了架构好看而上图」的项目。

3. **确定性留给拓扑，不确定性留给内容。** 顺序与合法性检查放进执行器代码（可达性、前置条件、资源类型匹配），不要交给 LLM 判断「下一步顺序对不对」。图的核心收益就是把这判断从模型的自觉性转移到可测试的检查上。

4. **给每张图定 TTL 与失效触发条件。** 工具集变更、schema 升级、任务分布漂移都应触发重建；图要能当缓存失效，而不是永久资产。同时保留「降级到文本」的开关，图坏掉时系统仍能跑，只是变慢变糙——这和 [[concepts/when-not-to-memory-architecture|何时不该做记忆架构]] 的判据同源。

5. **思维图必须有硬预算。** 深度、分支因子、节点总数设上界，并配剪枝准则（按节点评分与冲突信息裁剪）；否则搜索会吃掉整个 token 预算，而收益在第三层之后基本归零。预算分层的具体做法见 [[concepts/harness-context-window-management|上下文窗口管理]]。

6. **每条边可追溯，更新只追加。** 边要带来源与时间戳，新证据不覆盖旧边而是追加并标记失效。这样图出错时能定位到抽取环节、能整段回滚——这是模型建图唯一可行的可靠性手段，也是 harness 侧 [[entities/agent-harness-engineering-survey-2026|Agent Harness 工程综述]] 所说的组件化治理在图上的体现。

## 关联

- [[entities/graph-engineering-survey-system-intelligence-2026|Graph Engineering in the Era of LLM Agents（综述）]] — 系统智能组织层切面
- [[entities/graph-engineering-multi-agent-essence-xiaozhi|Graph Engineering 多智能体本质]] — 协作拓扑的工程解读
- [[entities/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026|Procedural Graphs]] — harness 视角的图定义
- [[raw/articles/graphmemix-multimodal-agent-memory-evidence-forest-zhiyanlab-xhs-2026-09|GraphMemix 原始存档]] — 记忆图的具体实现（尚无独立实体页）
- [[concepts/agent-memory-systematic-framework|Agent Memory 系统性框架]] — 记忆图的记忆层背景
- [[concepts/harness-long-running-task|长任务 Harness]] — 依赖建模的落地场景

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
- [[moc/multi-agent-coordination|Multi-Agent Coordination]]

→ [[raw/articles/graph-agents-survey-three-modeling-directions-software-journal-2026|原文存档]]
