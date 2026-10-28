---
title: "京东海博 AI 知识库能力建设：三层知识架构与 Harness 知识引擎"
type: entity
tags: [jd, haibo, knowledge-base, okf, context-engineering, ai-native, harness, skill, test-knowledge, enterprise-practice]
created: 2026-09-24
updated: 2026-10-26
sources: [raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24]
confidence: 0.9
provenance_state: extracted
---

# 京东海博 AI 知识库能力建设：三层知识架构与 Harness 知识引擎

京东技术（京东零售 徐双双）2026-09-24 发布，京东海博（本地生活 SaaS 中台，780+ 商家 10 万+ 门店）AI-Native 转型的**知识供给侧**建设。姊妹篇 [[entities/jd-haibo-ai-native-harness-dual-loop-knowledge|京东海博 AI-Native 研发工程体系]]（2026-07-29）覆盖 Harness/双 Loop/技能自迭代的**运行时**体系，本文补齐"上下文该如何组织、如何供给"的知识库完整方案——同平台不同能力层，两实体互补互链。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md]

## 核心立场：瓶颈不在检索在知识生产

向量 RAG/GraphRAG/Agentic Search 三种范式重心都在"运行时检索得更准"，但实践卡点在知识源头——需要结构化、绑定关联、精确定位到 `文件:行号`、随代码更新的全生命周期管理。方案：**把"知识编译"从运行时前移到维护期**（业界同判：Karpathy LLM Wiki、Google Cloud [[entities/google-okf-open-knowledge-format-v0-1-2026|OKF]]），知识入库时就消化、重写、组织成互链 Markdown 树，运行时只剩"导航+摘录"。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md]

## 三层知识架构（自底向上生长）

**底层：项目知识 × 领域知识矩阵**。横向每应用一个 `{app}-knowledge-catalog`（flows/ 一接口一文档、chains/ 端到端链路、**views/ 反向视图**——某表/Redis-key/MQ-topic 的读写方与生产消费方、**待澄清问题.md** AI 拿不准的业务点等人工回填）；纵向一级域挂大链路二级域挂细链路。关键设计：**领域知识不复制项目细节只做引用**——项目知识变了链路只更新链接层。横向看清"单个系统怎么跑"，纵向看清"一次业务改动横扫哪几个系统"。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md]

**中层：角色化知识层**——同一底层矩阵按角色重组（测试/产品/运营），一份地基多视角复用。**上层：Skill 能力层**——generate-cross-project-deps、project-analysis-tool、qa-test-breakdown、impact（d1/d2/d3 影响面分级）、okf-knowledge-read 五技能把知识变成各岗位可调用能力。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md]

## 分端分角色的知识生成（历史代码结构化）

- **后端**（入口/链路/落点/影响四问）：build-knowledge-catalog（GitNexus 索引）+ build-flow-chains（共享表/MQ topic/task_type 三类边聚合）+ generate-cross-project-deps。抽取规则：**只写能从代码查证的事实，查不到宁可留白绝不编造**。40w 行仓实测 **1051 篇**（flows 1001 [HTTP 584/RPC 167/MQ 140/JOB 96/OpenAPI 14] + chains 41 + ADR 3）；
- **前端**（操作/调用/权限/协作四问）：重点抽接口真实入参（透传参数须回调用点反推）与主子应用通信。实测 **530 篇**（API 139/组件 249/页面 101/架构 22）；
- **域层**（不读代码，读 catalog 换视角派生）：**研发库一次生成 → 域层零成本派生**业务/测试文档。聚合配送域实测 **93 篇**。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md]

## Harness 知识引擎：消费+沉淀双向闭环

消费侧：研发全流程（定档→粗评→沟通→评审→规约→TRD→编码）每步读知识库定方向+按需索引代码补细节；`/okf-knowledge-read` 六步 Pipeline（识别意图→发现知识库→匹配概念→加载文档→追踪链接→按 7 种 Type 分组注入）。沉淀侧：master 代码+过程产物经 `/distill-catalog` 蒸馏三类知识（业务/项目知识、项目开发经验、人维度经验），**蒸馏后人工确认再合并 master 知识库分支，AI 不擅自覆盖**。生命周期四步防腐：建档（绑定负责人+京 ME 通道）→蒸馏反哺→冲突确认→过期治理（基于知识置信度续期/降级/归档）。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md]

## AB 实验与测试实践

**AB 对照**（逐字一致提示词/同一需求/同一仓，唯一区别 A 用知识库 B 只用代码）：A 组快 **44%**（542s vs 967s，8 篇知识+3 源码文件 vs 翻查 16 文件），且追到两个只读本仓难以触达的隐藏阻塞点（跨仓消费端缺自动重试、配置白名单门禁静默失败）。**测试**：用例即知识资产（/prd-diff-testcase 正向生成+/harvest-cases 反向归档，用例挂业务锚点+代码锚点），git diff 沿知识库四级反查（直接命中必跑/接口命中应跑/数据面命中应跑/链路扩散建议跑）；4 业务组实测采纳率 90—100%、覆盖率 90—100%——测试同学从"从零写用例"转向"审校与补全"。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md]

