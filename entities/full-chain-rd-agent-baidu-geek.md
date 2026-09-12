---
title: "全链路研发智能体——从「体感能用」到「实际可用」的工程实践"
created: 2026-07-11
updated: 2026-09-10
type: entity
tags: [harness-engineering, agent, ai-coding, baidu]
source_url: ""
sources: [raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践, raw/articles/builderagent-ai-native-organization-baidu-2026-08-05, raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 全链路研发智能体——从「体感能用」到「实际可用」的工程实践

> **Source**：百度Geek说，发布于 2026-06-24。本文是对 百度Geek说 关于 全链路研发智能体 实践的系统整理。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

## 核心洞察

百度Geek说团队从真实交付复盘出发，提出 Harness 全链路研发智能体架构：将大模型放进可控研发流程，覆盖需求分析、接口设计、代码生成、自动CR、单测、冒烟验证、环境部署、问题排查的完整闭环，附带需求可执行性检查、状态机与质量门禁设计、失败回流机制等工程实践。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]


## 详细内容

## 全链路研发智能体 ——从"体感能用"到"实际可用"的工程实践

。

点击蓝字，关注我们

作者 |  阿拉丁RD&QA团队

导读  introduction  AI Coding 已改变研发方式，但单点代码生成只占研发工时 10-32%，整体提效有限。本文从我们团队在项目的真实交付复盘出发，提出 Harness 全链路研发智能体：把大模型放进一套可控的研发流程，将需求分析、接口设计、代码生成、自动 CR、单测、冒烟验证、环境部署、问题排查串成带反馈控制的闭环，让 AI 从"会写代码"升级为"能完成可验证交付"。本文系统阐述其需求可执行性检查、状态机与质量门禁设计、失败回流机制、复杂度感知编排、RD/QA 协同模式，并以行业研究（METR RCT、Google DORA、SWE-bench 等）做交叉印证。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

_ 全文 6746 字，预计阅读时间 7 分钟  _ GEEK TALK^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]


01

问题定义：AI Coding 为什么"用起来不错，但没省多少时间"？^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]


** 1.1 我们自己的复盘：coding 快了，但需求没更快交付  **^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]


2026 年 4 月起，我们团队在项目里全^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]


^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

## 关键特征

- 来源为 百度Geek说 的技术实践分享^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]
- 涵盖 Harness Engineering / Agent 框架的设计与工程落地^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]
- 包含实际代码架构与工程经验沉淀^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

## 深度分析

### 瓶颈在系统不在模型的核心论断

百度 Geek 说团队的核心发现——"瓶颈在系统不在模型"——与 METR RCT、Google DORA 和 SWE-bench 等多组独立研究相互印证。SWE-bench 在 30 个月内从 2% 涨到 80%+，说明模型解题能力已经过剩；而 METR 研究发现裸用模型甚至可能减速。真正决定实际产出的是模型外的 scaffolding——即一套可控的研发流程。这一论断是 Harness 工程范式的重要理论基础：AI Coding 的关键不是让模型更快地写代码，而是把清晰的需求、可执行的流程、可验证的质量门禁串成全链路。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

### 状态机与质量门禁的闭环设计

全链路研发智能体的核心工程创新是状态机驱动的闭环执行系统。每个阶段都有明确的输入、输出、成功条件、失败条件和回流目标。任一门禁失败时，系统不是停下来等人，而是保存失败上下文→选择回流阶段→在轮次上限内修复→重跑验证→记录全过程。这种"失败不是终点，而是修复输入"的设计，让 AI 从线性生成变成了带反馈控制的工程系统。复杂度感知编排确保简单需求走轻链路，复杂需求走重链路，避免了"一刀切"的效率损失。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

### RD/QA 对抗式协作架构

