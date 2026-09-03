---
title: "阿里数据团队 LLM Wiki 企业实践：LLM 编译思维构建结构化知识资产"
created: 2026-06-30
updated: 2026-08-28
type: concept
tags:
  - llm-wiki
  - knowledge-base
  - enterprise
  - data-warehouse
  - compiler
  - schema
  - graph
  - agent-orchestration
  - provenance
  - aws
  - bm25
  - prompt-caching
  - incremental-build
  - lint
sources:
  - raw/articles/alibaba-llm-wiki-enterprise-practice-compiler
  - raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28
---

# 阿里数据团队 LLM Wiki 企业实践：LLM 编译思维构建结构化知识资产

> 阿里数据团队提出的企业级 LLM Wiki 实践，核心主张：用 LLM 编译思维替代人工写 Wiki，把散落在 DDL、任务代码、文档中的领域知识编译为结构化、有约束、可验证的知识资产。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

## 核心主张

任何 AI 系统都由模型、知识、架构三部分组成。模型由供应商提供只能被动接受；架构常因模型能力升级而失效重做；**领域知识只能从内部积累，不可替代，是最值得长期投入的部分**。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

### RAG 的局限

直接套 RAG 解决不了知识管理问题——RAG 不改变原始材料本身的状态，散落的还是散落的，矛盾的还是矛盾的，过期的还是过期的。**问题出在知识本身，不在检索**。需要在检索之前加一道"编译过程"——把散落、矛盾、易腐化的源材料，预先加工为可被 AI 直接消费的知识。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

### LLM Wiki 与 RAG：互补而非冲突

- **Wiki = 编译时**：把原始材料预处理成高质量的知识页面（血缘显式记录、口径以代码为准做仲裁、层级结构支持渐进式披露）
- **RAG = 运行时**：在查询时刻做精准召回（frontmatter 用于硬过滤，正文用于语义匹配）
- Wiki 尚未覆盖的长尾知识仍可回退到原始材料检索 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

## 与 Karpathy LLM Wiki 的对比

| 维度 | Karpathy LLM Wiki | 阿里数据团队 LLM Wiki |
|------|------------------|---------------------|
| 场景 | 个人知识管理 | 企业数据仓库知识管理 |
| 源材料 | 文章、论文、报告 | DDL、任务代码、接口配置、看板元数据 |
| 编译方式 | Agent 单页编译 | 7-skill 编排流水线（编排层+干活层分离） |
| 关系图 | wikilink 隐式关联 | 8 种显式边类型（血缘/归属/消费/引用） |
| 校验 | lint 检查 | 6 项健康检查（结构/链接/格式/语法/图完整性） |
| 增量 | 全量重建 | 增量编译（对账+局部重跑） |
| 核心约束 | source-first 脚注 | 代码即真相 + 生成与判断分离 + Schema 即契约 |

## 关键设计决策

### 1. 代码即真相

不同来源对同一对象描述不一致时，以任务代码为权威——注释和文档可能长期失修，但任务代码每天实际跑在生产上。这条规则把所有"以谁为准"的争议收敛到唯一答案。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

### 2. 生成与判断分离

LLM 在生成基础 Wiki 时，需要推断的字段强制留空，只写有源材料直接支撑的内容。所有基础页面生成完后再独立跑判断阶段，基于已写入的内容综合输出候选，由用户确认后回填。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

### 3. Schema 即契约

每个页面是 frontmatter + 正文的双层结构。frontmatter 是 YAML 格式的结构化头部，包含共性字段和页面特有字段（如 table 的 upstream/downstream/domain/layer，metric 的 computed_by 等）。脚本可直接按字段提取，不需要解析自然语言。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

### 4. 编排层与干活层分离

编排器（wiki-orchestrator）只做四件事：意图路由、用户确认、子 Agent 调度、结果汇报。干活层 6 个 skill 各自覆盖一个阶段，通过文件系统约定的目录交互，任何 skill 都可以独立运行。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

## 多级文件系统设计

