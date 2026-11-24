---
title: "AWS BMad + AIDLC：用 BMAD Method 在 Serverless 上跑通可复制的 AI 驱动开发流程"
created: 2026-07-14
updated: 2026-11-14
type: entity
tags: [aws, aidlc, bmad, sdd, harness-engineering, ai-native-team, agent-orchestration, serverless]
sources: [raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# AWS BMad + AIDLC：用 BMAD Method 在 Serverless 上跑通可复制的 AI 驱动开发流程

> 原文存档：[[raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering|原文存档]]

AWS China Blog（2026-07-14）发表了一篇深度文章，介绍如何用 **BMAD Method（Build More Architect Dreams）** 框架在 AWS Serverless 上实现 **AIDLC（AI-Driven Developer Lifecycle）** 方法论，将 AI 驱动开发转化为可复制的工程流程。^[raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering.md]

## 核心架构：AIDLC 三阶段 + Review Gate

AIDLC 将开发过程划分为三个阶段，阶段之间由不可跳过的 **Review Gate（评审门禁）** 隔开：^[raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering.md]

1. **Inception（启动阶段）** — AI Agent 基于 Product Brief 并行产出 PRD、架构文档、测试策略；人定义约束、回应澄清、做即时反馈
2. **Construction（构建阶段）** — TDD 模式实现代码、CDK 基础设施、E2E 测试；人审阅每个 Story 的测试与实现
3. **Operations（运维阶段）** — 监控配置、告警规则、部署流水线定义；人提供业务 SLA 与运维约束

**关键设计思想**：Review Gate 不是"人唯一介入的地方"，而是"人必须签字放行的地方"。人始终在环（Human-in-the-Loop），贯穿每个阶段内部，不只是阶段间的关卡。AI 负责并行产出和起草，人负责定义约束、即时纠偏、做决策、最终验收。^[raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering.md]

## BMAD 的 Agent 约束机制

BMAD 使用 **TOML 配置文件**定义每个 Agent 的行为空间，这是整套方法的关键技术设计——没有约束的 Agent 在面对开放式问题时倾向于扩大讨论范围。以架构师 Agent 为例，配置包括：^[raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering.md]

- 技术栈已锁定（AWS Lambda + API Gateway + DynamoDB + CDK TypeScript）
- 数据库模式已确定（DynamoDB 单表设计）
- 认证方案已选（JWT via Lambda Authorizer）
- 时间约束（30 分钟完成架构文档）

约束机制确保各 Agent 在同一框架标准下并行工作。

## TDD 作为 AI 验证机制

文章强调**代码层面的正确性验证由 TDD 承担**，而非人工逐行审读。测试就是 AI 产出代码的可执行合同，比人工逐行 review 更高效可靠。人的精力应投入到 AI 看不出的语义和决策问题（涉及领域惯例、隐含假设、跨文档间接依赖），而机械的行级校验交给测试。^[raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering.md]

## 与现有实体的差异化

该文章与 wiki 中已有实体相比的独特贡献：^[raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering.md]

1. **平台化视角** — AWS 官方出品的 AIDLC 实施指南，从云平台基础设施角度讲述，区别于 [[entities/spec-kit-bmad-sdd-practice-yexiaocha|spec-kit-bmad-sdd-practice-yexiaocha]] 的个人实践者视角
2. **完整的约束机制示范** — 给出 BMAD TOML 配置文件的完整形态和分类，比 叶小钗 的对比文章更侧重 BMAD 作为框架的运行时机制
3. **Serverless 技术栈锁定** — 以 AWS Lambda + API Gateway + DynamoDB + CDK 作为具体实施参考，区别于其他泛化讨论
4. **Review Gate 的分阶段设计** — 给出了 Inception/Construction/Operations 各阶段的具体 AI vs 人工分工表
5. **多人团队协作模型** — 含 BMAD Party Mode 的多 Agent 交叉检查机制

→ [[entities/ai-dlc-zixun-ai-native-development-lifecycle|AI-DLC 紫讯 AI 原生研发实践]]
→ [[entities/harness-engineering-concept|Harness Engineering 概念]]
→ [[entities/sdd-spec-driven-development-summary-qoder|SDD 规格驱动开发总结]]
→ [[entities/spec-kit-openspec-superpowers-hybrid-harness|Spec-Kit + OpenSpec + Superpowers 混合 Harness]]

## 深度分析

### 约束前置：把临场发挥换成配置层确定性

BMAD 用 TOML 配置把两类信息固化进每个 Agent 的 persistent_facts：不可变前提（已锁定的 Lambda + API Gateway + DynamoDB 技术栈、单表设计、JWT 认证方案）与行为准则（不讨论选型替代方案、IAM 最小权限、产出必须能通过 cdk synth 验证）。这套机制的实际价值不是让 Agent"更聪明"，而是砍掉执行阶段的确认性往返——原文实测相同任务耗时从 45-60 分钟降到 20-25 分钟，减少的部分几乎全是确认对话。其原理在于：无约束的 Agent 面对开放式问题会本能地扩大讨论范围（要不要缓存？认证有哪几种方案？），这在探索阶段有价值，到了执行阶段就是纯粹的效率损耗。约束前置因此构成"可复制"的第一根支柱——产出的质量方差被压到配置层收口，而不是依赖每次对话的临场发挥；Agent 接到任务即可直接进入设计细节，跳过整个"对齐前提"的环节。`persistent_facts` 与 `principles` 的区分也有讲究：前者是 Agent 不可质疑、也无需向用户确认的既定事实，后者是每次决策都要遵守的行为准则——前者消灭事实层面的反复确认，后者压缩决策层面的自由度，两者共同把"跑偏"的空间收窄到接近零。

### 双工序门禁：机器比对与语义验收的分工

每个 Review Gate 由两道缺一不可的工序串联：先由 Party Mode 多 Agent 交叉评审扫"对不上"的硬性冲突（字段缺失、端点数量对不上、格式错位），几分钟内完成、覆盖面广；随后全角色人工评审处理"对得上但错了"的问题——领域惯例、隐含业务假设、跨文档间接依赖。原文的"删除"案例最典型：PRD 定义 DELETE 端点、架构文档设计软删除、测试标准写"删除后返回 404"，三方在字面上各自自洽，分歧藏在"删除"一词的语义理解里，Party Mode 全程未捕获，最终由具备业务经验的人工评审发现；对齐决策改为归档语义、验收标准同步改为"归档后返回 410 Gone 并含时间戳"。这个分工不可倒置：只跑 Party Mode 等于让 AI 给 AI 的产出签字；跳过机器比对则把人重新拖回机械校验。文中还给出一个高频事实——Gate #1 卡住问题的频率相当高，PRD 与架构之间的语义分歧几乎每次都会出现，而在此阶段修复只是改几行文档，流入构建阶段后就是代码与测试的成片返工。在多人协作场景下，这套门禁还与"规划收敛、实施发散"的团队模型咬合：Inception 阶段一人主笔、全员按角色评审，Construction 阶段按 Epic 并行发散。

### 回流闭环：变更驱动执行，而非工具层修补

当执行过程暴露需求遗漏或设计缺陷时，文章强调的正确路径是回到上游：更新 BMAD 的 Agent 配置或重跑相关工作流、修订 Spec，再让变更重新驱动一次执行——而不是在 IDE 里直接改代码绕过约束。这条"执行 → 回流上游 → 重新驱动"的闭环，正是 AIDLC 与 Spec 驱动方法可迭代、而非一次性瀑布的关键：约束始终是上游的 PRD、架构文档与 Spec，代码只是它们的产物，任何工具层的私自偏离都会立即破坏跨阶段一致性。这也解释了为什么方法论要绑定一个约束优先的执行环境——BMAD 的 Agent 以 Skill 形式运行，激活即加载约束，各 Skill 互不越权访问上下文，使"不跑偏"从人工纪律变成工具保障。

### 职责分层：测试管正确性，人管语义与决策

文章把验证责任切成互不重叠的两块：代码层面的正确性由 TDD 承担——先写测试定义预期行为、再实现使其通过，测试即 AI 产出代码的可执行合同，比逐行 code review 更高效也更可靠（PRD 的验收标准因此必须具体到可自动化，如"无效 JWT 返回 401 且响应体含 error 字段"）；人的精力则集中于 AI 看不出的语义与决策——"删除是物理删除还是归档"这类领域判断。再叠加强制对话隔离（每个独立任务、每次角色切换都开新对话，防止长对话中上下文累积导致 Agent 行为漂移），四个机制——TOML 约束、双工序门禁、TDD 验证、对话隔离——组合成完整闭环：AI 提供速度，人提供判断，机制承接两者之间的全部衔接，使"AI 高速产出、人全程把关"变得可控且可预测。相关背景可参见 [[concepts/sdd-specification-driven-development-harness|SDD 规格驱动开发]] 与 [[entities/spec-kit-bmad-sdd-practice-yexiaocha|叶小钗的 SDD 实践对比]]。

## 实践启示

1. **把约束写进配置而非对话**：技术选型、时间预算、质量标准应作为 persistent_facts 随 Agent 启动自动加载；省下的确认对话就是直接的交付速度，且约束可随项目沉淀复用。
2. **门禁必须双工序**：先用多 Agent 交叉评审扫硬性不一致，再由全角色做语义验收——任何一环被跳过，签字都失去意义。
3. **别让人逐行审读 AI 代码**：把行为正确性交给测试（验收标准具体到可自动化），人只处理语义分歧、隐含假设与关键决策。
4. **缺陷回流上游修**：执行中暴露的需求或设计问题，应修 Spec 与 Agent 配置后重新驱动执行，而不是在代码里绕过约束打补丁。
5. **对话隔离是纪律不是建议**：Story 实现、门禁评审、角色切换各自开新对话，防止上下文污染导致行为漂移。
6. **Brownfield 先做上下文发现**：让 AI 扫描代码库产出上下文确认卡、人工确认后生成 Agent 配置，并写入"破坏现有测试立即停止"的硬约束，每次修改后跑完整测试套件，不允许回归累积。

## 局限性

文章基于一个 Workshop 场景演示流程，Agent 配置包含时间限制等 Workshop 特有约束（如"30 分钟内完成架构文档"），并非生产推荐的配置值。实际项目中时间预算和技术约束应根据系统复杂度灵活调整。^[raw/articles/aws-bmad-aidlc-spec-ship-reproducible-engineering.md]
