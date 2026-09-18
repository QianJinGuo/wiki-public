---
title: "Agent 即分布式系统：报销工具的三阶段进化（LLM 纯函数 → Agent Loop → Skill+MCP）"
created: 2026-09-18
updated: 2026-09-18
type: entity
tags: [agent, distributed-systems, harness, idempotency, state-persistence, mcp, skill, error-handling, durability, aws]
sources: [raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化]
confidence: 0.8
provenance_state: extracted
---

# Agent 即分布式系统：报销工具的三阶段进化（LLM 纯函数 → Agent Loop → Skill+MCP）

以一个发票报销自动化工具（Concur/Emburse 场景）的三次版本演进为切片，穿过 AI 应用的三种形态：插件里的 LLM 纯函数、Agent Loop 驱动的浏览器扩展、Skill + MCP 的通用 Agent 助手；并给出第三种形态下四个真实生产事故，以及它们与分布式系统原则（超时、幂等、状态污染、缓存失效）的一一对应和工程修复。核心断言是：当 Agent 开始调用外部服务、产生真实写操作的那一刻，它就是一个分布式系统，必须用分布式系统的工具箱来约束，而不能靠「让用户确认一下」兜底。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

## 三阶段形态对比

| 维度 | 第一阶段 Plugin | 第二阶段 Extension + Agent | 第三阶段 Skill + MCP |
|---|---|---|---|
| LLM 角色 | 纯函数（PDF → JSON） | Agent Loop Driver | 通用 Agent Driver |
| 写操作 | 无（LLM 不执行任何外部操作） | 有（前端 Tool 创建 Expense，受控） | 有（直接调 MCP，无浏览器兜底） |
| 分类/编排 | 人工（按文件夹结构分类） | Agent 自动分类 + 固化提示词编排 | Agent 驱动 MCP 自动分类 + Skill 编排 |
| 错误影响范围 | 封闭（错在输出里） | 受控（用户在场） | 开放（可能影响外部系统） |
| 错误处理 | 确定性重试 | Agent 重试 + HITL 兜底 | 幂等键 + 状态持久化 + 先查后做 |
| 用户角色 | 分类器 + 操作员 | 审核者 | 观察者（甚至不在场） |
| 确定性 | 100% 确定性 | 半确定性（Agent + HITL） | 概率性 + 分布式系统约束 |

第一阶段把 LLM 当作 `PDF → JSON` 的纯函数嵌入 Tampermonkey 插件：LLM 没有任何副作用，错误被封闭在输出里，写操作（创建 Expense）由前端代码经 Concur GraphQL 完成，重试策略也是确定性的（`withRetry()` 对 5xx/超时重试，对 400/401/403/404 不重试）；此阶段的隐性成本是「人是分类器」——用户必须按预定义文件夹结构手工整理票据。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

第二阶段升级为 Plasmo Chrome Extension + Strands Agents SDK（部署在 Amazon Bedrock AgentCore Runtime），Agent 获得三个 Tool：后端 `processInvoices`（OCR/分类/匹配/合并 PDF）、跨端 `set_form_field`（改前端表单状态）、前端 `createExpenses`（用浏览器 Session Cookie 创建 Concur 条目）。关键转变是 Agent 第一次产生了会影响现实系统状态的写操作，但风险仍受控：写操作发生在用户浏览器里、有 HITL 节点审核、用户始终在场，且自然语言可用来修改字段（如「所有话费按 75% 报销」）。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

第三阶段的基本判断是「Agent Loop 不需要自己搭了」：个人 Agent 助手（Amazon Quick、Claude Code、Codex 等）本身就是功能完备的 Agent Runtime，场景能力沉淀为 Skill（可复用工作流描述）+ MCP（标准化工具接口）。用户不再需要自己部署、不再需要上传发票，且助手自带记忆。开发范式随之翻转——Agent 既是产品也是开发工具：作者用 Quick 的浏览器自动化能力录一遍手工报销操作，一次就提取出全部 API 接口、参数、可从系统侧获取的字段（policyId/expenseTypeId）、需用户输入的字段与枚举模板，原本数天的逆向工程缩短为一次对话。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

## 确定性消退：第三个阶段才真正需要 Harness

三个阶段背后有一条主线：工程确定性在持续消退。第一阶段是 100% 确定性——LLM 虽是黑盒函数，但被确定性代码包裹；第二阶段引入概率性（Agent 可能选不同 Tool、不同参数、不同顺序），但 HITL 是硬性兜底；第三阶段是全自动执行，「概率性 + 分布式」叠加，用户不在场时无法再指望人工确认，只能靠工程手段约束——幂等键（让重复执行安全）、状态持久化（让中断可恢复）、先查后做（让超时可控）、数据来源约束（让幻觉不可能发生）。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

文章用一句自问收束这条线：构建 AI Agent 时最该问的是「当它犯错时，系统允许它做到哪个程度」。第一阶段答案是「错在 OCR 输出里，无所谓」；第二阶段是「错到表单里，用户能看到」；第三阶段是「可能错到 Concur/Emburse 的生产系统里」。模型能力可以降低错误率，但无法消除网络失败、过期数据与状态不一致——这正是必须引入分布式系统工具箱的理由。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

## 四个生产事故 → 原则 → 修复

