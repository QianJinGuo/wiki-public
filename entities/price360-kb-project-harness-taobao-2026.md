---
title: "Price360-KB：AI 驱动研发体系的项目 Harness 实践（大淘宝技术）"
created: 2026-09-02
updated: 2026-09-07
type: entity
tags: [project-harness, ai-driven-rdd, llm-wiki, context-engineering, knowledge-management, karpathy, fde, taobao, price360]
sources: [raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026]
confidence: 0.9
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Price360-KB：AI 驱动研发体系的项目 Harness 实践

## 核心命题

大淘宝技术（营销&交易技术团队，默达）受 Karpathy LLM Wiki 启发，4 个月实践将团队从 AI 辅助编程推向 AI 驱动研发体系。核心洞察：**业务 AI 研发的真正瓶颈是上下文而非模型能力**——同一个问题，换一次会话、换一个人或换一个 Agent，结果可能完全不同。^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

## 三层公式

演进路径分三步：AI 辅助编程（人在提示词框指导）→ Coding Agent（`Agent = Model + Harness`）→ 项目 Harness（`业务研发Agent = Model + Coding Agent通用Harness + 项目Harness`）。Price360-KB 是第三层的具体实现——把业务知识、源代码、项目规则、流程 Skill、质量门禁和验证证据放在同一工作空间，通过 CLI/MCP 连接动态事实。^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

## 本地优先原则 🌟

尽可能将项目需要的所有信息放进同一个文件夹，用 Git 管理。Coding Agent 天然操作文件系统，Git 补上版本/分支/评审/来源/回滚。稳定上下文进入 Git，动态事实（Aone/测试环境/日志/数据库/配置中心）留在权威系统通过 MCP 或 CLI 提供。不需要建设复杂平台和操作界面，所有操作通过 Coding Agent 打开文件夹对话进行。^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

## 迭代即知识飞轮 🌟

每个迭代阶段都在为下一个阶段生产上下文：PRD 补充业务定义和验收标准→技术方案记录系统链路→开发和测试继续校验→归档时确认后的长期知识回流到 wiki 和 tech。所有改动通过 Git 管理，完整迭代保证代码和知识库一致性。项目开始时不需要完美知识库——知识建设可以成为真实产品迭代的副产物，再逐步变成后续迭代的加速器。^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

## 知识库保存什么：wiki / tech 分离

- **wiki**：面向产品/运营/测试和 Agent，保存业务概念、规则、口径、角色、流程和异常处理。可来自已有产品材料或从代码中挖掘，产品在 PRD 阶段补充历史逻辑，开发测试阶段校验，迭代完成后归档。
- **tech**：保存代码无法独立表达但影响 Agent 判断的技术上下文——跨系统上下游链路、接口和数据关系、新旧链路切换、废弃状态、运行态拓扑和观测方式。具体方法逻辑以代码为准，只保留链路位置说明。

**知识库不应该复制代码。它保存的是代码之外会影响业务和技术判断的信息。**^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

## 项目 Harness 目录结构

Price360-KB 不是独立研发平台，而是可被 Coding Agent 直接打开和执行的 Git 工作空间。核心结构：^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

```
price360-kb/
├── AGENTS.md          # 项目协作协议与总入口
├── iterations/        # PRD、方案、测试与归档
├── wiki/              # 业务知识
├── tech/              # 跨系统链路与关键技术上下文
├── raw/               # 未经改写的原始事实材料
├── src/               # 关联业务代码仓库（Git submodule）
├── olap/              # 指标、表结构与分析 SQL
├── briefs/            # 基于事实源生成的专题说明
├── outputs/           # 可重建的索引和分析结果
└── .agents/
    ├── skills/        # PRD、方案、测试、排障、归档等领域能力
    ├── scripts/       # 工作区校验、同步和发布门禁
    ├── repositories.json
    └── skill-dependencies.json
```

## 三层结构设计

项目 Harness 分三层，职责分离：^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

1. **AGENTS.md**：项目级协作协议——事实源、研发阶段、人工决策点、授权边界、完成条件
2. **.agents/skills**：可组合的领域能力——PRD/技术方案/测试设计/回放/排障/知识归档等任务
3. **.agents/scripts**：确定性执行动作——工作区检查、产物校验、状态同步、提交推送、发布后回查

规则说明"应该怎样做"，Skill 组织一类任务，脚本把不能靠语言自觉保证的门禁执行到底。模型或 Coding Agent 产品可替换，项目协作协议仍然保留。

## 知识文件 Metadata 与关联

知识文件由正文、Metadata、链接、事实来源和 Git 版本构成。title/category/tags/status 让 Agent 读取全文前判断适用性；source 字段说明知识来自哪份原始材料。tech 文档通过 wiki_ref 关联业务知识，形成**业务规则→技术链路→源代码**的双向检索路径。^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

## 机器可读迭代协议

每个需求有独立迭代目录（prd.md/solution.md/test/design.md/test/cases*.md/test/report.md/archive.md）。REQ/AC 描述需求与验收标准，TC 建立覆盖关系。阶段不自动越过——PRD 未确认不写方案，方案和测试未确认不进代码，没有真实测试证据不进发布。上游变化时下游必须更新。^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

## 人决策，Agent 执行

Agent 检索上下文、生成产物、执行检查、调用工具、收集证据；人描述需求、选择方案、确认测试设计、验收预发结果、授权合并和发布。高风险动作不因概括性同意自动执行。Harness 演进方式：把每次真实交付暴露的问题变成下一次可复用的规则、检查或工具。^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

## 结语：Harness 收缩与 FDE 角色

Model 和通用 Coding Agent 的 Harness 能力在扩大，项目 Harness 会收缩。但不会消失的四类东西：业务知识及其事实治理、项目规则与决策边界、领域工具和系统适配、验证标准和质量责任。技术人员角色将更接近 **FDE（Forward Deployed Engineer）**——深入业务现场，把问题、知识、系统和交付结果连接起来。^[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026.md]

→ [[raw/articles/price360-kb-project-harness-ai-driven-rdd-taobao-2026|原文存档]]
