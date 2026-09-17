---
title: "skill-mcp — 把 AI 技能当软件包管理（MCP 权限网关 + 只调度不执行的 Pipeline）"
created: 2026-06-30
updated: 2026-09-14
type: entity
tags:
  - skill-mcp
  - mcp
  - skill-management
  - versioning
  - dag-pipeline
  - security
  - prompt-injection
  - permission-gateway
  - shugex
  - open-source
sources:
  - raw/articles/skill-mcp-software-package-management-mcp-pipeline
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# skill-mcp — 把 AI 技能当软件包管理（MCP 权限网关 + 只调度不执行的 Pipeline）

> skill-mcp（GitHub: BeCrafter/skill-mcp）是一个开源项目，把 AI 技能当成有版本、有元数据、可权限控制的软件包来管理，再通过标准 MCP 协议暴露给任意 AI 客户端。定位：Cloud Skill File System & MCP Permission Gateway。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

## 核心理念：技能即软件包

每个技能是一个标准目录，含 manifest.json（name/version/entry/files）、SKILL.md、references/、templates/。skills 表维护 slug、version、category、tags、status、visibility、contentHash，带完整版本历史。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

## 三种部署场景

同一套代码，靠环境变量切换三种场景 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]:

| 场景 | 通信方式 | 存储 | 适合谁 |
|------|---------|------|--------|
| A 本地独立 | stdio | 本地 | 个人开发 |
| B 混合 | stdio（本地 MCP） | 远程共享存储 | 多客户端共享 |
| C 分布式 | HTTP/SSE | 本地或远程 | 生产、多并发 |

## 五个 MCP 工具

通过 MCP 协议暴露 5 个工具：skill_list、skill_view、skill_file、skill_pipeline、skill_feedback。启动时注入"必检"系统提示词，让 AI 在回答前先扫技能列表。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

## Pipeline：只调度、不执行（核心设计决策）

与传统 Workflow 引擎（LangGraph、CrewAI、Airflow、Temporal）不同，skill-mcp 的 Pipeline **只负责调度**，执行交给 AI Agent 自己。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

- DAGScheduler 用 Kahn 算法做拓扑排序，切成并行批次
- 自动检测循环依赖，快速失败
- 核心代码约 300 行
- YAML 定义，表达式借鉴 GitHub Actions

## 安全设计

导入环节三道关 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]:
1. **Prompt Injection 扫描** — 正则匹配常见注入模式
2. **路径遍历防护** — 拒绝 `..` 和绝对路径
3. **文件类型白名单** — 只放行文本格式

权限用标签交集判断：空 tags = 公开；有交集 = 受保护；无交集 = 受限。

## 与同类项目对比

| 特性 | Skill Pipeline | GitHub Actions | Airflow | Temporal |
|------|:-------------:|:--------------:|:-------:|:--------:|
| 目标用户 | AI Agent | CI/CD | 数据工程 | 分布式系统 |
| 执行者 | Agent 自己 | Runner | Worker | Worker |
| 复杂度 | 低 | 中 | 高 | 高 |

与 [[entities/hermes-skill-system|Hermes Skill 系统]] 的差异：
- Hermes 的技能是文件系统级的（SKILL.md + 目录），skill-mcp 是 MCP 协议级的（通过 MCP 暴露）
- Hermes 无版本管理，skill-mcp 有完整的版本历史 + rollback
- Hermes 无内置权限控制，skill-mcp 有标签交集权限
- skill-mcp 的 Pipeline 是"只调度不执行"，Hermes 无此概念

## 技术栈

@modelcontextprotocol/sdk ^1.12.1、better-sqlite3 + drizzle-orm、commander ^13、zod ^3.24、pino、prom-client、vitest。Node.js >= 22.0.0。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

## 当前状态

- 版本 0.1.0，首次正式发布 [1.0.0] - 2026-05-06
- 文档体系完整（中英双语 README、ARCHITECTURE、QUICK_START、三个 SCENARIO 等）
- CI：lint + tsc + test + coverage
- 测试覆盖 unit + integration + e2e 三层
- 主动列出未实现功能：condition、retry、执行状态持久化、Web UI

## 深度分析

### 「技能即软件包」是对技能蔓延的结构性回应

当技能数量从几十涨到几百上千，文件系统式管理最先崩掉的不是检索，而是「我现在装的到底是哪一版」。skill-mcp 用三条线对准三个不同的问题：manifest.json 里的 version 让人能回滚，skills 表中的 contentHash 让同一版本号下的静默篡改立刻暴露，status 字段则把「草稿—已发布—受限」拆成可编排的灰度阶段。三者的合力不是「更规范」，而是让技能库第一次具备软件发布流程的基本能力——可复现、可审计、可回退。纯文件系统的技能体系（SKILL.md + 目录）在个人尺度上足够优雅，但它缺少版本真值源，既回答不了「三个月前那个能跑的版本长什么样」，也无法按批次灰度放量，一旦进入多人协作就只能靠约定和记忆维系。[[entities/skill-version-management-semantic-versioning-practices-winty|Skill 版本管理五大原则]] 之所以值得和它并读，正是因为那些原则讲的大多是需要固化成机制、而非停留在纪律层面的约定。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

