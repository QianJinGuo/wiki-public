---
title: "驾驭AI Coding：面向团队的Harness Engineering落地规范"
created: 2026-07-17
updated: 2026-09-18
type: entity
tags: [harness-engineering, tencent, ai-coding, team-specification, multi-agent, mcp, skills, knowledge-base, guardrails, evaluation, tool-system, context-management, execution-orchestration]
sources:
  - raw/articles/tencent-harness-engineering-team-specification-2026
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 驾驭AI Coding：面向团队的Harness Engineering落地规范

腾讯技术工程 atreusliu 发布的团队级 Harness Engineering 完整落地规范，从理念到实操覆盖 6 大支柱、5 层架构、3 阶段路线图和详细配置步骤，是 AI Coding 工具链在团队中系统化落地的生产级实践指南。^[raw/articles/tencent-harness-engineering-team-specification-2026.md]

## Harness 6 大支柱

| 支柱 | 核心问题 | 对应工具与团队实践 |
|------|---------|------------------|
| 上下文管理 | AI 看到什么信息？ | AGENTS.md 渐进式披露、Spec 文档进 Git、changes/ 变更隔离、Skills 按需加载、知识库挂载、AI Wiki |
| 工具系统 | AI 能触达什么？ | MCP（DB/API/知识库/运维）+ Skills（工具接入/代码生成/元技能/搜索发现）+ 知识库（iWiki/代码库/AI Wiki） |
| 执行编排 | AI 按什么顺序做？ | "3+1 Phase"（Planner→Generator→Evaluator→Archiver），SDD 工作流 |
| 状态与记忆 | AI 记住什么？ | 短期=会话、中期=Memories、长期=Git Spec、变更=Spec Deltas |
| 评估与观测 | AI 做得对不对？ | L1-L4 四层评估 + AI Code Review |
| 约束与恢复 | AI 不能做什么？ | Rules 硬性红线 + Skills 软性约束 + Safety 安全策略，Git 回滚 |

^[raw/articles/tencent-harness-engineering-team-specification-2026.md]

## AI Coding 一体化架构（5 层）

输入层 → 工作台（CodeBuddy：配置中心+模式引擎+Agent核心）→ MCP 层（DB/API/Wiki/CI/CD）→ 输出层（代码/测试/文档）→ 度量层（AI占比/交付量/Bug率）→ 反馈优化配置中心。^[raw/articles/tencent-harness-engineering-team-specification-2026.md]

## "3+1 Phase" 多 Agent 协作

标准化工作流定义了四类 Agent 角色：
- **Planner**：理解需求、拆解任务、生成方案（Plan 模式 + 项目 Spec）
- **Generator**：按方案写代码、写测试（Rules + Skills + MCP）
- **Evaluator**：代码审查、规范检查、测试验证（Rules + 验收标准）
- **Archiver**：归档变更、更新知识库（归档脚本 + Git）^[raw/articles/tencent-harness-engineering-team-specification-2026.md]

## 实施路线图（3 阶段）

1. **基础建设**（1-2 周）：CodeBuddy 安装、team-harness 仓库、基础 Rules、知识库配置
2. **工具接入**（2-4 周）：MCP 接入、Skills 沉淀、Spec 驱动开发流程跑通
3. **持续优化**（持续）：度量看板、规范迭代、知识飞轮

## 工具系统三件套

MCP 是开门的钥匙，Skills 是开门后做的事情，知识库是进门前读的说明书。三个比喻精准概括了三者的职责分工与协作关系。^[raw/articles/tencent-harness-engineering-team-specification-2026.md]

## 关键理念

- **反模式总结**：7 种常见错误（环境配在 prompt 里、跳过 Plan、不 Review 直接合入等）
- **harness-audit Skill**：自动化合规性自检，确保团队规范被机械执行
- **约束三层**：硬性红线（不可违反）→ 软性约束（推荐遵循）→ 安全策略（兜底保护）

→ [[raw/articles/tencent-harness-engineering-team-specification-2026|原文存档]]

---
## 深度分析

