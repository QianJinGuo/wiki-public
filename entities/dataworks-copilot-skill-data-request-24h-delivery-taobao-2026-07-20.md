---
title: "DataWorks Copilot 需求交付 Skill — 数据需求 24h 交付的 Spec Coding 实践"
created: 2026-07-22
updated: 2026-10-24
type: entity
tags: [agent, skill, data-engineering, spec-coding, progressive-disclosure, taobao, alibaba, harness-engineering]
source_url:
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# DataWorks Copilot 需求交付 Skill — 数据需求 24h 交付的 Spec Coding 实践

淘宝直播数据团队基于 DataWorks Copilot 构建了一个面向数据需求交付的 Agent Skill（`copilot_req2sql`），通过 4 阶段 Spec Coding 流程（需求澄清 → 资源管理 → 模型设计 → 交互物产出）将数据需求交付从"串行排期"转变为"Agent 自动执行、人按优先级介入决策的并行推进"。^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]

## 背景：AI 生码的五大痛点

在淘宝直播数据团队内，DataWorks Copilot 等生码工具虽然能处理简单取数需求（口径清晰、来源表明确时约 10 分钟产出可用代码），但在真实工作场景中面临五个核心痛点：^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]

1. **写 Prompt 本身就很累**：用户需要同时装下业务口径 + 表结构 + 团队代码规范 + Copilot 偏好格式，一次复杂需求的 prompt 准备工作就要半小时起步
2. **复杂业务逻辑生码质量差**：LLM 对复杂嵌套逻辑和隐含业务约束理解有限，整体 SQL 常因某个语义翻译错误而完全不可用
3. **多轮对话的"熵增定律"**：上下文越来越长导致模型思考和效果衰减，开新对话又需重组全部上下文
4. **需求评审后没有明确的产物**：缺乏双方认可的结构化需求方案做锚点，口径理解不一致直到验收时才暴露
5. **多需求只能串行排期**：前置准备工作（需求澄清、口径确认、表结构梳理）耗时长且依赖人的连续注意力

这些痛点的共同根因是：从"业务需求"到"高质量 Prompt"之间，缺少标准化、可复用、不依赖个人经验的自动化工具。^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]


## copilot_req2sql：4 阶段 Spec Coding 解决方案

`copilot_req2sql` 是一个数据需求交付 Agent Skill，通过 4 阶段工作流将不可控的"黑盒代码生成"变为可审计的"分层交付"：^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]

| 阶段 | 产出物 | 核心任务 |
|------|--------|----------|
| P1 需求澄清 | `p1_requirement.md` | 拆解业务需求，澄清指标口径 |
| P2 资源管理 | `p2_resources.md` | 确定相关表、字段映射、伪代码 |
| P3 模型设计 | `p3_model_design.md` | 数据模型设计（人工 CR） |
| P4 交付物产出 | `p4_copilot_input.md` | 生成结构化的 Copilot 交互物 |

核心设计是 **Spec Coding（规约驱动编程）**：在让 AI 写代码之前，先产出经人类 review 的规格说明书（Spec），代码生成从"黑盒魔法"变为"翻译"——将经过 review 的模型设计翻译成代码。这与 [[entities/spec-driven-development-cognitive-framework|Spec-Driven Development]] 和 [[entities/sdd-spec-driven-development-summary-qoder|SDD 规约驱动编程]] 的理念一致。^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]


## Spec 目录结构

每个需求对应一个独立的 Spec 目录，兼顾分层、可追溯、可复用、可并行：^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]


```
specs/yyyymmdd_{任务名}/
├── stages/               # Agent 工作流自动生成
│   ├── p1_requirement.md
│   ├── p2_resources.md
│   ├── p3_model_design.md
│   ├── p4_copilot_input.md
│   └── workflow_events.jsonl   # 工作流事件记录
└── proposal/             # 人工整理的原始需求
    └── yyyymmdd_proposal.md
```

## 渐进式披露（Progressive Disclosure）

这是 Skill 区别于传统"写一份 prompt 模板"的关键设计：^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]

- 上一步不确认就不推进下一步
- 每一阶段产出的文档都是不可篡改的"锚点"
- 避免了 LLM 在长上下文中的"预期偏差"——不需要提前猜测用户意图

这一设计与 [[entities/打造高效易用的agent-skill|Agent Skill 设计]] 中提到的渐进式上下文披露理念吻合。^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]


## 技术栈与适用范围

- **计算引擎**：MaxCompute（ODPS）
- **数据开发平台**：阿里云 DataWorks（表结构查询、节点代码获取、数据血缘追溯、SQL 执行 API）
- **Agent 运行时**：支持 Skill 定义和自然语言交互的 Agent 框架

核心设计思路（标准化模板、基准表发现策略、降级验数策略）适用于任何有标准数据建模、元数据查询 API 和 SQL 执行能力的数仓研发平台。^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]

## 与相关实体的关系

- [[entities/spec-driven-development-cognitive-framework|Spec-Driven Development]] — Spec Coding 的理论框架基础
- [[entities/sdd-spec-driven-development-summary-qoder|SDD 规约驱动编程]] — SDD 在工程实践中的总结
- [[entities/打造高效易用的agent-skill|Agent Skill 工程实践]] — Agent Skill 的设计原则（含渐进式披露）
- [[entities/openspec-spec-driven-development-trae-solo|OpenSpec Spec-Driven Development]] — OpenSpec 的 SDD 实现
- [[entities/alibaba-devix-harness-ops-agent-7x24|阿里巴巴 Devix Harness Ops Agent]] — 阿里系 Agent 运维工程实践

## 深度分析

### SQL 生码的失败是欠规格，不是模型不够强

