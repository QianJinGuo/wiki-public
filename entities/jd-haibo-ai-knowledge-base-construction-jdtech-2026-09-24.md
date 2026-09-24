---
title: "京东海博 AI 知识库能力建设：三层知识架构与 Harness 知识引擎"
type: entity
tags: [jd, haibo, knowledge-base, okf, context-engineering, ai-native, harness, skill, test-knowledge, enterprise-practice]
created: 2026-09-24
updated: 2026-09-24
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

## 关联

- 姊妹实体：[[entities/jd-haibo-ai-native-harness-dual-loop-knowledge|京东海博 AI-Native 研发工程体系（运行时侧）]]
- 知识格式：[[entities/google-okf-open-knowledge-format-v0-1-2026|Google OKF]]（本文采纳的规范）、[[entities/llm-wiki-knowledge-management|LLM Wiki]]（同判前移思路）
- → [[raw/articles/jd-haibo-ai-knowledge-base-construction-jdtech-2026-09-24|原文存档]]
