---

title: "Build Specialized Agent Workflows for Your Business with Amazon Bedrock"
created: 2026-07-24
updated: 2026-10-24
type: entity
tags: [aws, bedrock, agent, workflow]
sources: [raw/articles/build-specialized-agent-workflows-for-your-business-with-ama]
confidence: 0.65
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Build Specialized Agent Workflows for Your Business with Amazon Bedrock

> 原标题如此命名；实际栈为 Amazon Quick（业务前台）+ NVIDIA NeMo Relay（后端编排），AgentCore 只做 MCP 网关与运行时托管。

## 摘要

供应链团队通常不缺数据，缺的是把「某个供应商延迟了」这条信号在几十分钟内收敛成可执行决定的能力：计划员要同时核对采购订单、库存、客户承诺、合同条款、物流选项和审批政策，漏掉任何一项都可能把可控延迟放大成违约。AWS 这篇实践的答案是分工——Amazon Quick 做业务前台（仪表板上下文、结构与非结构化知识、100 多个动作连接器、MCP 动作触发），NVIDIA NeMo Relay 做后台决策工作流（注册工具、编排调查、返回带证据的排序缓解方案）。样例以「供应商 A 延迟七月到货」为场景，端到端产出三个风险项，每项带不同的缓解动作与审批路径。文章的价值不在「更快地看图」，而在把口口相传的检查清单变成可测试、可审计、可复用的决策流水线。^[raw/articles/build-specialized-agent-workflows-for-your-business-with-ama.md:12-24]

## 核心要点

- **问题定义**：仪表板解决「发生了什么」，决策需要「做什么、依据是什么、谁批」。两者之间隔着一次跨六个数据源的因果推理，这才是延迟来源。^[raw/articles/build-specialized-agent-workflows-for-your-business-with-ama.md:14-16]
- **Quick 的三层能力**：结构化数据（Quick Sight 数据集与分析）、非结构化知识（Amazon S3、Google Drive、SharePoint、Confluence、内部网页）、动作层（100 多个预置连接器，覆盖 Outlook、Slack、Jira、Asana，并可调用通过 MCP 暴露的 agent 工作流）。^[raw/articles/build-specialized-agent-workflows-for-your-business-with-ama.md:20]
- **NeMo Relay 的定位**：开源、框架无关的 agent 工作流库，管连接、评测、profiling 与优化，与 LangChain、LlamaIndex、CrewAI、Semantic Kernel 并存而不要求推倒既有栈。
- **四段式架构**：Quick（界面、上下文、触发）→ AgentCore Gateway（把工作流以 MCP 动作暴露）→ Lambda 适配器 → AgentCore Runtime（托管 NeMo 容器，FastAPI 接收 `/invocations` 并加载工作流）。
- **六个注册函数**：`po_risk`（延迟采购单与受损 SKU）、`inventory_exposure`（可供天数与缺货风险）、`customer_impact`（客户订单、收入、优先级、SLA）、`contract_policy`（合同、寻源、货运审批、替代料规则）、`logistics_options`（加急、调拨、替代供应商）、`mitigation_recommendation`（排序方案并输出 critic 检查）。
- **编排是配置而非代码**：YAML 声明函数清单，`supply_chain_risk_orchestrator` 作为入口按序接线，于是每一步都能单独埋点、单测、复用，并进入评测与 profiling。^[raw/articles/build-specialized-agent-workflows-for-your-business-with-ama.md:138-169]
- **两种交互模式刻意分开**：诊断类问题（「哪些供应商延迟最多」）由 Quick 用仪表板与知识源直接回答；只有「我该怎么办」才触发 MCP 动作，且调用前会向用户请求批准。
- **输出是决策包而非一段话**：风险摘要、收入影响、受影响客户、排序后的缓解动作、所需审批，以及回溯到采购单、库存、客户订单、政策条款、物流选项的证据。
- **可观测与评测是交付物**：trace 每项对应一个注册函数；逐工具延迟定位优化目标（样例端到端 657ms，采购单查询 267ms 与合同政策检索 196ms 合计占七成以上）；本地 eval harness 用 JSONL 用例比对风险等级、缓解动作、审批路由、证据覆盖四项。^[raw/articles/build-specialized-agent-workflows-for-your-business-with-ama.md:308-356]

## 深度分析

### 一、瓶颈不是「看见」，而是信号到决定的收敛成本

多数团队已经能在五分钟内看到「某供应商出问题」，却不能在一小时内回答「哪几张采购单、哪个客户的 SLA、走哪条替代路径、谁签字」。成本不在查询本身——每张表都查得到——而在**跨源对齐**：SKU 编码在 ERP 与 WMS 里可能不同形，库存快照有时点语义，客户承诺散落在订单系统与合同附件，审批政策写在没人完整读过的 SOP 里。人在压力下必然走捷径：只看最熟的指标、跳过合同小字、沿用上次做法。这才是错误决策的真实成因，而不是「数据不够」。