一个被低估的事实：口径清晰、来源表明确时，Copilot 十分钟就能产出可用代码^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md:67]——能力早已过关，卡点在"这三个前提在真实场景里恰恰不成立"。诊断规则是：若失败集中在编造表名、口径偏差、聚合粒度错，问题不是采样，是任务被**欠规格化**（underspecified）；换更强的模型治不了，因为缺的信息不在参数里，在组织的元数据里。

一次性 prompt 必然失败，是因为它把四个本可独立验证的决策（交付什么 → 数据在哪 → 怎么建模 → 怎么措辞）压进同一条 token 流，任一处错则整段报废且不可定位，只能整段重写^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md:126]。4 阶段的实质是把一个联合概率拆成四个条件概率：每阶段只在上一步已确认的产物上做增量推理，错误因此可定位、可局部修复、可在最便宜的那层被拦。收益也就落在成本结构而非准确率：口径错在 P1 被拦是一轮对话的代价，同样的错流到代码层就是"改 bug 比从头写还多"。Spec 链本质是 shift-left 流水线，见 [[concepts/skill-engineering-principles|Skill 工程原则]]。

### "翻译"而非"设计"：难度从模型侧搬到规约侧

"P3 模型设计质量 → P4 交互物质量 → Copilot 生码质量"是一条严格的因果链，也是能力重分配：设计复杂度在 P3 被人消化，Copilot 接到的任务从"设计一段逻辑"退化为"把伪代码译成规范 SQL"。任务变窄，通过率自然上升——这解释了同一模型用 Spec 喂与用 PRD 喂为何差一个量级。"不是让 AI 更聪明，而是让 AI 的工作更可被审计"不是谦辞而是机制描述：它不动分母（模型能力），只动分子（需要模型猜的量）。

### 渐进式披露管"选规则"，不管"守规则"

SKILL.md 约 200 行只放 4 阶段路由与硬规则，模板、示例、规范下沉到 `references/` 的 7 组文件按阶段加载^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md:436]，治的是三类退化：约束被样板淹没（lost in the middle）、注意力稀释后分不清"必须"与"参考"、跨任务规则互相污染。这是 [[concepts/context-window-economics|上下文经济学]] 的一次具体操作。

但它有三重边界，文章自己已撞到第二重：**选得对 ≠ 守得住**——直播数仓的命名规范与业务黑话极其复杂，"即使通过渐进式披露让 LLM 自主使用，也很难完全遵守"^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md:572]，这才是 P3 强制 CR 的真实理由；**有策略 ≠ 规则已被写下**——新表与临时口径不在披露策略的作用域内；**加载 ≠ 不过期**——规范演进后 `references/` 会腐烂，而腐烂规则比没有规则更危险，模型会自信地执行它（[[concepts/harness-component-expiry-and-build-to-delete|组件过期]]）。

还有一层递归：P4 本身就是渐进式披露的产物——Copilot 不需要 P1 澄清、P2 映射、P3 的 DDL 演变史，只要结论。整条 Spec 链因此是有损压缩管道，每级丢过程留结论，末端产物自包含。这才使"新开对话不丢上下文"成立：它把 Copilot 从有状态服务降级成了无状态服务。多轮熵增的解法不是压缩上下文，而是把状态外置到磁盘。

### 成本结构与失效面

省下来的不是 token，是**人的连续注意力**：介入点从"从头跟到尾"压到两个（P1 确认、P3 CR），单人才可能同时挂 3-5 个需求。并行度 ≈ 1 / 人工介入次数，与模型速度关系不大；24h 交付不是生成变快，而是拆掉了生成之前的排队——排队论问题，不是推理速度问题。

失效面同样清晰：低复杂度需求摊不薄四阶段的固定成本（[[concepts/when-not-to-harness-engineering|何时不该做 Harness 工程]]）；难度被人承接而非消除，不常用口径与新表仍需人跑验证 SQL；P3 CR 是单点关键路径，加 Agent 扩不了容；最危险的是错误可被完美继承——P3 错了，P4 会忠实地把错误翻译成规范 SQL，Spec 链让错误可追溯却不使其可检测，双方都同意的错口径能穿过每道闸门，只在验数时炸。

## 实践启示

1. **先做失败归因，再决定要不要上 Spec。** 抽 20 个失败案例分类：错在表名、口径、聚合粒度的是欠规格，Spec 有救；错在方言细节与函数写法的是能力问题，等模型升级。
2. **用目录当状态机，别用对话当状态机。** `specs/yyyymmdd_{task}/stages/pN_*.md` 逐阶段落盘、只增不改，`workflow_events.jsonl` 留执行轨迹；每需求一个目录，即是并行的最小单位。
3. **只设两道强制闸门。** P1 交付物口径与 P3 模型设计强制人确认，其余自动推进。闸门多了人成瓶颈，少了错误直接流到代码层。
4. **SKILL.md 是路由，不是手册。** 主文件 200 行内只放路由与硬规则，阶段模板与示例下沉 `references/`；全阶段共享的规范（命名、ETL 规则）单独一层，避免与阶段规则互相污染。
5. **能查的交给语义层，查不到的显式标人工。** 常用口径自动检索并回查表结构确认字段；新表与临时口径走人工探查、由 Agent 归档进 P2。把"模型不许猜表名"写成硬规则。
6. **KPI 用介入次数与返工率，不用生成速度。** 度量 P1/P3 的 CR 时长、验数返工率、在飞需求数；给 `references/` 建 golden spec 回归，防止规范腐烂后模型自信地执行过期规则。

## 笔记

2026-07-22 入库（heuristic 评分 v=7/c=7/s=4 → v×c=49）。本文发表于 2026-07-20，来自大淘宝技术公众号。^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]


→ [[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20|原文存档]]
