---
title: "UI2App：只给静态截图，模型能否推断并实现隐含的交互行为"
created: 2026-10-24
updated: 2026-10-25
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

## 关联

- [[concepts/coding-agent-architecture|Coding Agent 架构]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]
- [[moc/evaluation-and-benchmarks|评测与基准主题地图]]
- [[moc/coding-agent-practice|Coding Agent 实践]]

→ [[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026|原文存档]] ^[raw/articles/ui2app-interaction-inference-screenshot-to-app-benchmark-hkust-gz-2026.md]
