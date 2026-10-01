---
title: "AI 取数信任基础设施：代号层 + 确定性 SQL 引擎 + Skill 门禁"
created: 2026-08-28
updated: 2026-10-01
type: entity
tags: [ai, data-retrieval, trust-infrastructure, semantic-layer, codename-layer, deterministic-sql, nl2sql, skill-gate, auditability, taobao, data-warehouse, text-to-sql, data-agent]
sources:
  - raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# AI 取数信任基础设施：代号层 + 确定性 SQL 引擎 + Skill 门禁

## 核心命题

AI 在复杂数仓环境中取数（将业务需求转化为 SQL 并取回数据）天然会产生幻觉、难以被信任。淘天集团-直播技术数据团队提出的解法是 **"AI 思考能力 + 工程确定性"** 的分工：AI 负责语义理解与用户对齐，工程负责确定性的 SQL 生成——用工程的强确定性弥补 AI 的不确定性，构建一套值得信任的 AI 取数基础设施。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

取数流程的核心矛盾是**"同一个东西的代号没有标准、没有达成共识"**：用户说 gmv，数仓里只有成交金额；成交金额又有多个口径（一个限制了字段A='Y'，一个没限制）。AI 独立取数时，在"理解数仓事实行为代号"与"写语义正确的 SQL"两个环节都会产生幻觉——数仓持续变化、冗余、膨胀导致多义性，AI 可以生成语法完美但语义错误的 SQL。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

## 三支柱架构

### 1. 代号层（Codename Layer）— 让同一个东西有标准

代号层是**业务行为的标准、唯一的数据代号化表达**，本质是一个字典/密码本。指标 = 业务主体行为的度量；维度 = 业务主体行为发生的环境。比如"在直播间成交的金额"这一业务行为的代号是"成交金额"，只有这一个名称——通过统一的直播数据大字典，让 AI 与用户用同一种语言沟通。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

代号层以**逻辑表为核心**构建（对整个数仓业务过程的又一次抽象），通过字段映射、自定义 CTE 等功能，把"不标准"的真实数仓挂载到标准的代号层上——即使真实数仓不按标准组织，也能从其中抽象出代号的具体实现。对 AI 而言代号层是**沟通共识层**（解决代号对齐问题），对 SQL 引擎而言是**数仓映射层**（解决 SQL 书写问题）。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

### 2. 确定性 SQL 引擎 — AI 不写 SQL，只出查询请求

把写 SQL 的复杂性屏蔽掉：AI 只输出用户的需求，构造一个**轻量级查询请求（JSON）**，工程将查询请求翻译为**确定性 SQL**。关键分界——**AI 不生成 SQL，SQL 由确定性工程规则生成**，从源头消除 AI 写 SQL 的幻觉。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

引擎内部的确定性规则包括：
- **代号具体化/唯一化**：在满足需求的前提下选择参与计算数据量最小的表（先收集本次需求的数仓要求，再从候选表中选择最优）
- **SQL 组织**：筛选提前做、确定维表关联方式、派生指标先加工原子指标再加工限定词、limit 语法收尾
- **监控分析**：中心化记录用户查询日志，用于 DSL 分析、工程稳定性监控、数仓迭代与评测系统构建^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

### 3. Skill 机制 — 将 AI 拴在流程的笼子里

用 Skill 把 AI 限制在确定性流程内，三个机制：
- **门禁机制**：高歧义场景 AI 必须停下来向用户确认，不允许自己做决定；AI 只能拿到工程生成的 SQL，**不允许修改任何 SQL**，最大限度缩小 AI 发挥空间
- **可审计回溯**：关键流程节点过程留痕——查询到的实体、生成的 JSON、生成的 SQL 都保存下来，每一步可追溯，AI 也可自查
- **钩子机制**：借助 Claude Code / Qoder / Codex 等 Agent 平台的工程钩子设计，让 Agent 执行到某些动作时强制执行某段逻辑^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

## 信任如何建立

每一环都有明确的角色来确认：代号对齐环节由用户判断代号是不是自己想要的；代号数仓映射环节由数仓同学判断口径；SQL 引擎生成环节由引擎判断语法语义正确性。系统对用户的要求有所变高——用户需要自己对 AI 对齐的代号有判断力，但**不需要关心这个数怎么取**（不依赖"有人会审 SQL"）。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

## 组织与角色变革

- **数据开发**：从"写 SQL 的人"变为"建语义层的人"，核心工作从重复取数开发转向高价值、高复用的**语义资产建设**与数据质量管理
- **协作模式**：信任基建可参与构建其他信任系统——下游分析 Agent 可直接对接取数 Skill 产出分析结论，运营工作台可接入取数基建；类似印刷机重塑印刷行业，用户更专注于"要印什么"而非"怎么印"^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md]

## 深度分析

### 从"检验正确"到"消除错误"

