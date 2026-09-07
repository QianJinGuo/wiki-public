---
title: "工具型 Agent 分层评测：Outcome/Decision/Action/Reliability 四层 + 过程归因与发布门禁"
created: 2026-09-04
updated: 2026-09-07
type: entity
tags: [agent-evaluation, layered-evaluation, process-evaluation, trajectory-evaluation, hard-gate, root-cause-attribution, release-gating, tool-use-agent, aliexpress, validators]
sources: [raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 工具型 Agent 分层评测：Outcome/Decision/Action/Reliability 四层 + 过程归因与发布门禁

> AliExpress技术（布雷扬）面向含多步决策和工具调用的 Agent，把评测对象从 Final Answer 扩展到可观察执行轨迹，再走向失败归因。核心产出是 Outcome/Decision/Action/Reliability 四层评测框架 + Case Contract + Validator/Judge/Hard Gate 分工 + 根因→修复对象映射 + 渐进式发布门禁。^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md]

## 基本论断：Completed ≠ Correct ≠ Ready for Release

Runtime 的 completed 只说明系统认为执行结束，不能自动证明数据正确、业务口径正确、结果完整，更不证明满足发布条件。^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md] Agent 的最终答案来自一条执行链（识别任务→选 Skill→调工具→处理结果→生成产物），最终文本只保留链路的部分信息，只看它很难区分"模型不会""工具不可用""SQL 写错"与"输出格式不稳定"。因此评测需同时读取结果和过程证据——可观察轨迹（Skill 路由、工具名、参数摘要、状态码、产物引用、外部错误），而非模型私有思维链。

## 评测对象的三阶段演进

评测对象从最终结果走向过程归因分三阶段：第一阶段只检查最终结果（参考答案/规则/LLM-as-a-Judge），适合短问答但压掉工具调用证据；第二阶段把执行轨迹纳入评测（TRAJECT-Bench 检查工具选择/参数正确性/调用依赖顺序，EnConda-Bench 把环境配置任务拆成规划/诊断/修复/执行）；第三阶段从"判错"走向"归因"（Agent-as-a-Judge 主动检查证据，AgentRx 从失败轨迹定位关键步骤与根因，SWE-Together 纳入用户纠正次数）。^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md]

## Outcome/Decision/Action/Reliability 四层框架

四层不是为了增指标数量，而是对应不同问题类型和修复对象：^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md]

| 层级 | 回答的问题 | 典型检查 | 失败后优先修改 |
|------|-----------|---------|--------------|
| Outcome | 最终做成了吗？ | 结果、数值、格式、业务护栏 | 数据逻辑、产物生成 |
| Decision | 选对了吗？ | Skill、工具、执行路线 | 路由、指令、Skill 描述 |
| Action | 做对了吗？ | 参数、顺序、状态处理、副作用 | Tool Schema、执行器 |
| Reliability | 多跑几次还可靠吗？ | 成功率、P90、成本、人工纠正 | 模型配置、重试、预算 |

Outcome 检查输出是否满足 Schema/报告模板、必需字段、数值容差、样本量与统计口径、业务护栏、产物真实存在。Decision 检查任务识别、Skill 适配、必需工具是否进计划、是否违反"必须查询/禁止写入"约束——即使结果碰巧正确也可能选高风险或不可复现路线。Action 检查参数完整性、调用依赖顺序、工具失败是否被误判成功、副作用幂等保护、是否反复执行同一动作。Reliability 需在多次真实运行上统计通过率/连续通过率、P50/P90 延迟、Token 与工具成本、用户纠正次数、超时限流与工具错误率。^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md]

## 判定机制：确定性优先 + Hard Gate 发布门禁

完整流程从运行前冻结 Case Contract 开始（四类约束须运行前确定：输入、必需能力、结果断言、风险等级），依次经确定性 Validator（Schema/字段/数值/必需工具/参数范围/状态码）→ Agent/LLM Judge（解释完整性/证据支持结论/开放业务判断）→ 人工复核。能由程序精确判断的事实不应交给概率裁判；Judge 必须看到证据而非只看最终答案。^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md]

**Hard Gate 独立于综合得分**：关键数值错误、权限越界、输出 Contract 失败和高风险副作用，不能因为"文案很好"被平均成及格总分。发布条件=所有 Hard Gate 通过且 Outcome/Decision/Action 均达最低阈值；数值字段用相对误差判定，超容差直接 BLOCK。^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md]

## 根因→修复对象映射

评测不只"判卷"，还把问题送到正确的修改层：^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md]

- 数据/指标逻辑（查询成功但关键数值断言失败）→ SQL、数据路由、统计口径
- Skill/策略（选错 Skill 或违反任务约束）→ 路由、系统指令、Skill 描述
- Tool 参数/顺序（漏调用、参数错、依赖序错）→ Tool Schema、计划、执行器
- 输出契约（内容可读但 Schema/字段不满足）→ 接口 Contract、确定性序列化
- 环境/权限（必需工具缺失或 permission denied）→ Runtime、白名单、部署配置
- 模型语义（证据完整但解释归纳仍错）→ 模型、Prompt、上下文

## 渐进式发布门禁与工程闭环

发布门禁可采用 Shadow（只记录命中统计误报漏报）→ Warning（展示证据根因、允许人工确认发布）→ Blocking（仅稳定可解释高风险规则升级 Hard Gate）→ 持续维护（规则/Golden/容差版本化、例外放行记录原因）的渐进方式。^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md] 最小闭环五环节：沉淀 Case → 版本化 Case Contract → Trace Adapter 统一证据 → Validator/Judge/Hard Gate 分层判定 → 输出含发布结论/证据/根因/修复对象的 Scorecard 并回归，形成 Case → Run → Evaluation → Fix → Regression 持续闭环。不同 Agent 类型有不同评测重点：数据分析 Agent 重数值口径、检索 Agent 重来源与引用一致、工具执行 Agent 重参数/顺序/幂等、多 Skill Agent 重路由准确率、写操作 Agent 重权限/审计/副作用 Hard Gate。

## 验证边界

本文完成的是 3 个 Case、5 条 Run 的机制验证（数值与口径错误、输出契约错误、环境与权限错误三类），证明分层规则、Hard Gate 和根因输出按预期执行；五条 Run 的 Runtime 均 completed 但只有两条修复后满足发布条件。它尚未覆盖未知失败上的识别准确率，也未完成 Decision 与 Reliability 的真实效果验证；后续需靠历史回填（报告 TP/FP/FN/TN 命中/误报/漏报）与多次独立运行补充证据。^[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04.md]

## 与现有评测知识的关系

- **互补（同源不同框架）**：[[entities/agent-evaluation-fine-grained-system-aliexpress-2026|AliExpress 模块级白盒评测]]（砚东）是"评什么/怎么采集指标"——按架构模块（感知/规划/记忆/工具）白盒诊断，质量×成本×性能三维 35+ 指标；本文是"结果驱动四层 + 从过程归因 + 发布决策"，两条轴正交，同时入库互为诊断工具与评分体系。
- **互补**：[[entities/agent-evaluation-turing-meituan-2026|美团图灵评测方法论]] 聚焦评估方法论（人人一致/独裁者/Rubric 二元化/桥梁指标）；本文聚焦工具型 Agent 的过程轨迹归因与发布门禁。
- **呼应**：[[entities/harness-engineering|Harness Engineering]] 中 Agent 运行证据审计、根因定位的实践；「确定性下沉、不确定性上浮」（FDE 产品架构）与本文 Validator 确定性优先原则同一思想——能确定判定的不留概率。

→ [[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04|原文存档]]