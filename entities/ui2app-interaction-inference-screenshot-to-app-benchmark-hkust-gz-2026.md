---
title: "UI2App：只给静态截图，模型能否推断并实现隐含的交互行为"
created: 2026-10-24
updated: 2026-11-06
type: entity
tags: [benchmark, evaluation, coding-agent, frontend, interaction-inference, arxiv]
sources: [raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026]
confidence: 0.75
provenance_state: extracted
---

# UI2App：只给静态截图，模型能否推断并实现隐含的交互行为

港科广（HKUST-GZ）团队的 UI2App（arXiv:2607.06306v2，代码 github.com/chenmancm169/UI2App）把 Screenshot-to-App 里长期被视觉相似度掩盖的一层单独拎出来量化：**Interaction Inference**——不给功能说明、不给点击轨迹、只给静态 UI 截图，模型能否自行推断其中隐含的应用行为并落地成可运行的多页面应用。 ^[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026.md]

## 为什么现有评测不够

Screenshot-to-Code 类评测大多停留在视觉相似度；少数交互式生成任务则会提供显式的行为指导。真实需求恰恰落在两者之间：用户给几张截图，关心的是"按钮能不能用""购物车会不会更新""切换页面后状态还在不在"，而不是页面像不像。UI2App 的设计因此把评测目标从**页面能否被复刻**推进到**应用行为能否从视觉线索中被推断并实现**。 ^[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026.md]

## 数据集与三个指标

| 维度 | 内容 |
|---|---|
| 规模 | 600 张截图，组成 95 组**状态一致**的多页面 Web 应用 |
| 单任务 | 3–14 张截图，需生成可运行的 React 多页面应用 |
| 评测项 | 665 个"应用-交互类别"项 |
| 输入限制 | 只有静态截图，无操作轨迹、无行为说明 |
| EXEC | 能否真正构建并运行 |
| VFS | 与截图的视觉相似度 |
| IIS | 截图暗含的交互能否被推断并实现（覆盖、真正产生预期结果、状态在组件/页面间正确保持） |

## 按"状态作用范围"分层：S1/S2/S3

UI2App 覆盖开关、展开/折叠、列表操作、数据增删改查、表单验证、通知反馈、跨页面状态共 7 类交互，并按状态作用范围给出复杂度阶梯： ^[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026.md]

- **S1 局部 UI 状态**：按钮、下拉框、标签页；
- **S2 组件间共享数据状态**：搜索、排序、CRUD；
- **S3 跨路由持续状态**：购物车、登录态、编辑中的内容。

分层的价值在于评测不只给一个总分，而能区分模型在"局部按钮""同页数据联动""跨页面状态保持"三档上的能力断层——这与 Agent 系统里 context/state 管理的工程难点正好对应（参见 [[concepts/coding-agent-architecture|Coding Agent 架构]]）。 ^[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026.md]

## 核心发现：视觉第一 ≠ 交互第一

- Gemini 3.1 Pro Preview 以 **VFS 73.2** 拿到视觉还原第一，但 **IIS 只有 8.1、排名第四**；
- Claude Sonnet 4.6 的 VFS 71.3 略低，但 **IIS 36.8**，是全评测最高值；
- 即使最高 IIS 也只有 36.8/100 — 仅凭静态截图推断并实现应用行为仍处早期；
- **S3 跨路由状态是共同难点**：除最佳者外，其余参评模型得分都不超过 3.5，最佳成绩也只有 15.8/100。

方法论结论很直白：只看视觉还原会**高估**模型对交互的实际实现能力。这也解释了为什么"截图生成 demo"在演示里惊艳、到了真实产品逻辑就露馅——难点不在还原像素，而在推断"哪些信息应该共享、一次操作会改变哪些数据、变化该在何处继续生效"。 ^[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026.md]

## 对 Agent 工程的三点启示

1. **评测要卡在"行为"而非"外观"**：UI 生成的验收标准应与产品验收一致（状态一致、跨页保持），而不是截图 diff。
2. **状态作用域是复杂度的真实刻度**：S1→S3 的能力断崖提示，构建端到端页面时跨路由状态管理应被单独设计与验证。
3. **该基准天然可用于 harness 回归**：EXEC/VFS/IIS 三档可分项打分，适合作为 coding agent 前端任务的分层验收集（参见 [[concepts/evaluation-harness-design|评测 Harness 设计]] 与 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]）。 ^[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026.md]

## 深度分析

### 交互推断的本质是逆向工程而非视觉生成

