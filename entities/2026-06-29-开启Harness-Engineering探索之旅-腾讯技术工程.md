---
title: "开启Harness Engineering探索之旅 腾讯技术工程"
created: 2026-08-01
updated: 2026-11-07
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md|原文存档]]

## 摘要

腾讯团队复盘其 Harness Engineering 实践：AI 写代码占比一路走高但版本节奏提效不成正比，出码率与提效之间"裂开一道缝"，根因是研发瓶颈从来不在"写"而在理解、对齐、验证、沉淀等非编码环节，需要为 Agent 搭建可执行、可约束、可验证、可反馈的工程环境 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。整套体系是"2 条轨道 + 1 个长期记忆"：研发端到端交付落在 SpecWorker 上（协议层定义 AI 每步输入输出契约、管线层标准化 P1 需求到 P6 归档的 6+1 阶段、纪律层硬编码 TDD/Debug/Verify/Review/Evaluate 五道门禁且评分 95 分以下打回重做），线上运营轨道用 7 步处理告警闭环，知识库作为 AI 的长期记忆分项目级 specs/ 与变更级 knowledge-spec/ 两套并通过 index.md 互通 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。文章给出了大量具体机制：P1 阶段 TAPD 拉取 + AC 可测 + test-cases 同源、P3 的 UI 像素+SSIM 双 95% 五轮自愈校准、P4 后端 API 测试失败时自动拉 CLS 日志查 MySQL 看 Redis 的诊断 SubAgent、P5 的 SQL 变更强制人工确认、P6 的 changes-sync/knowledge-sync/specs-generator 归档三件套 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。文末总结 4 条工程原则（追求确定性而非自由发挥、上下文控制、Token 成本优化、确定性过程用脚本实现并优先 SKILL 而非 MCP）与 4 个典型问题（指令遵循、需求歧义、设计稿还原、产物可靠性）的标准应对，并坦诚列出了六件还在路上的事 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。

## 关键要点

- 概念脉络：Prompt Engineering（2022-2024，关心单次调用）→ Context Engineering（2025，关心每步喂什么）→ Harness Engineering（2026，关心整个任务）；Mitchell Hashimoto 2026 年 2 月命名，定义是"每当你发现 Agent 犯了一个错，就工程化一个方案让它永远不再犯"
- 协议层四件事：每步产出什么格式、文档用标准模板、写完机器自动校验、变更只记增量保留完整历史；核心立场是"人与人协作靠默契，人与 AI 协作必须靠契约"
- 设计阶段：design.md 是契约不是说明——接口签名/数据模型写死成 Markdown 表格 + Mermaid 图，D-x 改动点列表标注"文件:行号 @ 函数名"作为 P3 的工单池，前端 design.md 顶部强制 sandbox_mode 字段
- 可监测性三维度与关键踩坑：可追踪（.phase-metrics.jsonl 记录每阶段 token、耗时、代码改动量并按费率折算 estimated_cost_usd）、可回溯（按失败类型走固定 SOP 检索路径）、可度量（双层 Hook 汇聚 hook_events 表）；踩过的坑是 SubAgent 上下文独立计费——code-reviewer 最初设计为读全文件做契约 review，看似甩了负担实则另起一份账，改为优先读 git diff 并把 token 双层结算（父 Skill 与 SubAgent 独立计费）定为硬规则
- 待解决清单：评分机制与下游真实消耗未耦合、知识库自动治理（老化淘汰）缺失、运营轨道跨项目 SOP 复用在试、老项目"水下的冰山"式隐性约束适配、AI 生成测试用例质量评估


## 深度分析

### 1. 提效裂缝的本质：AI 只压缩了附属复杂度，瓶颈结构性转移到非编码环节

用 Brooks 的两层复杂度框架看，AI 砍掉的恰好是附属复杂度（语法、工具、平台的"翻译成本"），本质复杂度（概念构造、需求顺应）一分未减——甚至因为代码产出更多，下游对齐、review、维护反而更重。局部把"写"踩到十倍速，链路总时长由没被加速的理解、对齐、验证、沉淀决定，瓶颈只是从"写"挪到"收"。这解释了为什么出码率是局部指标而非链路指标；而腾讯团队用 .phase-metrics.jsonl 把每阶段 token、耗时、改动量按费率折算落盘，本质是把这道"裂缝"从体感变成可测量的治理对象 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。

### 2. 契约先于智能：把"默契"全面替换为"机器可读协议"是第一性原理

全篇最一致的立场是"人与人协作靠默契，人与 AI 协作必须靠契约"——AI 没有常识兜底，每个"自以为说清楚了"的场景都会变成下游偏差。具体落法环环相扣：AC 必须 WHEN→THEN 可测（消灭模糊描述）、P1 禁止 AI 复述需求只允许引用 TAPD 原文（消灭转述失真）、design.md 写死成 Markdown 表格 + Mermaid 契约且 D-x 改动点标注到「文件:行号 @ 函数名」（让 code-reviewer 能机读比对而非凭感觉评）。隐含的取舍是：契约粒度换来了可验证性，代价是设计文档维护成本上升——这套体系实际上把传统 review 的隐性人力成本显式化并前置到 P2。

### 3. SubAgent 双层计费揭示：隔离上下文 ≠ 节省上下文

