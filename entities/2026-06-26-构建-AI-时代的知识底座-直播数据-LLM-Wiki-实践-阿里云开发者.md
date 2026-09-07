---
title: "构建 AI 时代的知识底座 直播数据 LLM Wiki 实践 阿里云开发者"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-26-构建-AI-时代的知识底座-直播数据-LLM-Wiki-实践-阿里云开发者]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/2026-06-26-构建-AI-时代的知识底座-直播数据-LLM-Wiki-实践-阿里云开发者.md|原文存档]]

sha256: 582389a6564e4563ad3a33f0fc1dfec7ae32c47cfb14301b72ddda1af0e7d5c1 ^[raw/articles/2026-06-26-构建-AI-时代的知识底座-直播数据-LLM-Wiki-实践-阿里云开发者.md]

## 摘要

阿里云作者提出"LLM Wiki"——用 LLM 编译思维把散落在 DDL、任务代码、文档、接口配置、看板元数据里的领域知识，编译为结构化、有约束、可验证的知识资产。核心论点是 RAG 解决不了知识本身的问题（散落的还是散落的、矛盾的还是矛盾的），需要在检索之前加一道"编译过程"；LLM Wiki 与 RAG 的关系是"编译时 vs 运行时"互补——Wiki 提供高质量语料，RAG 提供精准召回。系统类比数据库：多级文件系统当存储引擎、图索引+树索引、Schema 当 DDL、Agent 编排当执行引擎。构建遵循"提取→生成→归类→聚合→链接→验证"六步流水线，关键工程纪律包括：代码即真相（来源冲突以每天跑在生产上的任务代码为权威）、生成与判断分离（推断字段强制留空、判断结果过机械门禁+人工门禁）、证据链可追溯（frontmatter 保留 sources 字段）。^[raw/articles/2026-06-26-构建-AI-时代的知识底座-直播数据-LLM-Wiki-实践-阿里云开发者.md]

架构上采用编排层/干活层分离：wiki-orchestrator 只做意图路由、用户确认、子 Agent 调度、结果汇报，干活层 6 个 skill（material-prep、base-gen、advanced-gen、graph-builder、health-check、query）高内聚低耦合、仅通过文件目录交互。流水线分三阶段：Phase 0 材料预处理全脚本化（LLM 仅做清单解析，验证分流到 raw/ready、pending、archive 三态）；Phase 1 基础生成"批间串行、批内并行"（每批 5 对象、批内 5 个子 Agent 实现上下文隔离防张冠李戴），高阶生成按 DAG 三路并行后串行汇聚产出域/看板/指标/维度/index/overview，图构建沉淀 graph.json（8 种节点 8 种边、只存正向边+反向按需+回填 downstream）；Phase 2 健康检查 6 项规则兜底（数量一致性、结构完整性、链接有效性、Domain 格式、YAML 语法、Graph 完整性），不通过不发布。检索栈为"意图识别（精确/关系/模糊三路由）→ 多路召回（域召回+关键词召回+图扩展）→ 重排序（脚本粗排+LLM 精排，覆盖度是硬门槛、通用性单独成维对抗重复建设）"。实际效果：数据模型迭代场景下血缘查询 30min→2min（15 倍）、SQL 生成 0.5 天→10min（72 倍）、下游表遗漏率 20%→0%，影响分析从半天缩短到小时级。^[raw/articles/2026-06-26-构建-AI-时代的知识底座-直播数据-LLM-Wiki-实践-阿里云开发者.md]

## 关键要点

- 每个 Wiki 页面是 frontmatter+正文双层结构：frontmatter 承载关系字段（type、domain、upstream、computed_by 等）供脚本直接读取，正文用固定章节模板承载语义；正文中的双链 Wikilink 引用是弱关联兜底。
- 知识来源分编译时知识（表/接口/文档/任务/看板 5 类载体固化入 Wiki）与运行时知识（物理表数据、任务日志等 Agent 工具现取不进 Wiki）；深度上每张表不止抓 DDL 还要抓产出任务代码。
- 增量编译通过扫描现状三方对账（源材料目录、已有 Wiki、sources 字段），让构建成本只与变化量相关；Lint 把健康检查扩展为每周持续巡检。
- 模糊搜索前强制做域推断（读 index.md 收敛到 2~3 个相关域），是渐进式披露原则在检索侧的落地。
- 未来规划包括：引入向量检索做混合检索、知识保鲜（源材料变更自动触发增量重编译）、建立查询准确率/召回率评测 benchmark。

## 来源

- 原文: [[raw/articles/2026-06-26-构建-AI-时代的知识底座-直播数据-LLM-Wiki-实践-阿里云开发者.md|构建 AI 时代的知识底座 直播数据 LLM Wiki 实践 阿里云开发者]]