行业实践中一个隐蔽的质量风险是"AI 自己写、自己验"——当开发和测试共享同一份上下文时，模型容易跳步、伪造 Mock 输出，甚至以 bug 验证 bug。百度团队的解法是将测试拆为独立的 QA SubAgent，采用"隔离式架构"（各自维护专属上下文）和"对抗式校验"（双模型开发与验证，QA Agent 标准化验收为唯一放行依据）。这是对 AI 测试可信度问题的系统性回应，其价值在于承认了"共享上下文"和"独立判定"之间的根本矛盾。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

### 全链路覆盖的提效空间量化

团队的数据复盘显示，编码环节仅占研发工时的 10-32%，即使提效 55%，整体也只节省 5-16%。真正的提效空间分布在需求分析、验证、环境部署、线上排查等"编码外"环节。基于这一量化理解，全链路智能体将需求可执行性检查、接口设计、代码生成、自动 CR、单测、冒烟验证、环境部署、问题排查全部纳入闭环。与 Antenna、Stripe、GitHub 等业界研究的数据交叉印证，这一量化框架具有较强的通用参考价值。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

### 环境部署与运维纳入 AI 流程

百度团队将环境部署从"人工附加步骤"改造为全链路的一环：RD 或无背景同学不需要从零摸索环境搭建，代码生成和基础验证后可直接进入测试环境部署。同时，非 Coding 高频事务（日志排查、配置变更、答疑）通过群聊助手远端触发，降低了触达门槛。这标志着 AI Coding 的边界从"写代码"扩展到了"完整交付"，将运维经验从口头支持转化为流程中的可执行能力。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]

## 实践启示

1. **量化提效空间，避免盲区**：先测量编码环节在总工时中的实际占比（10-32%），再决定优化方向。只优化编码而忽略需求、验证、环境等环节，整体提效有限。
2. **质量门禁必须有失败回流机制**：不要让 AI 在失败时停下来等人。状态机驱动的回流（保存上下文→选择回流阶段→修复→重跑）是实现自主闭环的关键工程模式。
3. **测试必须独立于开发**：共享上下文导致"自己写、自己验"的认知偏差。隔离 QA 上下文、采用双模型校验是保证测试可信度的有效手段。
4. **复杂度感知编排避免一刀切**：简单改文案和复杂跨服务需求走不同链路，避免用重流程拖慢轻任务、用轻流程漏掉重验证。
5. **把非编码环节纳入 AI 覆盖范围**：环境部署、线上排查、配置变更这些"不写代码但高频"的工作是真正的提效空间，通过远端触发和工具化将其纳入流程。

## Workflow 型 Harness 的设计细节：阶段流水线 + Manager 协议纪律 + 状态可恢复（百度Geek说，李忠泽，2026-09-10）

百度Geek说 2026-09-10 发表的另一套自研方案（作者李忠泽，全文 8,019 字）——与本实体前述「阿拉丁团队全链路智能体」同属百度 Harness 工程但**设计路线不同**：前者是带状态机与质量门禁的研发闭环，本篇是**「Workflow 为主 + 阶段级 Multi-Agent + 保留 Human-in-the-loop」的固定阶段流水线**，明确回答「什么样的 Harness 应该做成固定流程」以及「每个设计决策的代价是什么」。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### 为什么需要固定流水线：四类失效模式

一次性把真实需求丢给长上下文 Agent 会遇到：①**上下文膨胀与结果漂移**（需求/方案/代码/测试/日志全堆在一个上下文里，越到后面越容易忘了前面说好的约定）；②**跳过必要的工程步骤**（可能直接写代码，遗漏需求澄清、方案评审或测试设计）；③**过程不可恢复**（会话被压缩、重启或中断后很难判断"现在在哪一步、正在等什么、哪些任务已完成"）；④**自动化过程缺少人工门禁**（没有门禁与确认点，错误方向可能一路运行到最后，返工消耗更多人力）。作者的判断是不要「再造一个很聪明的 Agent」，而是**给 AI 编码套一副工程框架**。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### 11 阶段固定生命周期与自动执行区