### 规范才是可复制的单位，而不是工具
这份文档的价值不在于介绍了 CodeBuddy、MCP 或 Skills 这些具体组件，而在于把"某个人用某个 AI 工具写代码很顺"翻译成组织可复用的资产：契约、目录结构、流程与自检脚本。工具会随模型换代被替换，规范却规定了无论换哪个模型与工作台，上下文如何分层、变更如何隔离、交付如何验收。因此它是团队级 AI Coding 真正的最小可复制单元——新人 clone 仓库后拿到与老手相同的 harness 契约，产出质量的方差被结构性收窄。^[raw/articles/tencent-harness-engineering-team-specification-2026.md:13-15]

### 六大支柱互相咬合，最先失效的是状态与记忆
六个支柱不是并列清单而是闭环依赖：工具系统决定 AI 能触达什么，[[concepts/context-management-agent-systems|上下文管理]]决定它看到什么，执行编排决定顺序，评估决定"做得对不对"，约束决定"不能做什么"，而状态与记忆把一次会话的成果沉淀为跨会话资产。真正的先失效点不是工具缺失——工具最容易补——而是状态与记忆缺失：没有 Git 里的 Spec 与 Spec Deltas，每一步都在重新解释需求，上下文管理退化为 prompt 堆料，评估也失去可比对的基线。^[raw/articles/tencent-harness-engineering-team-specification-2026.md:17-45]

### 5 层架构回答了护栏、上下文与评估该住在哪一层
输入层只负责表达意图，工作台层承载模式引擎与 Agent 核心，MCP 层提供外部能力，输出层交付代码/测试/文档，度量层把结果回灌配置中心。这个分层的含义是：约束应尽量下沉为工作台与 MCP 层的可执行配置（Rules、Safety、权限），而不是写在每一条 prompt 里；上下文治理属于工作台层的配置中心；评估与度量必须独立成层，否则它会被生成方自己定义标准，永远给出"通过"。^[raw/articles/tencent-harness-engineering-team-specification-2026.md:47-49]

### "3+1 Phase" 对齐的是可验证检查点，不是组织角色
Planner / Generator / Evaluator / Archiver 表面上像四个岗位，实质上是用角色名包装的四个检查点：requirements.md 必须经人工审核才生成 task.md，代码必须过规范与逻辑检查才可交付，变更必须归档才形成长期记忆。它的设计意图是让上下文交接发生在文件上而非人的记忆里——因此这四个角色完全可以由同一个模型在四次不同的上下文中扮演，[[concepts/agent-role-specialization|Agent 角色专业化]]与 [[concepts/sdd-specification-driven-development-harness|SDD 规范驱动开发]]在小团队同样可落地。^[raw/articles/tencent-harness-engineering-team-specification-2026.md:29-36]

### 3 阶段路线图排的是信任，而非工具
基础建设（1-2 周）先固化"每个人手上的 harness 一致"，工具接入（2-4 周）再放大能力边界，持续优化阶段才引入度量看板与规范迭代。顺序不可颠倒的关键在于：没有一致基线与可回滚的 Git 约束，提前接入 MCP 只会让不可复现的失败更频繁地发生，度量数据也会因口径不一而失去意义——度量用来校准规范，规范必须先存在。^[raw/articles/tencent-harness-engineering-team-specification-2026.md:51-55]

## 实践启示

1. **先写下来，再谈工具**：落地顺序是先写分层的 Rules（结构/行为/安全约束）与 requirements、task 模板，再选工作台；规范文件是 Git 资产，不是聊天记录。
2. **把约束分三层**：不可违反的硬性红线进 Rules，推荐做法沉进 Skills，兜底交给 Safety 策略；三者分开才能在评审时判断是规范问题还是工具问题。
3. **度量之前先定口径**：AI 代码占比、交付量与 Bug 率要在第二阶段跑通前定义清楚，否则第三阶段的看板只是在解释噪音。
4. **让评估独立于生成**：设定 L1 编译/Lint、L2 单元测试、L3 规范检查、L4 人工加 AI 联合审查，并把 AI Code Review 放在合入前；评价标准不能由生成方自己写。
5. **先清扫 7 种反模式**：把环境配置写进 prompt、跳过 Plan、不 Review 直接合入、不做安全审计、单个 Rules 文件写数千行、Skill 无测试即共享、不做度量——每一条都可做成 harness-audit 的自检项。
6. **照搬角色，不必照搬人头**：Planner → Generator → Evaluator → Archiver 可由同一模型分四次上下文扮演，保留的是检查点，参考 [[entities/tencent-ai-team-knowledge-harness|团队知识 Harness]] 的做法把沉淀环节固定下来。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

