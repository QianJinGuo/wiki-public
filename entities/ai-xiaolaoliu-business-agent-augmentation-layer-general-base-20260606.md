---

title: "小刘商业 Agent 增强层通用基座"
created: 2026-06-10
updated: 2026-09-15
tags: [agent, architecture, code, data, evaluation, llm, memory, mlops, observability, prompt, rag, robotics, search, security, tool-use]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 小刘商业 Agent 增强层通用基座

## 摘要

这份材料主张：业务团队做 Agent 不该从零搭框架，而应把 Codex、Claude Code 这类通用 Agent 基座当作现成的执行中心，团队只补它天然不知道的部分——业务知识、内部工具、流程规则、权限边界、评测集与线上观测。文章称这条路线为「业务 Agent 增强层」，并用能力边界表划清了基座与团队的职责。^[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606.md]

## 核心要点

- 默认复用成熟通用 Agent 作执行引擎，只在强私有化、封闭运行时、极端时延或合规受限等少数场景才自研专项 Agent。
- 立项前用 10 到 30 个真实 case（工单、告警、代码修改、发布检查）跑裸基座 baseline，先看清原生能力边界再决定补什么。
- 裸基座能解决约 60% 的任务，就补剩下 40% 的短板；完全失效时先定位是缺知识、缺工具、缺流程，还是安全边界不允许接入。
- 基座提供「智能」（任务理解、代码与文件、工具调用、上下文、人机协作），团队补「落地」（术语与验收标准、仓库规则、工具 schema 与权限、知识库、确认点）。
- 增强层分六层：业务入口、知识与上下文、工具能力、流程编排、安全治理、评测观测；第一版不做平台，先跑通一个真实场景闭环。
- 知识库是判断材料而非资料仓库，应从「问题清单」反推沉淀什么；评测必须看增量，保留 baseline 才知道投入有没有回报。

## 深度分析

### 为什么不自己搭 Agent Framework

很多团队一说要做业务 Agent，第一反应是搭一套自己的 Agent Framework：规划器、执行循环、工具调度、记忆、权限、人机交互。方向听起来完整，落地却容易把团队拖进基础设施泥潭：通用基座已经会读代码、拆任务、调用工具、按观察调整路线，让团队再维护一个「比 Codex 更懂代码、比 Claude Code 更会工具调用」的大系统，投入产出比极低。^[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606.md]

### 通用基座与增强层的边界划分

业务 Agent 失败，很多时候不是模型笨，而是边界混乱。任务理解上，基座负责把自然语言目标拆成计划并边执行边修正，团队补任务模板、验收标准、停止条件与业务术语；代码与文件上，基座负责读代码、改文件、跑测试、总结 diff，团队补仓库规则、模块边界、测试命令与发布约束。^[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606.md]

其余四个域同理：工具调用上基座负责选工具、填参数、解析结果，团队补稳定 schema、错误码、权限、dry-run 与回滚；上下文处理上基座负责组织信息与压缩执行状态，团队补知识库、历史案例、检索策略与记忆写入规则；人机协作上基座负责不确定时询问用户，团队补确认点、交付格式与责任边界；质量保障上基座只能完成单次任务，团队必须补评测集、回归体系与失败归因。执行循环也不必自研，团队要定义的是协议：读输入、给短计划、取证可追溯、保留证据、高风险动作前人工确认、交付结论与证据。^[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606.md]

### 业务增强层需要补的六类能力

六层各司其职：业务入口层承接用户任务（飞书机器人、Web、CLI、工单入口）；知识与上下文层补业务语境（SOP、历史案例、仓库规则、服务画像、记忆策略）；工具能力层让 Agent 查得到、做得动（MCP Server、内部 CLI、OpenAPI、日志、CI、发布、配置查询）；流程编排层约束推进方式（任务模板、审批点、人工确认、失败兜底、交付格式）。^[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606.md]

安全治理层守住权限与变更边界（读写分离、最小权限、dry-run、敏感动作确认、回滚），评测观测层判断增强是否真的有用（baseline 对比、回归集、trace、指标、失败归因、成本统计）。知识与工具必须一起做：知识回答「怎么理解」，工具回答「怎么取证」和「怎么执行」；只有知识没工具，Agent 停在建议层，只有工具没知识，Agent 拿到数据却不会判断。^[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606.md]

### 增强层如何被验证

复用通用 Agent 的路线，评测必须看增量：增强版跑得不错并不代表知识库、工具和流程都有效，也可能裸基座本来就能做到，所以必须保留 baseline。上线门禁可以朴素但必须可执行：关键路径通过率不低于 90%，P0/P1 case 必须全部通过，高风险动作确认覆盖率 100%，无证据结论率控制在 5% 以下，知识、工具与流程变更后必须跑回归并归因。评测集除输入与期望行为外，还要带 required_evidence、forbidden_behavior 与 label。^[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606.md]

节奏上按月推进而非按平台想象推进：第 1 周跑通裸基座 baseline（含 20 到 50 条初始 case），第 2 周补最小知识库与关键工具，第 3 周补流程与可控性，第 4 周建立 baseline 对比评测，第 2 个月小流量试用，第 3 个月再谈跨场景复用。如果一个场景连 20 条高质量 case 都凑不出来，先别急着做 Agent——没有 case 就没有 baseline，后面所有优化都会靠体感。^[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606.md]

## 实践启示

1. 先跑裸基座 baseline：用 10 到 30 个真实 case 记录原生能力边界再定增强方向，别一上手就重写规划器与执行器。
2. 把业务规则从超长 prompt 搬进带 owner 与 freshness 的知识块或工具，否则长期难更新、难评测、难定位回归。
3. 工具只接 3 到 5 个高价值项，参数结构化、返回成功标志、数据、证据与错误码，并把工具失败当一等信息记录。
4. 写操作、权限变更、外部通知、删除一律进确认门，配 dry-run 与回滚，敏感动作 100% 确认或审批。
5. 用含 expected_behavior、required_evidence、forbidden_behavior 与 label 的评测集跑回归，让每次改动的收益可分。
6. 不要过早多 Agent 化：先做扎实单闭环、沉淀失败样本、做实评测闭环，再谈平台化与规模化。

## 相关实体

- [[entities/你不知道的-agent原理架构与工程实践-v2|你不知道的 Agent 原理与工程实践]]
- [[entities/karpathy-vibe-coding-agentic-engineering|从 Vibe Coding 到 Agentic Engineering]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr|AgentOps：规模化运营 Agentic AI]]
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构与生产设计]]
- [[entities/agent-harness-observability-production|Agent Harness 可观测性]]
- [[entities/anthropic-官方生产级-agent-最佳实践12-个可复用的-mcp-设计模式|Anthropic 生产级 Agent 与 MCP 模式]]

→ [[raw/articles/ai-xiaolaoliu-business-agent-augmentation-layer-general-base-20260606|原文存档]]