`init → specify → plan → tasks → test-plan → plan-review → implement → code-quality → e2e-run → human-acceptance → commit-push → archive`。其中 **`implement → code-quality → e2e-run` 是自动执行区**，其余大多数阶段在完成后停下等用户确认；`human-acceptance` 与 `archive` **没有独立 Sub-Agent**，由 Manager 负责呈现、验收与收尾。**「裁判员」与「运动员」分开**：plan 与 plan-review 由两个 Sub-Agent 承担（一个做技术设计、另一个评估），参考**对抗性**思想——凡涉及审查的环节（技术设计、测试）都要**挑战设计中的假设**，而不是确认正常流程能跑通。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### CLAUDE.md 的双重角色与「知识层插件」

`CLAUDE.md` 承担两件事：**①框架入口**——它是整条链上**唯一会被自动读到的文件**，因此本身不写任何流程逻辑，只点明阶段流程、指向三份规则文件（流转表 `state-graph.json` / 通信协议 `protocol.md` / 状态字段说明 `state-schema.md`），以及最关键的一句「主会话必须始终遵循 `.harness/rules/manager.md`」；**②项目知识库**——Harness 本体通用而项目特定（有哪些代码库、怎么起服务、连哪个测试库、提交要建什么卡、代码库之间谁依赖谁），这些「项目事实」全部登记在知识库索引里（环境表/研发流程/依赖顺序）。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

关键约定：**只有 Manager 读 CLAUDE.md，所有 Sub-Agent 一律不自己读**。Manager 按相关性挑片段，通过 `input` 的**具名字段**下发——环境表内容装进 `environment_knowledge` 给 e2e-run、建卡空间装进 `icafe_space` 给 commit-push、文档链接装进 `knowledge_refs` 给 plan。三个收益：①不让每个 Sub-Agent 把整个知识库吞进上下文（只拿与自己这一步有关的那几行）；②**Sub-Agent 保持通用**——只认 input 里的字段、不认某个项目的具体布局，换项目只需换一份 CLAUDE.md，`agents/` 与 `rules/` 一行都不用动；③**知识变更只改一处**（新增代码库在表里加一行，不用改十个 Agent 文件）。另有被实测坑出来的原则：**不假设、缺就问**（不同代码库配置位置与形态可能完全不同，如某些服务在 `conf/servicer/*.toml` 但那不是通用规则）。作者由此给出定位：**`CLAUDE.md` 就是「知识层插件」——`rules/` 是骨架、`agents/` 是阶段，都是通用件**。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### Manager 铁律：只读协议，不读正文

Manager 一人分饰三职——**调度**（决定这一轮派哪个阶段、往前走还是往回退）、**状态管理**（唯一有权写 workflow-state 的人，负责需求在「进行中/挂起/已归档」目录间搬移）、**对话输出**（Sub-Agent 只跟它用结构化协议对话，用户只跟它用自然语言对话，两边的话都由它转）。它的铁律是：**只认 Sub-Agent 回传的结构化协议，不读任何产物的正文（spec、方案、任务、测试、报告），也不读业务代码**——只从协议取「结论」（通过/打回/要你确认），从状态文件取「进度」。**因为 Manager 从不把产物正文和代码吞进自己的上下文，主会话才能在一个需求走完十几个阶段、来回折腾很多轮之后仍然轻、稳、不飘**；用户问「刚才那条结论为什么这么判」，它不会自己去翻代码，而是把问题**重新丢回给当初给出结论的子代理**去解释（这条后来在规则里收敛掉了 Manager 越界答疑）。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### 双向协议：请求侧三个字段 + 响应侧 status/output

Manager 与每个阶段 Sub-Agent 之间**只用同一个协议对象**（派发时 Manager 填请求侧、返回时 Sub-Agent 填响应侧）：^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