从 3–14 张截图还原一个应用，模型实际在做两件性质不同的事：像素级的视觉重建，以及对隐含"状态-迁移"模型的归纳——判断哪些截图是同一界面的不同状态、哪些操作触发界面切换、哪些数据被多处共享。前者有明确的参照目标，后者没有：行为的证据只散落在截图序列的差异里。IIS 最高仅 36.8 而各模型 VFS 普遍不低，说明瓶颈系统性地落在归纳这一步——模型能"照着画"，但不太会"从画面反推程序语义"。

### S1/S2/S3 断层的真实位置在架构层而非组件层

S1 局部交互（按钮、标签页）接近"绑定事件"即可完成，各模型表现尚可；到 S3 跨路由状态（购物车、登录态），除最佳者外全部不超过 3.5。这个断崖说明失败不在组件实现，而在全局数据流设计：状态该放在哪、路由切换后由谁持有、失效后如何恢复。这恰是 [[concepts/context-management-agent-systems|Agent 系统的 Context 管理]] 中"跨步骤状态持久化"问题的前端投影——会写组件的模型不少，会设计 store 的模型很少。

### VFS 与 IIS 排序倒挂揭示训练目标错位

Gemini 3.1 Pro VFS 第一（73.2）但 IIS 第四（8.1），Claude Sonnet 4.6 反以略低的 VFS 拿下最高 IIS（36.8）：视觉相似度排名对交互能力几乎没有预测力。多模态模型的主流对齐目标是图文匹配，而交互推断要求从视觉证据恢复程序行为，两者目标函数不同。这提示单靠更强的视觉骨干不会自动带来交互能力，需要在训练或提示层面显式引入"行为推断"的任务形态。

### "状态一致的多页面组"是更难造假的设计

UI2App 用 95 组状态一致的应用而非孤立截图构成评测，使每个交互项都隐含前后文：购物车页里的商品必须与商品页加入的一致。模型无法靠复述截图内容得分，必须实现真实数据流才能通过"操作→状态断言"。这种以状态一致性为纽带的评测构造，对任何行为型 benchmark 都有参考价值——孤立样本易被模式匹配攻破，一致约束才能逼出真实能力。

## 实践启示

### 评测集构建：采集同一应用的多状态截图序列

自建前端评测时，截图应按"同一应用在关键操作前后的多个状态"组织（列表→详情→购物车），而非随机页面拼盘；输入只给截图序列，把行为预期编码在状态差异里，判分时逐项执行"点击 X 后应出现 Y、切页后 Z 应保持"。这直接复刻 UI2App 的构造成本低，却能区分"会抄页面"与"会做应用"两类模型。

### 回归防线：用状态断言的 E2E 测试替代视觉 diff

Agent 生成前端后的验收管道可照搬三层结构：先跑构建与启动（对应 EXEC），再做截图对比（对应 VFS），最后用 Playwright 类工具执行操作脚本并断言状态变化——加入购物车后计数变更、筛选后列表收窄、刷新后登录态保留（对应 IIS）。视觉 diff 只作辅助信号，不作为放行依据，避免"像但不可用"的产出进入主干。

### 选型决策：多页面场景按 S3 分项而非综合榜选模型

若产品场景涉及购物车、登录态等多页面应用，模型选择应直接看跨路由状态分项成绩：S1/S2 高分不保证 S3 可用（多数参评模型 S3 不超过 3.5）。可行策略是按场景分层选型——单页工具类可放开选综合榜靠前者；多页应用类优先 S3 得分最高者，或让状态层由人工与显式规格接管，模型只负责页面与组件。

### 任务分解：把"交互推断"显式化为独立前置步骤

与其让模型在一次生成里同时做视觉重建和行为推断两件异质任务，不如在 agent 管线中插入一步"从截图组推导交互清单"：输出结构化规格（元素、触发、状态归属、作用范围 S1/S2/S3），经人工快速确认后再交给代码生成。这既降低了单次生成的认知负荷，也让推断错误在人审环节被拦截，与 [[concepts/context-engineering|Context 工程]] 中"先显式化隐含信息再执行"的思路一致。

### 需求接口：把截图当输入时要预留最小澄清环节

截图是无工程背景用户最自然的表达方式，但 IIS 上限表明纯截图推断不可靠。产品化截图驱动的生成时，应允许 agent 就高歧义交互（提交后跳转目标、数据是否持久化）发起少量澄清提问，用几轮交互换取状态逻辑的正确率——这比事后修一个"看着对、逻辑错"的应用便宜得多。

## 关联

- [[concepts/coding-agent-architecture|Coding Agent 架构]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]
- [[moc/evaluation-and-benchmarks|评测与基准主题地图]]
- [[moc/coding-agent-practice|Coding Agent 实践]]

→ [[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026|原文存档]] ^[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026.md]