## 与姊妹实体的关系（Sister-Capability）

| 维度 | 前作（运行时体系） | 本文（知识供给侧） |
|---|---|---|
| 核心问题 | AI 执行怎么保证质量 | AI 上下文怎么供给 |
| 主体 | Harness 流水线/双 Loop TDD/编审分离/技能自迭代 | 三层知识架构/知识生成技能族/蒸馏闭环/精准测试 |
| 闭环 | 开发期 TDD+部署后自测 | 消费-沉淀双向知识闭环 |

两文共同构成海博 AI-Native 全景：前作的 17 技能与规则层**消费**本文的知识库，本文的蒸馏反哺依赖前作的 harness 过程产物。知识库是两者交汇点。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md, raw/articles/jd-haibo-ai-native-harness-dual-loop-knowledge.md]

## 深度分析

### 三层不是抽象梯度，而是知识的来源与责任分配

把三层读成"由细到粗的抽象栈"是最省事也最失真的读法。按实现细节倒推，三层真正的区分变量是**这条知识从哪来、谁替它负责**：底层由代码扫描产出并附人工留白（`待澄清问题.md` 是显式的"未知"占位），中层不新增任何事实、只按读者重组，上层把知识固化成有输入输出的动作。所以领域知识"只做引用不复制项目细节"不是分层美学，而是所有权约束——复制一条事实的代价不是存储，而是同一条事实出现两个所有者，此后每次代码变更都要同步两处，漏一处知识库就从"可信任"退化为"需复核"，检索收益瞬间归零。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:32-35]

费用结构也印证这一点。后端 1051 篇、前端 530 篇、域层 93 篇——域层是从既有 catalog 换视角派生而非再读一遍代码，说明整条链路里唯一有真实边际成本的动作是"读代码并判断"，其余都是重排。这也解释了抽取规则为什么写成"只写能从代码查证的事实，查不到宁可留白绝不编造"：这条规则改变的不是准确率，而是**失败模式的可观测性**。留白可以统计、可以派活、可以进待澄清清单；编造只能等到下游出错才暴露，而那时已无法归因到具体某一条知识。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:43-48]

### 消费+沉淀双闭环：知识衰减的成本模型

知识不是静态资产，是有半衰期的库存。消费侧（六步 Pipeline：识别意图→发现库→匹配概念→加载→追踪链接→按 Type 注入）只压缩读取成本，不改变半衰期；真正决定衰减速度的是沉淀侧。因此生命周期四步（建档→蒸馏反哺→冲突确认→过期治理）应当被读成一套**给衰减定价的机制**：续期＝重新验证，降级＝降低注入权重，归档＝退出上下文。若把四步理解成行政流程，就会漏掉它们的实质——它们是唯一让"过期"这件事在系统里可表示、可查询、可负责的部分。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:52-56]

"蒸馏后人工确认再合并 master，AI 不擅自覆盖"是最容易被当作保守而跳过的一句，但它其实是整套体系的成本上限所在。蒸馏把成本从"写"转移到"审"：单条成本下降，总量却随需求数线性增长，于是瓶颈必然从生产端漂移到确认端。文章在总结里把"提升 contracts/chains 置信度以减少人工蒸馏成本"列为下一步，等于承认审校就是当前的边际成本。据此可推一个可操作的判断：知识库规模扩张时，先扩确认带宽（评审排期、置信度自动化），再扩生成能力；反过来做只会堆积待确认条目，让"知识库有 1051 篇"变成一种库存幻觉。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:54-56]

### 从知识库到知识产品：四个缺一不可的部件

面向 Agent 的知识库若只满足"文件存在且能被 grep 到"，它仍然是一堆文档。成为**产品**至少需要四件：可检索（摘要在前、逐层下钻，按需付费加载）、可版本化（index.md 渐进式披露 + log.md 追溯 + master 分支作为唯一事实源）、可消解冲突（负责人拍板，而非取最新写入）、可检测过期（置信度驱动的续期/降级/归档）。四件里最容易缺的是后两件，因为它们不产生演示效果；但它们缺失时，知识的错误会以"AI 说得不对"这种不可定位的形式出现，团队无法把它变成待修项。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:28-28]

AB 实验里那句"每条结论标注来源（业务链/行号）可回溯可审计"值得单列：它把知识从"背景材料"变成"可归因的输入"。有了归因，一次错误回答能反查到是链路缺失、视图过期还是 AI 未读到；没有归因，任何质量讨论都只能停在整体体感。这也是它与一般 RAG 选型讨论的分水岭——多数企业停在"检索得更准"，参见 [[entities/rag-vs-llm-wiki-enterprise-knowledge-base|RAG vs LLM Wiki 对比]] 与 [[entities/knowledge-base-construction|知识库构建方法论]]；本文的增量在于追问"谁负责一条知识、它什么时候作废"。同类的组织级知识记忆尝试可见 [[entities/qoder-team-knowledge-engine-compiled-knowledge|Qoder 团队知识引擎]]、[[entities/codewiki-llm-code-knowledge-base-gaode-2026-07-22|CodeWiki 代码知识库]]。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:61-61]