- **请求侧**：`state`（派给哪个阶段）、`user_message`（**用户原话逐字透传**）、`input`（本阶段所需上下文，**每个阶段的专属 Key 由 `manager.md` 的调度规则定义**——这正是「Sub-Agent 保持通用」的关键）；`protocol.md`、各阶段模板、业务代码都**由 Sub-Agent 自己读**，不由 Manager 传。另有三个**通用可选 Key**，只在特定情形出现（如 `skip_gates`）。
- **响应侧**只有三种 `status`：**`ok`**（阶段做完，等用户确认后前进）、**`needs_input`**（没做完，需人介入——多轮澄清、环境卡住、或要回退到上游补东西）、**`fail`**（阶段失败，由 `output.next_state` 给出建议打回目标，如 e2e-run 不过 → `implement`）。`output` 有**四个通用字段**（几乎每阶段都有）+ 各阶段专属小字段；响应侧信息包装在一个 `control-result` 里，供 Manager 解析。

### 状态机与「置位时机」精算（冷启动恢复正确性）

流程状态**不依赖聊天记忆**，落在每个需求目录的 `workflow-state.json`：`current_state`（当前阶段）、`pending`（当前等待点——等待前进确认/澄清/回退确认/人工验收/推送确认/CR 评审）、`states`（阶段历史、状态与产物路径）、`review_findings`（门禁发现的问题及其应跳转的阶段）、loop 计数/涉及仓库/验收服务。设计思想是**不应让 Agent 完全依赖自身上下文记忆，要有 Handoff 机制**——「你用 Claude Code 做到某一阶段，接下来由 Codex 接手，如果 Codex 不具备 Claude Code 的记忆，它并不知道当前执行到哪里」；同一 Coding Agent 内部上下文压缩、Memory 清空或新开会话后同样可以接手。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

一个精细但关键的细节：**等待用户确认期间，当前阶段一直是「进行中」，终态只在真正转移的那一次写入里才落到状态及文件**——「已完成」严格意味着「用户已经拍板、流程已经离开它」，绝不在等确认时提前写。目的是**让冷启动恢复时状态永远正确：任何时刻至多一个阶段处于「进行中」，且它必然等于 `current_state`**。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### 回退统一机制与流水线内的 Loop Engineering

**所有回退都走同一套机制**，不论触发者是谁：①用户主动推翻上游（做到 plan 才想起需求说漏）；②下游 Sub-Agent 报出上游缺口（test-plan 发现 plan 没定义接口路由、写不出可执行用例）；③门禁打回（plan-review 审出问题，指向**最早出问题的那个阶段**）；④自动执行区的测试打回（code-quality / e2e-run 不过 → 回 implement）。硬约束：**回退的目标阶段必须在流转表 `order` 里位于当前之前，且在这个需求的历史里已经真正完成过**——防止「回退」变成乱跳。校验通过且用户确认后，Manager **在同一次写入里**完成状态转移（当前阶段落终态、目标阶段由「已完成」改回「进行中」、`current_state` 指向目标），并把回退原因作为 `rollback_reason` 放进目标 Sub-Agent 的 input，让它知道这次回来要补什么。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

自动执行区可连续完成「**implement 写代码 → code-quality 静态审查 → e2e-run 跑系统测试 → implement 定向修复**」（参考 Codex `/goal` 与 Loop Engineering 概念），但**自动化不是无限循环**：每条自动修复循环都设**次数上限**，达阈值仍未过就停下把问题呈现给用户，让用户三选一——继续修 / 直接放行 / 要阶段 Sub-Agent 解释。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### 多需求隔离：git worktree 工作空间

