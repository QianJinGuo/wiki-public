---
title: "DataWorks Copilot 需求交付 Skill — 数据需求 24h 交付的 Spec Coding 实践"
created: 2026-07-22
updated: 2026-08-01
type: entity
tags: [agent, skill, data-engineering, spec-coding, progressive-disclosure, taobao, alibaba, harness-engineering]
source_url:
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20]
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

## 笔记

2026-07-22 入库（heuristic 评分 v=7/c=7/s=4 → v×c=49）。本文发表于 2026-07-20，来自大淘宝技术公众号。^[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20.md]


→ [[raw/articles/dataworks-copilot-skill-data-request-24h-delivery-taobao-2026-07-20|原文存档]]