原文给出一个关键数字：AI 可以生成语法完美但语义错误的 SQL，经过多种手段优化能到 70%–90% 的正确率——这在评测领域是好成绩，但在生产环境意味着永远有出错的可能，且用户无从判断某一次结果到底是对是错。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:50-56] 更致命的是，这套系统的用户是运营、产品、分析师，甚至是 AI Agent 本身，他们无法判断一段 SQL 是否语义正确——如果可靠性只能建立在"有人会审 SQL"的前提上，用户就没有被真正解放。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:56] 这正是 [[concepts/verifier-paradox|验证者悖论]] 的结构性困境：当验证者的能力不高于生成者时，事后检验是徒劳的。本文的解法因此不是"找一个更强的检查器"，而是**删除产生错误的步骤**——AI 不再写 SQL，幻觉便失去了载体。这是验证面从输出端回撤到生成机制的一次结构性转移，与 [[concepts/verifier-driven-development|验证器驱动开发]] 的思路同源。

### 代号层是一个双边契约

代号层不只是"字典"：对 AI 它是沟通共识层，解决代号对齐问题；对 SQL 引擎它是数仓映射层，让代号回归到物理表和字段。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:90-92] 一个抽象、两类消费者，这是它能成为"基础设施"而非普通术语表的原因。它的实现方式也包含对真实数仓的务实态度：数仓天然冗余存储，同一个代号往往有多份物理实现，代号层通过逻辑表把这些字段绑定到唯一代号上；真实数仓不按标准组织时，再用字段映射、自定义 CTE 等机制挂载到标准代号层。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:100-106] 这种"不要求世界标准化，而由标准方吸收非标准"的适配层设计，是语义治理能落地的前提——治理不是靠改造数仓完成，而是靠映射完成。

### 笼子：收缩自由度的漏斗设计

Skill 的三个机制可以读作对 AI 自由度的层层漏斗：门禁机制在高歧义场景强制 AI 停下来问用户、不允许自行决定；SQL 由工程生成后 AI 不允许修改；查询实体、JSON、SQL 全部留痕可审计；钩子在 Agent 平台层面对特定动作强制执行逻辑。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:126-140] 它们的共同点不是"让 AI 更聪明"，而是系统性缩小 AI 的发挥空间——信任不来自模型变强，而来自出错面变小。这也是本文与"更强模型路线"的根本分歧：文中明确分工是 AI 负责语义理解与用户对齐，工程负责确定性的输出。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:58-64]

### 信任链：判断力的再分配

原文对信任的定义是链条式的：每一环都有明确的角色负责判断——用户判断代号、数仓同学判断口径、引擎判断 SQL 的语法语义。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:68-78] 值得注意的是对用户的要求反而变高了：用户必须能判断 AI 对齐的代号是不是自己想要的，这反向要求代号层不能有二义性、含义必须唯一。信任基建并没有消灭人的判断，而是把判断从"会读 SQL"的专业动作下沉为"认对代号"的业务动作——把专业性转化为产品设计。

## 实践启示

**构建数据 Agent 时看两个指标，不是一个。** 90% 的 SQL 正确率在生产中不构成意义——真正要建设的是代号层的覆盖度与无二义性、SQL 引擎的确定性覆盖面；正确性应交给结构，而不是寄望于模型。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:50-56]

**语义治理从字典开始，而不是从改造数仓开始。** 先把最小可用的代号集合做唯一、做无歧义，再用字段映射、自定义 CTE 等适配机制把不标准的存量数仓挂上来；不要试图先建一个完美数仓再谈 AI 取数。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:76-106]

**把 AI 的输出接口压到尽量轻。** 让 AI 产出轻量 JSON 查询请求而非最终 SQL，接口越小、幻觉面越小、门禁越容易做——这是高风险 Agent 场景的通用模式。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:58-64]

**约束要成体系地落在平台机制上。** 门禁（必须停）、禁改（不许动）、留痕（可追溯）、钩子（强制执行）四层叠加，不能依赖提示词里的"请遵守"；主流 Agent 平台的钩子设计正是承载这类强制约束的工程设施。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:126-140]

**中心化日志是信任系统的飞轮起点。** SQL 引擎把查询请求忠实记录下来，供 DSL 分析、工程稳定性监控、评测系统构建与数仓迭代使用——没有留痕，就没有评测，也就没有持续改进的闭环。^[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026.md:122-124]

## 与既有实体的关系

本框架与库内数据 Agent / 语义层实体是**互补维度**而非重复：
- [[data-agent-product-design|火山 Data Agent]] 走"NL2SQL：LLM 生成 SQL + Schema 约束 + 校验"路线，本文是"AI 不写 SQL，只出 JSON 查询请求，确定性工程生成 SQL"——生成权从 LLM 转移至确定性规则
- [[anthropic-95pct-data-analysis-skill-stack-architecture|Anthropic 95% 数据分析]] 的语义层是"编译好的指标定义 + Agent 调用函数"，本文代号层是"统一业务代号的字典 + 挂载物理数仓"，且强调用户直接对齐代号而非通过函数调用
- [[insight-agent-trustworthy-reasoning-guandata|观远洞察 Agent]] 聚焦"可信推理链 + 决策闭环"，本文聚焦"取数环节的确定性生成"，二者覆盖链路不同段

→ [[raw/articles/trust-infrastructure-ai-data-retrieval-taobao-2026|原文存档]]