为兼容多需求并行，**Implement 阶段之前** Manager 依据 tasks 产出的仓库集合，为每个业务仓建立同名特性分支与独立 `git worktree`；实现、静态审查与 e2e **复用同一 worktree**，**修复轮不重建，归档时才清理**。根目录下只需有全部代码库（各自及时同步 master），每个需求据最新代码新建 worktree，并有自己的 `workflow-state.json` 与阶段产物。三个直接收益：①主工作区不被临时产物污染；②**代码审查看到的是同一份改动，而不是每轮重新推导的快照**；③中断或回退后可继续已有任务，**已勾选的 task 不需要重复执行**。目录上区分 `.harness/active/`（正在做的需求，**全局至多一个**）、`process/`（已开始、挂起）、`archived/`（已完成归档）、`worktrees/`；`.harness/rules/` 是规则与模板层（含 `manager.md` / `protocol.md` / `state-schema.md` / `state-graph.json` / 七份阶段产物模板），`.claude/agents/` 是阶段层（10 个 agent 文件，加/删/改即改流程），`.claude/skills/` 是可插拔能力层（连测试库/连缓存/建 iCafe 卡/推 iCode CR/取 ugate token/Playwright 驱动浏览器）。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### 多仓提交依赖链：commit-push 两段式

`commit-push` 分两段：**`prepare`**（整理改动、建立 iCafe 卡、生成提交确认表）与 **`execute`**（用户确认后才真正 commit、push 和创建 CR）——CR 评审由人工完成，Harness 只根据评审结果继续下一层或归档，**不代替评审者合入主分支**。归档阶段负责停掉验收服务、迁移需求目录、删除 worktree 和已合入分支。针对多仓的公共库依赖，作者做了一个特别设计：**代码库依赖关系写进 `CLAUDE.md`，Manager 带给 commit-push**，由它**先提交依赖代码库的 CR → 等待用户反馈是否合入**（目前 iCode CR 还需人工 +2，若有 Skill 能直接拉取评分即可改自动化）→ 合入后**派发 `implement` 去修改其他代码库的依赖关系**（如 `go get xxx@latest`）→ 再继续提交其他仓的 CR。此处给 `implement` 加了一个可选 Key **`skip_gates`**，使 Loop Engineering **只执行而跳过测试**（存在自测需求或小改动、不需要复杂 e2e 的场景）。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### 「中间极简、外围皆插件」：与 DeepSeek Harness 的对照

作者以 DeepSeek Harness（2026-08 开源、MIT、命令 `dsh`）的「everything is a plugin」为参照——模型/工具/技能/会话/沙箱/存储/循环/调度/UI 全是可替换插件，**连驱动每一轮对话的 agent loop 本身也能从配置里换掉**，哲学是「从一个已经能用的完整 Agent 出发去替换零件，而不是从一堆零件出发去攒一个 Agent」，底层 append-only 会话日志支撑 resume/fork/replay。作者据此把自研框架的定位说清：**每个阶段都是插件**（一个阶段就是一个 markdown 定义的 Sub-Agent 文件，想加一道「安全审计」就加一个 agent 文件并在流程表排上，想去掉就删）；**能力（skill）是插件**（查库/连缓存/建卡/推 CR/Playwright 按名字挂上）；**规则与知识是配置**（通信协议、流程次序、状态字段、阶段模板、项目知识库全是框架外文件）；**进度也是可恢复的**（状态全落盘与 append-only 会话日志是同一诉求——让流程能被恢复、被接续）。差异在于：dsh 是通用 Agent 运行时（连 agent loop 都能换），这套是**专为「编码生命周期」定制、带固定阶段流水线的 Harness**；但**「Manager 极简、阶段技能皆可换」这一点是共通的**。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

### 冒烟测试复盘与八个未解决问题

作者按真实需求从头到尾跑通整条流水线，特意覆盖正常前进、多轮澄清、中途改需求回退、门禁打回后逐级复审、自动执行区打回-修复循环、冷启动恢复、提交建 CR、最后归档等路径。**如实记录的八个未解问题**（本实体的姊妹篇中亦无此等失败面清单）：^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md]