### AB 只能证明"有知识更好"，不能充当唯一质量闸门

AB 对照的证据很硬：同一提示词、同一需求、同一仓，唯一变量是有无知识库，A 组 542s 对 B 组 967s（8 篇知识+3 个源码文件 对 16 个文件），并追到两个只读本仓难以触达的阻塞点。但它能回答的问题只有一个——"在**这类**任务上，有知识是否更快更全"。三类问题它测不到：一是覆盖率偏差，样本天然偏向"知识库已覆盖的域"，冷门域、长尾问题、以及"读代码本来就不慢"的小改动被系统性排除；二是知识错误的代价，若某条知识已经过期，A 组不会变慢，只会更自信地错，而 AB 的指标（时长、token、找全率）对"自信地错"不敏感；三是维护成本，B 组没有维护负担，A 组的 1051 篇要持续付利息，这部分不计入对照。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:60-63]

所以 AB 是必要证据而非质量闸门。真正的闸门是覆盖率抽样、过期率、冲突未决数、待澄清问题积压这类"负向指标"，它们不产生漂亮的提速数字，却决定知识库在第 6 个月还有没有人敢信。测试侧的四级反查（直接命中/接口命中/数据面命中/链路扩散）其实提供了一个现成的覆盖率探针：一次 git diff 若四级都空，要么改动确实无影响，要么知识覆盖有洞，这个二义性本身就是待查信号。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:69-70]

### 组织前提：角色层跟着组织架构走，而不是跟着岗位说明书走

角色化知识层（测试/产品/运营）是整个架构里最便宜也最脆的一层：它不新增事实，只做重组，所以建设成本近乎为零；但它假设"角色稳定且有人在位"。团队重组、域拆分合并、负责人离职时，顺序总是**所有者先失效、知识随后失效**，而宽容消费（字段缺失自动降级、链接失效不报错）会把这一过程变成静默腐烂——没有任何报警，只有 AI 的回答悄悄变浅。据此，"知识找人"不该只被读成检索体验的升级（人找知识→知识主动找人），它实质是维护责任随组织变动重新分配的机制；缺了它，知识库的折旧曲线由组织调整频率决定，而不是由代码变更频率决定。对一个 780+ 商家、10 万+ 门店、几十子仓的中台，组织变动比代码变动更快，这是比技术选型更硬的约束。^[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24.md:72-74]

## 实践启示

1. **先测"读代码"的边际成本，再决定是否投资知识库**。1051 篇/40w 行的量级给出了参考：若单个需求的代码翻查成本低于维护对应知识的摊销，知识库在这个域就是负收益，应显式排除而不是全仓铺开。
2. **把"不知道"做成一级公民**。`待澄清问题.md` 式的显式留白比任何准确率指标都重要——它让知识缺口可统计、可派活；而编造只会把缺口转成无法归因的下游故障。
3. **按库存管理，而不是按文档管理**。每条知识建档即绑负责人和复核期，过期治理要有降级与归档两个中间态；只有"有效/无效"二元状态的知识库，实际上无法处理部分失效。
4. **预算配到审校端，不配到生成端**。蒸馏的真实瓶颈是人工确认；生成能力扩张若不同步扩确认带宽，只会积累待确认条目，制造"篇数很多"的库存幻觉。
5. **AB 用来定性，负向指标用来守门**。把覆盖率抽样、过期率、未决冲突数、待澄清积压作为常规看板；AB 的提速百分比只在一个知识覆盖充分的域内才有解释力。
6. **角色层随组织架构重排，且要有人负责重排**。每次团队调整都应触发一次知识所有权迁移；否则知识不会随组织一起搬家，只会随人一起消失。

## 关联

- 姊妹实体：[[entities/jd-haibo-ai-native-harness-dual-loop-knowledge|京东海博 AI-Native 研发工程体系（运行时侧）]]
- 知识格式：[[entities/google-okf-open-knowledge-format-v0-1-2026|Google OKF]]（本文采纳的规范）、[[entities/llm-wiki-knowledge-management|LLM Wiki]]（同判前移思路）
- → [[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24|原文存档]]

## 同平台姊妹篇（数据生产侧，2026-09-28）

京东技术同号第 3 能力层：《数据涅槃：Agentic 重塑搜推数据生产体系》——Forge Agent + Sindri + ADF 受控数据研发链路（领域 DSL/两阶段提交门禁/质量血缘贯穿/先证据后自动化），与本文知识供给侧、运行时侧实体构成同平台三层能力互链：→ [[entities/jd-forge-agent-sindri-search-rec-data-platform-2026]] ^[raw/articles/jd-forge-agent-sindri-search-rec-data-platform-2026.md]