所以这套架构的价值不是把仪表板做得更好看，而是把「每次都走同一条检查清单」固化下来：一次判断不再依赖某位资深计划员当天是否记得第五步。

### 二、前台 / 后台分工买到了什么，代价在哪里

Quick 承担身份、权限、界面与上下文注入，NeMo Relay 承担工具注册、编排、trace 与评测。这个切分买到三样东西：业务用户不必学新工具；决策工作流可独立版本化、独立评测；同一套工作流能被别的门面复用（Teams、Slack、自建门户），因为对外契约只是一个 MCP 动作。

代价同样明确。**MCP 是单点契约**：Quick 侧只看得见「一个动作」，看不见内部六步，任何 schema 变更都要两侧同步，而两侧往往由不同团队维护。**上下文被切成两半**：仪表板数据在 Quick 侧，工具查询结果在 NeMo 侧，一旦口径不一致（SKU 命名、时间窗、库存快照时点），推荐结论就会和用户屏幕上的数字互相矛盾——这是分层架构最常见也最难查的生产事故。**责任边界模糊**：推荐出错时，是编排、数据还是检索的问题只能靠 trace 定位，否则等于把调试成本转嫁给运维。

### 三、带证据的推荐与聊天回答，是两种不同的信任构造

聊天回答的正确性只能靠「读起来是否合理」判断，因此天然不可回归、不可审计。决策包不同：风险等级、缓解动作、审批路由、证据覆盖这四项是可断言字段，既构成自动化测试的期望值，也构成事后审计的轨迹。审计员问「为什么给 SKU-1044 选了部分空运」，系统能回溯到具体那张采购单、那批库存、那条货运审批条款，而不是「模型认为如此」。

更关键的是，证据把争论从结论层挪到了证据层。业务方很难反驳「建议空运」，却能指出「这条紧急空运条款不适用于亚太区」——后者才有信息量。critic 检查与 evaluator flags 同理：把「模型很自信」换成「检查是否通过」。

### 四、从 demo 到生产，四个必须先成立的前提

**数据契约**：SKU 与供应商 ID 的规范形式、库存快照的时点语义、收入归属口径必须先定义。样例用 CSV，生产换成 Athena、RDS、ERP/WMS/TMS API 或其他 MCP 工具——换数据源不能换语义，否则编排逻辑静默失效。

**政策即机器可读规则**：合同条款、货运审批、替代料规则必须能返回结构化判定，而不只是检索到一段 PDF 让模型总结。「有上下文」与「可执行」之间差着一条明确阈值。

**连接器权限**：100 多个连接器既是能力也是攻击面。每个工具只应拿到该函数所需的最小数据集与操作集，行级列级权限要与 Quick Sight 侧一致，否则前台看不见的数据会从后台工具漏出去。

**人审边界**：读动作可以自动，写动作（改供应商、建工单、通知客户、更新 ERP）必须过审批闸门。样例把动作批准做成显式交互，这是生产前提而非演示细节。

缺任何一条，系统都会退化成「看起来能跑、实际没人敢用」的 demo——这正是多数企业 agent 项目的终局。

## 实践启示

1. **先定决策包的字段，再选框架。** 把「风险等级 / 缓解动作 / 审批路由 / 证据」写成 schema，编排器、评测集、审计报表都从它派生。字段定不下来，说明业务本身还没想清楚。
2. **按「诊断」与「行动」切两个入口。** 诊断走只读上下文，行动走显式审批；混合入口让用户无法预判这次回答会不会真改数据，信任经不起一次误操作。
3. **把检查清单拆成单一职责的注册函数。** 一个函数对应一个可独立验证的判定，而不是塞进巨型 prompt。拆开后每一步能单测、能计时、能定位失败。
4. **第一天就采 trace、逐工具延迟和评测用例。** 「这次改 prompt 有没有变好」必须可回归验证，否则只能凭感觉调参；延迟数据还能直接排出优化顺序。
5. **先在三五个 SKU 的真实场景跑通，再换数据源、再扩范围。** 样例数据的作用是校准语义而非演示效果；语义对齐后替换生产数据源才是一次可验证的迁移。
6. **安全默认值要反过来设。** 演示可以无鉴权，生产必须鉴权优先（CUSTOM_JWT / OAuth-OIDC，校验签发方、受众与密钥轮换）；派生部署应默认拒绝无鉴权。

## 相关实体

- [[entities/amazon-quick-accelerating-the-path-from-enterprise-data-to-ai-powered-decisions|Amazon Quick：从企业数据到 AI 决策]]
- [[entities/amazon-bedrock-agentcore-gateway-mcp-extension|AgentCore Gateway 的 MCP 扩展]]
- [[entities/amazon-bedrock-agentcore-runtime-deep-dive-and-scenario-analysis|AgentCore Runtime 深度解析]]
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]
- [[concepts/evaluation-harness-design|评估 Harness 设计]]

→ [[raw/articles/build-specialized-agent-workflows-for-your-business-with-ama|原文存档]]