| # | 事故现场 | 根因 | 工程修复 | 对应分布式系统原则 |
|---|---|---|---|---|
| 1 | 幻觉 reportId：后续 `add_expense` 从对话上下文「回忆」了一个不相关 ID，全部操作静默失败 | 上下文即状态，而对话上下文是无来源标注、无失效机制的状态 | 创建后立即把 reportId 写入 `expense_state.json`，后续只从文件读，永不从上下文取 | 上下文即状态，状态会污染 |
| 2 | 批量 `add_expense` 超时后重试，导致每张发票出现两个条目、触发 blocking 告警 | 客户端超时 ≠ 服务端失败；MCP 调用跨网络边界，两端状态可能不一致 | 幂等键（中国发票号 invoiceNo 是天然幂等键）写入 state + 先查后做（超时后先 `get_expense_report` 按 amount/date/typeId/vendor 匹配）+ 连续 3 次超时则停止批处理并通知用户 | 超时 = 未知；幂等性必须焊死在工具层 |
| 3 | 同日往返两张高铁票（同商户同金额）被判为重复发票 | Agent 把对话上下文当数据源，按日期/商户/金额「推测」文件名，丢失了文件名的 `(1)` 后缀 | 强制数据来源约束：MCP 工具参数只能来自 `json.load()` 派生链，文件路径只能取 `groups[].primary_pdf` / `paired_pdf`，禁止现场手打或推测 | 记忆当缓存：可失效、带来源 |
| 4 | 上传收据时部分 Expense ID 已漂移，收据附到错误条目 | Concur 内部对账会在某些条件下重新分配 ID，而上一步缓存的映射被当成权威 | 每个关键步骤开头强制向权威数据源刷新（`get_expense_report` + 按 amount/date/typeId 重新映射），不信任本地缓存 | 缓存必须可失效 |

上述四条不是孤立技巧，而是同一类问题的四个切面：Agent 的对话记忆同时充当了「状态」「数据源」和「缓存」三种角色，却缺少来源标注、失效机制和持久化保证；把这三重身份拆回文件系统（状态）、源数据（数据源）与 API 权威查询（缓存刷新），失败模式才被逐个关闭。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

## 任意点可恢复的状态机

综合上面四个事故，Skill 落地成一个「任意点中断可恢复」的状态机：`expense_state.json` 存 `{invoiceNo → expenseId}`（幂等 + 持久化），`receipt_state.json` 存 `{expenseId → [imageIds]}`（记忆失效 + 权威源刷新）。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

Step 4（add_expense）的恢复语义：成功 → 写 state 继续；超时 → `get_report` 匹配，已存在则写 state，不存在才安全重试；校验错误 → 修复后重试（无重复风险）。Step 5（上传并附加收据）的恢复语义：上传前查 `receipt_state`，已附则跳过；upload 超时 → 安全重传（孤儿 image 无害）；attach 超时 → `receiptImageId` 已在则写 state 跳过。^[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化.md]

## 与既有 Harness 知识的关系

- 本文的「幂等键 + 状态持久化 + 先查后做 + 数据来源约束」四抓手，是 [[concepts/harness-long-running-task|长时任务 Harness]] 在「有外部写操作」场景下的具体化：长时任务关心的是中断与恢复，本文补上了中断后**外部副作用**如何不重复、不错位。
- 与 [[entities/building-reliable-agentic-ai-systems-martinfowler|Building Reliable Agentic AI Systems（Martin Fowler）]] 同属「可靠性工程」视角，但切入点不同：Fowler 侧偏架构模式与失败分类，本文侧偏一次真实演进中踩出的四个坑及其代码级修复。
- 第三阶段「Skill + MCP = 能力包管理」的形态判断与 [[entities/skill-mcp-software-package-management|Skill + MCP 软件包管理]]、[[entities/agentic-ai-system-architecture-harness-skill-mcp|Harness + Skill + MCP 体系架构]] 一致；本文额外给出「使用者也是开发者」的社区迭代闭环（Agent 执行记录还原现场 → 定位根因 → 改 Skill/MCP → 回馈 Slack 频道）。
- Tool 形态从「前端 Tool（浏览器 Session 兜底）」到「MCP 直连 REST API」的迁移，对应 [[concepts/harness-tool-design-evolution|Tool 设计演进]] 中「工具边界变化如何改变失败模式」的命题：同一动作跨过浏览器边界后，兜底消失，约束必须前移到工具层。
- 可观测性抓手（state 文件记录映射链）与 [[concepts/managed-agents-architecture|Managed Agents 架构]] 中「执行记录作为一等公民」的取向呼应——本文更进一步把记录直接用作恢复依据。

## 工程约束清单（可直接复用）

- 幂等键优先选业务天然唯一键（发票号、订单号），并在工具层而非提示词层实现。
- 任何「跨网络边界的调用」都不允许盲目重试；超时一律按「未知」处理，先查询服务端真实状态。
- 关键状态只允许从文件/数据库读取，禁止从对话上下文读取；禁止 Agent 在代码里手打或推测参数。
- 每个关键步骤开头向权威 API 刷新状态，不信任上一步的本地缓存。
- 批处理要控制扇出（文中取 max 5 并行），连续失败要有熔断点（连续 3 次超时即停止并通知用户）。
- 最终提交保持 user-gated（最小权限），即使流程已经全自动。

## 引用

- Salman Munaf（TikTok SRE 技术负责人）：「超时不代表失败，超时代表未知」「上下文即状态，状态会污染」「我们应该把记忆当作缓存来处理——它应该可以被失效，它应该带着来源信息」「你必须把幂等性焊死在工具层里」。
- 原文引用的演讲编译：《TikTok SRE 技术负责人：AI Agents 说到底就是分布式系统》（InfoQ）。

→ [[raw/articles/从-llm-ocr-到分布式-agent一个报销工具的三次进化|原文存档]]