1. **Sub-Agent 每轮重新加载上下文**——需要多轮澄清的阶段（specify、plan）每轮都得重新加载默认要读的那批东西（Claude Code 在提问类话术下会自动复用上一 Sub-Agent 会话 ID，但当前调度还没做到这层）。
2. **plan 阶段没有搜索边界、容易超时**——实测跑到过一小时；现在的规则只是让它「自己去 Grep/Read」，没划边界。**需要给它补知识库、用渐进式索引**，明确本次允许读哪些仓、最多翻多少文件、调用链最多追几层、多久没收敛就该停、范围太大时该反过来问用户什么。
3. **产物与状态不是原子完成的**——实测 test-plan.md 已在且标注 Ready，重入时 Manager 很难判断文件完整不完整、Sub-Agent 是真做完了还是结果丢了、该不该覆盖、能否直接往下走。**需要把「写产物」与「改状态」做成原子的、可校验的一步**，重入时才能可靠判断阶段是否完成。
4. **Manager 越界替用户答疑**——审查类阶段把打回结论与理由抛给用户后，用户一有疑问 Manager 有时会自己跑去读代码解答，破了「不读正文/代码」的规矩（已在规则里收敛）。
5. **前端设计需要更多能力兜底**——第一次 AI 设计的界面过度复杂，功能实现了但交互与样式不够简洁，可能需要前端/设计介入并提供前端实现或设计规范类 Skill。
6. **云端 memory 的自进化还没做**——理想中的 Harness Engineering 应是每跑完一次流程就沉淀实现问题与测试 bug；现有说法是面对 bug「**第一优先级不该是让 AI 去想怎么修这个，而是怎么让下次不再产生**」；现在只往本地 memory 存，换机器就丢，**需要云端共享 memory 让经验跨机器跨会话攒起来**。
7. **工作目录结构对团队使用不友好**——当前放在一个 git 仓库里但实际使用不是这么用；见过其他团队用 `git submodule`；工作空间根**不能是一个 .git 目录**，后续应维护成团队友好的 WorkSpace。
8. **未建立完善的线上运行可观测性与监控**——框架产物只有两类（各阶段产物与报告 / 状态机快照），能做到「单个需求怎么走完」，但**无法观测某个阶段是否执行正确、调用链路与读取内容是否符合预期**；如同传统线上业务需要监控、日志、报警，Agent Workflow Framework 同样需要（可自定义 Hook 或 Coding Agent 内部 Hook 采集）。

**与库内百度实体族的关系**：本页上方是阿拉丁团队的「全链路研发智能体」（状态机+质量门禁+失败回流+复杂度感知编排）与数字人团队的「AI Native 组织」（组织与流程维度），本节是第三套自研路线（Workflow 型固定阶段流水线 + Manager 协议纪律）；其 `code-quality` / `plan-review` 阶段承担的门禁职责与 [[entities/baidu-ai-coding-quality-gates|百度 AI Coding 质量关卡实践]]（把验证流程左移进 Agent 开发过程：前置审查/运行时验证/视觉验证 + 拦截经验沉淀为 Skill/Rules）互为工程侧实现与质量侧方法论的对照——一是「谁来卡」，一是「怎么卡」。^[raw/articles/agentic-harness-workflow-ai-coding-engineering-li-zhongze-baidu-2026-09-10.md, raw/articles/baidu-agent-engineering-quality-gates.md]

## 相关实体

[[entities/claude-code-large-codebase-harness-configuration]]、[[entities/超级ai背后的秘密武器agent-harness深度解析]]、[[entities/harness-engineering]]、[[entities/claude-code-founder-harness-100-lines]]、[[entities/tencent-knowledge-harness-practice]]

## 原文存档

→ [[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践|原文存档（阿拉丁RD&QA团队）]]
→ [[raw/articles/builderagent-ai-native-organization-baidu-2026-08-05|原文存档（数字人BuilderAgent团队，Supplementary）]]

## BuilderAgent 与 AI Native 组织（数字人团队，2026-08-05 Supplementary）

百度另一团队（数字人BuilderAgent团队）的姊妹篇实践，从**组织与流程维度**扩展了全链路智能体：^[raw/articles/builderagent-ai-native-organization-baidu-2026-08-05.md]

