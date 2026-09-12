---
title: "在数据所在处构建 Agent: CrewAI + Snowflake 企业级 Agent 部署"
created: 2026-06-30
updated: 2026-09-11
type: entity
tags: [agent, crewai, snowflake, enterprise, data-governance, agent-deployment]
sources: [raw/articles/how-to-build-agents-where-data-already-lives]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 在数据所在处构建 Agent: CrewAI + Snowflake 企业级 Agent 部署

> CrewAI 提出企业 Agent 的瓶颈已从用例和模型转向"治理下的构建吞吐量"——如何在权限、数据边界、审批路径、审计日志等现有业务系统内高效构建和部署 Agent；与 Snowflake 的深度集成是解决这一问题的关键路径。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

## 摘要

企业手里有 20、100、有时 800 个已识别的 Agent 用例，而 AI 团队一年实际只能交付约 10 个——瓶颈已从"想法与模型"迁移到"治理约束下的构建吞吐量"。CrewAI 的判词是：**"自治需要信任，信任需要控制，控制扼杀自治——当它被施加在错误的层级时。"** 其解法是：把治理上移为平台层能力、把构建下沉到最贴近业务的人，并借与 Snowflake 的深度集成，让 Agent 在数据本已存在、且已被业务信任的边界内运行。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

## 核心要点

- **吞吐量才是真瓶颈**：用例不缺（20/100/800），模型也不缺；缺的是在复杂运营中快速"构建—部署—扩展"Agent，而不让十名工程师成为每个工作流的永久所有者。
- **治理必须左移**：构建者需预先知道能碰哪些数据、模型、工具，什么要人工复核，失败时发生什么；体验引导合规选择，运行时强制执行。
- **数据有重力（data gravity）**：企业数据不愿移动，住在数据湖、SaaS、Snowflake、Salesforce 与内部系统里，其访问策略自有正当理由。
- **两条坏路**：跨边界拉取上下文 → 治理崩溃；锁死在单一系统 → 够不到足够远、做不成有用的事。
- **治理化编排（governed orchestration）**：Agent 跨系统协同，平台从第一个设计决策起即承载治理，而非事后补丁。
- **Snowflake 是天然数据枢纽**：它已掌握企业信任的权限、血缘与访问模型；Cortex Analyst（结构化）与 Cortex Search（非结构化）经 Cortex Agents 协调数据访问，CrewAI 现已集成两者。
- **DocuSign 是工作示例**：在 CrewAI、Snowflake、Salesforce 与内部系统上构建运营闭环，Agent 工作坐在业务流程**内部**。
- **把构建者变成赋能者**：工程团队造"飞轮"（集成、平台控制、可复用组件、护栏），离业务最近的人在其边界内自建。

## 深度分析

### 瓶颈迁移：用例不缺，缺的是治理下的吞吐量

CrewAI 的观察是：工具越来越好、"构建被商品化"的说法越来越多，但企业从 Agent 获得持久业务价值的速率并未同步提升。一个潜在客户手里握着 800 个已识别用例（后来成了付费客户），而 AI 团队一年能交付的仍只有 10 个左右。这里的算术很残酷：交付速率一旦被 AI 团队的人力上限锁死，用例清单涨得越快、等待队列越长，组织对 Agent 的信任反而因"永远轮不到我"而流失。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

所以真正的约束不是"能不能造出一个 Agent"，而是"能不能在权限、数据边界、审批路径、采购规则、审计日志、以及从未为 agentic 负载设计过的业务系统**内部**，把 Agent 批量造出来、部署下去、扩展开来"。CrewAI 把这条约束命名为 **building throughput under governance**：治理不是吞吐量的对立面，而是它的前置条件——把治理当"事后合规审查"的组织会在规模化时被迫停下补作业，把它内建进平台的组织才能让更多人同时动手。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

### "自治—信任—控制"悖论：控制施加在哪一层

> Autonomy requires trust, trust requires control, control kills autonomy — that is, when it is applied in the wrong layer.

这句话的锋利之处在最后半句。控制本身不扼杀自治；**把控制放在错误的层**才扼杀自治。若控制以"每个工作流逐个审批、每个团队各管密钥与供应商"的形式出现，治理成本就按 Agent 数量线性摊开，构建者每前进一步都要等审批，吞吐量必然趋零——这正是"control kills autonomy"的机制。

而如果同一套控制被放到**平台层**——统一策略、继承权限、集中的模型访问与密钥管理、统一的审计与遥测——构建者就是在一个已经合规的"围栏"内自由活动，无需逐案重复证明合规，控制反而**提升**吞吐量。可操作的区分是：错误的层 = 逐工作流的门禁，随构建次数消耗治理预算；正确的层 = 平台级策略 + 继承权限，一次投入、全下游复用。CrewAI 的架构据此分层——**平台层**设定控制（权限、模型访问、数据边界、密钥、遥测、FinOps），**运行时层**让工作流活过演示期（扩展、人工复核、执行上限、可观测性、重试），**构建层**则让更多人创建 Agent。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

配套的一点是"agency 的位置"：多数 Agent 架构把自主性当成系统中心，而在生产中它应被**小心安置在确定性流程之内**——Flows 提供的正是这条确定性主干（时序、状态、分支、重试、升级）。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

### 数据重力，与"在数据所在处构建"

