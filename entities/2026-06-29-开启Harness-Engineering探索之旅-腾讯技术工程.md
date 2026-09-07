---
title: "开启Harness Engineering探索之旅 腾讯技术工程"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md|原文存档]]

sha256: b9670709d7fa796cefb9e7f2a6d32a810619e03f6a5577d808a16cee4aafb744 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]

## 摘要

腾讯团队复盘其 Harness Engineering 实践：AI 写代码占比一路走高但版本节奏提效不成正比，出码率与提效之间"裂开一道缝"，根因是研发瓶颈从来不在"写"而在理解、对齐、验证、沉淀等非编码环节，需要为 Agent 搭建可执行、可约束、可验证、可反馈的工程环境 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。整套体系是"2 条轨道 + 1 个长期记忆"：研发端到端交付落在 SpecWorker 上（协议层定义 AI 每步输入输出契约、管线层标准化 P1 需求到 P6 归档的 6+1 阶段、纪律层硬编码 TDD/Debug/Verify/Review/Evaluate 五道门禁且评分 95 分以下打回重做），线上运营轨道用 7 步处理告警闭环，知识库作为 AI 的长期记忆分项目级 specs/ 与变更级 knowledge-spec/ 两套并通过 index.md 互通 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。文章给出了大量具体机制：P1 阶段 TAPD 拉取 + AC 可测 + test-cases 同源、P3 的 UI 像素+SSIM 双 95% 五轮自愈校准、P4 后端 API 测试失败时自动拉 CLS 日志查 MySQL 看 Redis 的诊断 SubAgent、P5 的 SQL 变更强制人工确认、P6 的 changes-sync/knowledge-sync/specs-generator 归档三件套 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。文末总结 4 条工程原则（追求确定性而非自由发挥、上下文控制、Token 成本优化、确定性过程用脚本实现并优先 SKILL 而非 MCP）与 4 个典型问题（指令遵循、需求歧义、设计稿还原、产物可靠性）的标准应对，并坦诚列出了六件还在路上的事 ^[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md]。

## 关键要点

- 概念脉络：Prompt Engineering（2022-2024，关心单次调用）→ Context Engineering（2025，关心每步喂什么）→ Harness Engineering（2026，关心整个任务）；Mitchell Hashimoto 2026 年 2 月命名，定义是"每当你发现 Agent 犯了一个错，就工程化一个方案让它永远不再犯"
- 协议层四件事：每步产出什么格式、文档用标准模板、写完机器自动校验、变更只记增量保留完整历史；核心立场是"人与人协作靠默契，人与 AI 协作必须靠契约"
- 设计阶段：design.md 是契约不是说明——接口签名/数据模型写死成 Markdown 表格 + Mermaid 图，D-x 改动点列表标注"文件:行号 @ 函数名"作为 P3 的工单池，前端 design.md 顶部强制 sandbox_mode 字段
- 可监测性三维度与关键踩坑：可追踪（.phase-metrics.jsonl 记录每阶段 token、耗时、代码改动量并按费率折算 estimated_cost_usd）、可回溯（按失败类型走固定 SOP 检索路径）、可度量（双层 Hook 汇聚 hook_events 表）；踩过的坑是 SubAgent 上下文独立计费——code-reviewer 最初设计为读全文件做契约 review，看似甩了负担实则另起一份账，改为优先读 git diff 并把 token 双层结算（父 Skill 与 SubAgent 独立计费）定为硬规则
- 待解决清单：评分机制与下游真实消耗未耦合、知识库自动治理（老化淘汰）缺失、运营轨道跨项目 SOP 复用在试、老项目"水下的冰山"式隐性约束适配、AI 生成测试用例质量评估

## 来源

- 原文：[[raw/articles/2026-06-29-开启Harness-Engineering探索之旅-腾讯技术工程.md|开启Harness Engineering探索之旅 腾讯技术工程]]