### AI 提效分级框架 L1/L2/L3

三级的本质区别是 **Context 归属**（而非能力高低）：选择依据是这个需求的完成依赖多少"人类独有、AI 当前不具备"的 context。L3/部分 L2 需求 Builder 个人可完成全生命周期；L1 与复杂 L2 需多 Builder 共担。^[raw/articles/builderagent-ai-native-organization-baidu-2026-08-05.md]

### 流动效率（Flow Efficiency）度量

核心度量 = 真正干活时间 ÷ 总交付周期。数据印证：单点 AI 优化了"一小段里的一小部分"，撬不动占 80%+ 的流程性等待与交接——与阿拉丁团队"编码仅占 10-32%"的量化结论互相印证。^[raw/articles/builderagent-ai-native-organization-baidu-2026-08-05.md]

### AI Native 组织：打破职能边界

模糊 PM/UE/RD/QA/OP 边界，以 Builder 角色对同一业务结果共同负责，共享同一份 Context 与交付物。PM 不再"写完 PRD 就甩给 RD"，QA 验收标准前移进 Spec。AI Native 组织不取消专业角色，而是隐性知识显性化 + 专家协同。^[raw/articles/builderagent-ai-native-organization-baidu-2026-08-05.md]

### 缺陷修复回流 Spec 闭环

验收或上线后发现问题不就地打补丁（那样验收标准不更新、同类缺陷反复出现），而是：触发（QA 验收失败/数据回收/智能运维发现）→ 缺陷归因（Spec/方案/实现哪一层）→ **回流 Spec**（缺陷转化为新验收标准写回 Spec）→ 定级修复（L2/L3，根因在需求则回 Spec 层重做）→ QA 回归（复用原用例+新增用例）→ 沉淀规则/用例库。^[raw/articles/builderagent-ai-native-organization-baidu-2026-08-05.md]

### 知识/Skill 自动沉淀机制

交付完成或缺陷修复完成自动触发沉淀（非人工事后补写）：抽取专家补位 diff、Spec 增量变更、新增缺陷用例、方案决策 → AI 归类生成/增量更新 Skill（可复用 SOP）、验收模板、知识 → 下一轮 Spec 澄清与 workspace 拉起时自动加载。这是"提效可复制、不依赖个别熟手"的机制来源。^[raw/articles/builderagent-ai-native-organization-baidu-2026-08-05.md]

### 五个关键构件

Spec 工具（需求澄清一次做对）、沙箱与 workspace（业务维度独立开发部署环境）、Builder 协同（交接→共担）、QA 自动化协同（Spec 定标准→QA 自动验→结果回流 Spec）、智能运维协同（持续感知→主动分析→分级处置）。^[raw/articles/builderagent-ai-native-organization-baidu-2026-08-05.md]

## 补充要点（同源重复页合并）

- **缺陷越晚发现越贵**：NIST 显示生产环境修复成本是开发阶段的 30 倍，IBM 给出设计→维护成本比 1x:6.5x:15x:100x——这是闭环 Loop 在每个阶段设质量门禁、失败即回流的经济学依据。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]
- **闭环 Loop 的工程化约束**：记录失败证据、最小修复、重跑失败门禁、影响公共逻辑补跑复审、最多 3 轮止损、过程可复查。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]
- **"实际可用"的三条可检验标准**：端到端（需求卡片到提测材料无人工断点）、可验证（保留 CR 证据）、可复制（无项目背景的同学也能独立交付）。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]
- **典型教训**：新增透传字段的需求在 AI 辅助下单测全部通过，但真实调用接口返回不符合预期——"生成完成 ≠ 功能正确"，验证必须扩展到真实接口冒烟和历史功能回归两层；一次无背景介入的需求开发从 1 人日压缩到半人日，但环境搭建和端到端验证反而花了半天。^[raw/articles/全链路研发智能体-从体感能用到实际可用的工程实践.md]