企业数据有很强的重力——它不想移动。上下文散落在数据湖、SaaS 应用、Snowflake、Salesforce 和内部系统里，每处的访问策略都服务于正当目的。由此产生的张力是：Agent 跨太多边界拉取上下文，治理就崩了；被困在单一系统里，又够不到足够远。团队因此卡在两个坏选项之间——要么受治理但狭窄，要么铺得够宽却制造一地治理烂账，两者都拿不到规模化、高影响力的生产 Agent。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

"在数据所在处构建"之所以化解张力，是因为它把**上下文组装放进既有数据边界之内**，而不是把敏感企业上下文复制进无人管理的中间层。实现上，CrewAI Agent 可直接连上 Snowflake 托管 MCP server 与 Cortex 解决方案：Cortex Analyst 处理结构化数据、Cortex Search 处理非结构化数据、Cortex Agents 提供 Snowflake 原生推理，SQL 执行与自定义工具都成为工作流里的工具节点。Snowflake 的角色与策略保持原样，Agent 使用其上下文而不把数据拖进非受管路径；同时 CrewAI 也能以 Snowflake 作受治理的**模型访问通道**，客户带自己的凭证，从数据已所在的同一受管环境触达主流与开源模型。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

这解释了为何**平台级集成优于每用例的定制连接器**：后者意味着每个团队各管一套密钥、一个供应商、一条审批路径，治理成本随用例数复利增长；前者让受治理的工具与模型一次打通、处处复用。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

### DocuSign 案例、扩展模式与可复用基础设施

DocuSign 展示的是一套客服外联系统：从用量数据、合同上下文与内部标准中识别目标客户，在 Snowflake、Salesforce、内部系统、PDF、网页与向量数据库之间调研，撰写并校验个性化外联，走人工复核与投递，再度量迭代。CrewAI Flows 负责协调：Identifier Agent 筛选客户、Researcher Agent 汇总网络与内部上下文、Composer Agent 撰写信息、Validator Agent 检查质量与幻觉风险；未达标者在推进前被拦下，达标者被路由到内部系统、投递、可观测性、Slack 与人工复核。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

真正重要的是**架构上的证明**：系统从一次性的"销售代表备料"演进为可重复的运营闭环，扛住了 A/B 测试、扩展出首个用例之外。如今双方合作已覆盖全球用例、支持远超 GTM 的职能——这正是健康的 Agent 基础设施应有的扩展模式：从一条业务线渗进后台职能，Agent 经逐轮"执行—度量—精炼"与业务纠缠在一起，最终不再是副项目，而是操作系统的一部分。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

但"每个工作流配十名工程师当永久所有者"注定不可扩展。出路是把构建者变成赋能者：与其让工程师写每个工作流，不如让他们造飞轮——集成、平台控制、可复用组件、护栏、可复用的 Agent / 工具 / 技能仓库——再让离业务最近的人在这些边界内构建。CrewAI 以 Studio 作构建面，以 AMP 作控制面（策略、部署、可观测性、模型访问与权限），治理由此成为**架构层**而非每 Agent 的门禁。^[raw/articles/how-to-build-agents-where-data-already-lives.md]

需标注的定位问题是：这是 CrewAI 自家博客，在为它与 Snowflake 的集成做论证。**结构性**论断——数据重力、治理吞吐量、控制层次论、平台复用优于定制连接器——换掉厂商名依然成立；**推广性**部分——"65% 的 Fortune 500 在用""20 亿次执行""数百个 Agent 已在用 Snowflake"——是规模声明而非证据，且 DocuSign 闭环通篇没有吞吐量或 ROI 数字。该案例应作**架构模式**而非**效益证明**引用。

## 实践启示

1. **把治理当作架构层，而非每个 Agent 的门禁**：统一策略、继承权限、集中模型访问与密钥管理，让控制一次投入、全下游复用；逐工作流审批只会线性消耗治理预算并压死吞吐量。
2. **把上下文组装放进既有数据边界内**：不要把敏感数据复制到无人管理的中间层；优先通过平台级集成（Snowflake 托管 MCP + Cortex）在原权限模型下取用上下文。
3. **平台能力优先于定制连接器**：先在平台层打通受治理的工具与模型访问，再让用例在其上组装，避免每团队各管密钥与审批。
4. **为复用而设计**：沉淀可复用的 Agent / 工具 / 技能仓库与模板，让工程团队造飞轮（集成、护栏、组件），业务侧在此边界内自建。
5. **把自主性放进确定性流程之内**：用确定性主干承载时序、状态、分支、重试与升级，只在划定位置引入 Agency，并预先定义人工复核与失败路径。
6. **以"构建吞吐量"作为核心 KPI**：不用 demo 数量衡量进展，而用"治理边界内每年稳定交付并维护多少条生产工作流"及"渗入多少职能"衡量。

## 相关实体

- [[entities/agent-data-governance-crewai-credential-patterns|Agent 数据治理与凭证模式]]
- [[entities/agent-development-crawl-walk-run-crewai-iterative|CrewAI 小步快跑开发法]]
- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai|Agent 安全三步序列]]
- [[entities/agentium-agent-framework|Agentium Agent 框架]]
- [[entities/snowflake-agentic-enterprise-summit-2026|Snowflake Agentic Enterprise Summit 2026]]
- [[concepts/enterprise-ai-adoption|企业 AI 采纳]]
- [[concepts/production-agent-engineering|生产级 Agent 工程]]
- [[concepts/model-context-protocol-mcp|Model Context Protocol (MCP)]]

→ [[raw/articles/how-to-build-agents-where-data-already-lives|原文存档]]