code-reviewer 最初设计为"读全文件做契约 review"，看似把读取负担甩给了子代理，实则 SubAgent 上下文独立计费——主 Agent 无感知，成本照样产生，直到核对 token 账单才暴露。改为优先读 git diff、仅在上文不足时读全文件，并把父 Skill 与 SubAgent 独立结算定为硬规则后才真正收敛。这是对 [[concepts/subagent-spawning-pattern]] 的重要修正：SubAgent 是上下文隔离的手段，不是节省的手段，隔离收益必须先抵掉复制上下文的成本再评估。同理，"SKILL 优于 MCP"的判断（MCP 固定占用上下文、工具不可灵活选配）指向同一个经济学：上下文是稀缺资源，[[concepts/context-engineering]] 的核心是信噪比而非窗口大小。

### 4. 确定性/概率性的分工边界决定体系可维护性

四条工程原则中最容易被低估的是第四条：确定性强、可重复的流程沉淀为脚本，智能只花在真正概率性的环节。体现在三处——可回溯的检索 SOP 写死（API 失败 → trace-id → CLS 日志 → MySQL → Redis，"不让 Agent 自由发挥"）、skill.md scripts/ 承载确定性动作、两级查找禁止全局通配（index.md → spec 两跳命中）。这与"grep 是否够用"之争同构：可预测的检索路径不需要智能介入。[[concepts/harness-engineering]] 的适用边界由此清晰——它治理的是概率模型的输出环境，不是用概率模型做确定性工作。

### 5. 信任被工程化为"证据先于断言"，但评分体系仍有自我指认的盲区

执行主体从人变成 AI 后"追责到人"失效，于是每步强制留下机器可读证据：.phase-metrics.jsonl（流水账）、evaluation.md（独立 scorer 评分 ≥95 才放行、最多 3 轮整改）、hook_events 表（双层 Hook 汇聚全局）。这套机制针对的正是 AI 的四种"偷懒模式"——跳过测试、猜修复、未验证自称完成、自评偏高。但作者自己承认关键盲区：scorer 只看本阶段产物，"P2 得 90 分但 P3 翻车"的下游反哺尚未落到分数上。也就是说，门禁防住了 Agent 骗自己，还没防住"阶段局部最优、链路次优"——与开头的提效裂缝形成同构呼应：度量体系本身也有它度量不到的缝。

## 实践启示

1. **照抄"偷懒模式→门禁"映射表，比泛泛强调纪律有效。** 跳过测试→TDD 纪律强制先写测试；遇 bug 猜着改→Debug 纪律强制根因分析；没验证就说完成→Verify 纪律要求运行证据；自评偏高→独立 SubAgent 打分。每道门禁拦截一种具体偷懒模式，且硬编码进管线（触发否决直接阻断），而非写在提示词里"建议遵守"。

2. **需求口径在源头钉死：禁止 AI 复述，只允许引用；测试用例与需求同源。** TAPD 原文做 requirements.md 底稿 + AC 拆成 WHEN→THEN 且不可测必改写 + test-cases.md 与 requirements.md 共用同一份 AC 列表。下游 P4 直接拿 test-cases 跑，不再"重新理解一遍需求"——这一招同时消解了需求歧义和测试漂移两个问题。

3. **审计 token 必须按父 Skill/SubAgent 分层结算，SubAgent 统一"优先读 diff"。** 在把 SubAgent 当成本优化手段之前先算账：独立计费意味着每次全文件读取都是一份新开销。配套动作：重要规则固化进 rules 防长会话压缩、无关联任务开新 session、SKILL 按需读取文件。

4. **主观环节强行量化成"可自愈、有退出条件"的闭环。** UI 还原用像素+SSIM 双 95%、最多 5 轮、耗尽自动 fallback 交人决策；API 测试失败走写死 SOP，同一用例 3 轮未修通 STOP 标注"需人工介入"。自愈循环必须带轮次上限和升级路径，否则就是无限烧 token 的假自动化。"A→B 一步到位很差时，中间插一层 A→C→B"（Figma→html/css/切图→代码）是可复用的普适手法。

5. **高危动作设人工闸门：SQL 一律 yes/no，灰度回滚留给 SRE。** 数据库 DDL/DML 不允许 AI 自动执行，必须拼成可读 diff 由用户显式确认，Critical 修复也不例外——这是踩过线上事故后立的规矩。边界同样重要：AI 不解决灰度策略和回滚决策，只负责"按计划执行、失败时报错"。部署阶段宁可慢，不可错。

6. **归档当复利：三件套不跑完，下一个 change 的 P1 不许起。** changes-sync（代码与文档对齐）→ knowledge-sync（坑与契约沉淀进 specs/）→ specs-generator（按 delta-spec 的 ADDED/MODIFIED/REMOVED/RENAMED 四类标记增量合并，防知识库膨胀）。这套知识库作为 AI 长期记忆的运作方式可与 [[entities/tencent-ai-team-knowledge-harness]] 互参；同时警惕作者承认的缺口——老化淘汰机制缺位，知识库只进不出终会稀释信噪比。另外沙箱直写模式（sandbox_mode 字段贯穿 P2→P3，见 [[concepts/agent-sandbox]]）提示：AI 写入行为的隔离开关应该是显式契约字段，不是口头约定。
## 来源

- 原文：[[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md|开启Harness Engineering探索之旅 腾讯技术工程]]

## 关联

- 同题异语种孪生页：[[entities/harness-engineering-exploration-tencent-tech]]（归并候选，提案卡 #11 批1）