### 「只调度、不执行」是一次把复杂度让渡给 Agent 的赌注

把执行权从引擎手里拿走，砍掉的不只是代码量：没有 runner/worker 集群，就没有心跳、租约、任务重试队列和那一整套执行状态机，DAGScheduler 因此能收敛到约 300 行。代价是确定性被重新分配——同一份 YAML 交给不同的 Agent，执行路径和结果都可能不同，也正因如此 condition 与 retry 才只能被预留而无法真正落地。可观测性同样跟着搬家：引擎侧看不到步骤内部发生了什么，只能靠 skill_feedback 这类回流信号拼凑效果；失败恢复也从「引擎按策略重试」退化为「Agent 自行判断是否重来」。这套取舍适合把 Agent 本身当作执行主体的场景，也就是技能分发加轻量编排；一旦需要严格可重放的流水线，调度与执行分离反而会从优点变成负担。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

### 安全模型的形状：供应链闸门，而非运行时沙箱

三道关全部落在导入时刻——正则扫 prompt injection、拒绝 `..` 与绝对路径、只放行文本格式白名单——这是一道典型的供应链门禁：它试图保证「进入技能库的东西是干净的」，而不是「跑起来的技能不能干坏事」。这个边界放在 [[concepts/model-context-protocol-mcp|Model Context Protocol]] 分发的语境里是合理的，因为 MCP 只负责把技能内容送到客户端，真正的工具执行权在 Agent 侧，网关管不到。但也正因如此，它的防线是脆的：正则挡不住改写过的注入话术（[[concepts/prompt-injection-defense|Prompt 注入防御]] 的难点正在于模式识别永远滞后于变体），文本白名单挡不住危险指令被写进 markdown 再由 Agent 主动执行，标签交集权限只在 MCP 请求层生效，技能内容一旦被复制到本地就完全失效。理解这一点，才能把它当准入审查用，而不是当运行时隔离用。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

### 诚实的成熟度信号本身就是决策输入

版本 0.1.0、首次正式发布 [1.0.0] 落在 2026-05-06、主动列出未实现的 condition / retry / 执行状态持久化 / Web UI——这些细节放在一起，比任何 star 数都更能说明项目处在什么阶段。愿意公开未完成项，通常意味着维护者清楚边界在哪，也意味着你不用花两周才发现「原来 retry 是空的」。配套的 CI（lint + tsc + test + coverage）与 unit / integration / e2e 三层测试说明基础质量有兜底，约 300 行的核心调度代码也小到可以自己通读一遍。正确的用法是拿它做校准：先对照自己的需求勾选功能清单，再决定是「试用并容忍缺口」还是「只借设计思路」。对年轻基础设施，信任应当给设计，而不是给版本号。 ^[raw/articles/skill-mcp-software-package-management-mcp-pipeline.md]

## 实践启示

1. **上线前先锁版本**：用 version + contentHash 把技能钉住，再用 status 做灰度，别让「最新版」直接进生产；回滚能力必须在出事之前就演练过一次。
2. **把技能导入当供应链来管**：注入扫描、路径遍历防护、类型白名单只是及格线，关键技能仍需人工过一遍，并且始终记住它是准入审查而不是运行时沙箱。
3. **Agent 能自己干活时，优先选「只调度不执行」**：省掉 runner/worker 与执行状态机，代价是确定性与可观测性下降；适合轻量编排，不适合需要严格重放的任务。
4. **权限标签要在第一次发布前定好**：标签交集语义一旦上线就很难改，「空 tags = 公开」这个默认值尤其危险，先设计好分类再发布技能。
5. **用「未实现清单」对齐自己的需求**：缺少 condition、retry、执行状态持久化和 Web UI 是否构成阻塞，取决于你是要跑长期 DAG 还是只做顺序触发，别默认它什么都能做。
6. **保留一条文件系统退路**：技能内容一旦离开 MCP 网关，权限就失效；本地留一份可读副本（例如 SKILL.md 目录）能让工具升级或网关故障时不至于全线停摆。

## 资源

- GitHub：https://github.com/BeCrafter/skill-mcp

→ [[raw/articles/skill-mcp-software-package-management-mcp-pipeline|原文存档]]
→ [[entities/hermes-skill-system|Hermes Skill 系统]]