```
${KB_ROOT}/
├── pre/              # 待处理清单 + 临时下载产物
├── raw/              # 三态分流：ready / pending / archive
├── wiki/             # 编译完成的结构化页面
│   ├── tables/       # 表页面（按存储类型分目录）
│   ├── domains/      # 域页面（按父域分目录）
│   ├── concepts/
│   ├── metrics/
│   ├── dimensions/
│   ├── dashboards/
│   ├── apis/
│   ├── datasets/
│   ├── graph.json    # 全局关系图
│   ├── index.md      # 全局索引
│   └── overview.md   # 全景概览
├── log/              # 构建日志、健康检查报告
├── tmp/              # 跨 skill 协作中间状态
└── schema/           # 页面模板
```

raw/ 三态分流（ready/pending/archive）把"材料是否可用"从隐藏字段变成目录归属的物理事实。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

## 关系图设计

8 种节点类型，8 种正向边：

| 边类型 | 含义 | 方向 |
|--------|------|------|
| has_upstream | ODPS 离线血缘 | 当前表 → 上游表 |
| syncs_from | 同步任务血缘 | 当前表 → 来源表 |
| writes_from | Flink 实时血缘 | 当前表 → 来源表 |
| belongs_to | 归属关系 | 表/接口/看板/子域 → 父域 |
| computed_by | 指标/维度来源 | 指标/维度 → 表 |
| uses | 看板使用 | 看板 → 数据集 |
| queries | 接口/数据集查询 | 接口/数据集 → 表 |
| wikilink | 兜底引用 | 引用页 → 被引页 |

只存正向边，反向按需计算，回填关键反向字段到 frontmatter。以单个 JSON 文件存储，无需图数据库。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

## 编译流水线

| 阶段 | 名称 | 关键设计 |
|------|------|---------|
| Phase 0 | 材料预处理 | 全脚本化，支持断点续传，三态分流 |
| Phase 1 基础 | 基础 Wiki 生成 | 批间串行批内并行（每批 5 对象），统一校验 |
| Phase 1 高阶 | 高阶 Wiki 生成 | 一阶段三路并行，二阶段串行汇聚 |
| Phase 1 链接 | 图构建 | 脚本扫描 frontmatter，不依赖 LLM |
| Phase 2 | 健康检查 | 6 项检查，任意不通过视为编译失败 |

## 检索栈

三步检索：意图识别（精确/关系/模糊）→ 多路召回（域召回 + 关键词召回 → 图扩展）→ 重排序（粗排 score + 精排 LLM 三维度：覆盖度/相关性/通用性）。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

## 增量编译与持续 Lint

增量识别的关键是扫描现状对账（源材料目录 vs 已有 Wiki 页面 vs sources 字段）。三类场景：新材料入库、知识腐化更新、结构性修复。

Lint 把健康检查从"构建后一次性兜底"扩展为"持续性质量巡检"，每周或重大变更后定期跑一次。 ^[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler.md]

## 与本 Wiki 的关联

阿里数据团队的 LLM Wiki 实践与 [[concepts/llm-wiki-paradigm|Karpathy LLM Wiki 范式]] 互补——Karpathy 解决个人知识管理，阿里解决企业数仓知识管理。两者共享"LLM 作为编译器"的核心隐喻，但在架构复杂度、校验机制、增量编译、关系图设计上差异显著。

本 Wiki（Hermes-Wiki）的 frontmatter + sources + lint 设计与阿里方案在"Schema 即契约"和"证据链可追溯"两个原则上高度一致，但本 Wiki 采用更轻量的实现（单文件 lint 脚本 + wikilink 隐式关联），阿里方案采用更工业化的实现（7-skill 编排 + 8 种显式边 + 6 项健康检查）。

→ [[raw/articles/alibaba-llm-wiki-enterprise-practice-compiler|原文存档]]
→ [[concepts/llm-wiki-paradigm|LLM Wiki 知识范式]]
→ [[concepts/source-first-knowledge-compilation|source-first 知识编译]]

## 第 2 来源 — AWS LLM Wiki 企业级实践三道坎（2026-08-28）

> AWS China Blog 四个月企业合规知识问答实践（127 份中英文政策文档 → 204 个活跃 Wiki 页面），把 Karpathy LLM Wiki 范式推向企业生产环境时遇到的三道坎及解法，含量化验证。与阿里方案同属「LLM Wiki 企业实践」artifact family，互补角度 5 条。^[raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28.md]

**互补角度（5 条）**：

1. **并发编译正确性（38% 页面重复）**：批量导入 65 份 PDF 生成 245 页，其中 93 页（38%）语义重复——根因是 Plan 阶段并发冲突发生在语义空间而非存储层（两个任务基于同一目录快照各自判断创建 tax / tax-policy）。解法：**串行规划、并发编译**——Plan 读最新 Wiki Index 并写入 planned 占位后，Parse/Generate 仍保持并发；一次上百份文档批量导入零任务失败、目录审计零语义重复。^[raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28.md]
2. **查询完整性（一行摘要是有损压缩）**：答案已写进 Wiki 但系统找不到——目录导航只暴露一行摘要，费率/小众术语等细节被压缩掉。解法：目录导航 ∪ **BM25 全文旁路**（取并集而非改写 LLM 选页结果）；150 题评测命中率 70.0% → 92.7%（摘要盲区题 60%→92.5%，常规题 78.8%→93.8%）。^[raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28.md]
3. **人工修正存续（Pin 语义层）**：文本 diff 不可靠（LLM 重写改变措辞/段落，patch 失去锚点）；改为保存**结构化 pin**（claim = 专家确认的事实 + anchor + provenance），重编译后逐条检查——满足则存活、冲突则送人工、章节消失则标 orphan。受控实验 12 pin：8 存活 + 4 冲突显式上报 + 0 静默丢失。^[raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28.md]
4. **Prompt Caching 控制全目录输入成本**：204 页目录约 2.8 万 token 是稳定的 prompt 前缀，Bedrock Prompt Caching 在目录与问题间设缓存点——30 次查询命中率 100%、选页成本下降约 90%、时延 3.0s→2.3s（前提：目录构建保持确定性，避免动态字段）。^[raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28.md]
5. **RAG vs LLM Wiki 执行模式分野**：LLM Wiki 不是 RAG 替代品——RAG 适合「广而新」知识（海量、高频更新、无需预编译），LLM Wiki 适合「少而精、准而深」知识（强权威、强一致、可追溯）；实践采用 Wiki-first、RAG fallback。^[raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28.md]

### 与阿里方案的对照

| 维度 | 阿里数据团队方案 | AWS 企业实践 |
|------|------------------|--------------|
| 源材料 | DDL/任务代码/接口配置 | 政策/法规/操作指引 PDF |
| 编译流水线 | 7-skill 编排（编排层+干活层分离） | Ingest → Compile → Serve（S3/EventBridge/SQS/Step Functions） |
| 并发 | 批内并行、批间串行 | **Plan 串行 + Generate 并发**（并发边界按状态依赖划分） |
| 检索 | 意图识别→多路召回→重排序 | 目录导航 ∪ BM25 全文旁路 |
| 人工审核 | 生成与判断分离（回填） | 审核门在 Generate 之后（审核产物而非计划）+ Pin 语义层 |
| 增量 | 对账+局部重跑 | 页面发布后增量维护 BM25 索引 |

两方案共享「LLM 作为编译器」「编译时 vs 运行时」「知识质量从源头治理」核心隐喻，但 AWS 实践给出了企业规模下更细的并发/检索/人工修正量化数据。^[raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28.md]

→ [[raw/articles/llm-wiki-enterprise-practice-three-hurdles-aws-2026-08-28|原文存档]]
→ [[concepts/llm-wiki-paradigm|LLM Wiki 知识范式]]
→ [[concepts/source-first-knowledge-compilation|source-first 知识编译]]

## 所属 MOC

- [[moc/wiki-pending-concepts-roadmap|Wiki Pending Concepts Roadmap]]
