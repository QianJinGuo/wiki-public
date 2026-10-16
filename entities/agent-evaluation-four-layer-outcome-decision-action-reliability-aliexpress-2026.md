---
title: "工具型 Agent 分层评测：Outcome/Decision/Action/Reliability 四层 + 过程归因与发布门禁"
created: 2026-09-04
updated: 2026-10-15
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

## 深度分析

### 为什么每一层的"完成"都不等于"正确"

Completed 是 Runtime 的状态位，而 Correct 是语义断言，两者位于不同的证据空间，这正是四层各自需要独立判定的根本原因。在 Outcome 层，"有答案"与"答案可信"之间隔着 Schema 满足性、数值容差、样本量口径和业务护栏四道断言：一个数据分析 Agent 返回了结构完整、语言流畅的 Markdown 报告，但关键值实际 100、参考 250，相对误差 60%——Runtime 视角是 completed，业务视角是报废。在 Decision 层，结果碰巧正确反而可能掩盖更危险的问题：本应查询权威数据源却用模型记忆直接作答，答案正确但路线不可复现、不可审计，一旦底层数据变化就会静默失效。在 Action 层，Decision 正确不保证执行正确——参数类型错误、调用依赖序错乱、工具失败被误读为成功，这些偏差都发生在"选对"与"做对"之间的缝隙里。在 Reliability 层，单次通过的证据强度最低：Agent 有随机性，一次成功可能只是采样运气，连续通过率与 P90 才构成"可交付"的统计证据。四层各自回答"完成"到"正确"之间的不同缺口，删掉任何一层，对应类型的失败都会伪装成成功。

### 确定性 Validator 与 LLM Judge 的经济学

"能由程序精确判断的事实不应交给概率裁判"不只是原则，更是成本与方差的双重最优。成本侧：确定性 Validator 是一次字符串比较或数值容差计算，边际成本近乎为零，可以放在 CI 每次提交、PR 回归、定时评测里高频运行；LLM Judge 每次调用消耗 Token 且随证据长度线性增长，只适合处理 Schema/字段/数值/状态码覆盖不了的语义维度——解释是否完整、证据是否支持结论、开放业务判断。方差侧：Validator 的判定是可复现的（同一输入永远同一结论），因此才能承担 Hard Gate 职责——发布门禁的每一次 BLOCK 都必须经得起追溯和申诉；LLM Judge 的判定有采样方差，同一证据可能给出不同结论，把它放在门禁位上会引入"随机阻断"，放在分级判定链的前端做守门、后端做解释才是正确分工。这套"确定性下沉、概率上浮"的经济学与其他评测体系的结论一致：[[entities/llm-as-a-judge-agent-eval-offline-huolala-2026|货拉拉 LLM-as-a-Judge 离线评测]] 同样把 Judge 限定在规则难以覆盖的语义层，并用人工复核抽样校准其漂移。

### 层级归因即调试压缩

四层框架的真正杠杆不在"评分"，而在把一次失败从"整条执行链都可能是嫌疑"压缩到"这一层的一个修复对象"。没有分层时，工程师面对一次失败要人工排查：Skill 路由对不对？参数全不全？SQL 口径错没错？环境权限够不够？——这是一次 O(全链路) 的线性搜索。分层归因把这个搜索压缩成 O(1) 的查表：关键数值断言失败 + 查询本身成功 → 数据逻辑层，改 SQL 或统计口径；必需工具返回 permission_denied + 路由正确 → 环境层，改白名单或部署；证据完整但归纳仍错 → 模型语义层，才轮到改 Prompt 或换模型。原文的三类机制验证（数值口径、输出契约、环境权限）本质上是在验证这张映射表的查表正确性。调试压缩的价值随系统规模放大：Agent 链路越长、Skill 越多，"先归因再修复"相对"逐段排查"节省的定位成本越大，这正是评测从"判卷"演化为"调试基础设施"的含义。Trajectory 层面的同类实践（把执行轨迹结构化为可检索证据再定位失败环节）在 [[entities/langsmith-trajectory-evals|LangSmith 轨迹评测]] 中也有独立印证。

### Hard Gate 与综合得分的组合为什么抗"高分掩盖"

平均分机制的失效模式是线性可加性假设：它隐含地允许一维的优势（文案流畅、格式漂亮）抵偿另一维的致命缺陷（数值错误、权限越界）。Hard Gate 的设计则把缺陷分为两类处理——可容忍维度进综合得分做排序与趋势比较，不可容忍维度（关键数值错误、权限越界、输出契约失败、高风险副作用）进布尔门禁，任何一项失败直接 BLOCK，不参与平均。发布条件因此是"所有 Hard Gate 通过 **且** Outcome/Decision/Action 达最低阈值"的合取，而不是加权分数过线。这种组合在工程上等价于把"及格线"从连续量纲拆成离散约束集，其代价是维护成本（Gate 规则需版本化、例外放行需记录原因），收益是高风险维度永远无法被其他维度的长板稀释。这与 AI 编码评测中"五维度 + 三级门禁"的组合逻辑同构：[[entities/ai-coding-practice-agent-evaluation-five-dimension-three-level-gating|AI 编码五维度三级门禁评测]] 同样把阻断项从评分体系中剥离为独立门禁。

## 实践启示

1. **把"发布条件"写成合取式，而非加权总分**：所有 Hard Gate 通过 AND 三层达最低阈值。任何"综合分 85 但数值断言失败"的 Case 都应 BLOCK——数值容差用相对误差 |实际−参考|/max(|参考|, ε) 判定，超容差不进平均。
2. **评测标准冻结在运行之前**：Case Contract 的四类约束（输入、必需能力、结果断言、风险等级）必须在 Run 前确定，否则断言会被实际输出反向"拟合"，评测退化成事后解释。契约要版本化，随业务口径演进而非随失败临时修改。
3. **按根因分配修复工单，而不是按症状**：出现失败时先归因到层（数据逻辑 / Skill 路由 / Tool 参数顺序 / 输出契约 / 环境权限 / 模型语义），再决定改 SQL、改描述、改 Schema、改部署还是改 Prompt。最常见的浪费是结果错了就去调 Prompt，而真实根因在统计口径或白名单。
4. **给 Judge 喂证据，而不是喂最终答案**：LLM Judge 只在 Validator 覆盖不了的语义维度上启用，且必须读取工具状态、参数摘要、产物引用等可观察轨迹——只看最终文本的 Judge 会被"语言流畅的错误"系统性欺骗。
5. **单次通过不下结论，先跑满统计口径**：同一 Case 至少重复运行统计连续通过率与 P90，并保留模型/Skill/Tool/环境版本，避免把环境变化误判为模型波动。Reliability 的证据只能来自多次独立运行，不能用一次高分代替。
6. **新规则先 Shadow 后 Blocking**：新增的判定规则先旁路运行统计误报/漏报，稳定后再升级为 Warning 乃至 Hard Gate；例外放行必须记录原因，防止门禁被逐次豁免而空心化。Agent 的稳定性问题（上下文漂移、工具幻觉等长尾失效）尤其依赖这种渐进收口，可参考 [[entities/agent-reliability-context-drift-tool-hallucination|Agent 稳定性与上下文漂移]]。

## 与现有评测知识的关系

- **互补（同源不同框架）**：[[entities/agent-evaluation-fine-grained-system-aliexpress-2026|AliExpress 模块级白盒评测]]（砚东）是"评什么/怎么采集指标"——按架构模块（感知/规划/记忆/工具）白盒诊断，质量×成本×性能三维 35+ 指标；本文是"结果驱动四层 + 从过程归因 + 发布决策"，两条轴正交，同时入库互为诊断工具与评分体系。
- **互补**：[[entities/agent-evaluation-turing-meituan-2026|美团图灵评测方法论]] 聚焦评估方法论（人人一致/独裁者/Rubric 二元化/桥梁指标）；本文聚焦工具型 Agent 的过程轨迹归因与发布门禁。
- **呼应**：[[entities/harness-engineering|Harness Engineering]] 中 Agent 运行证据审计、根因定位的实践；「确定性下沉、不确定性上浮」（FDE 产品架构）与本文 Validator 确定性优先原则同一思想——能确定判定的不留概率。

→ [[raw/articles/agent-evaluation-four-layer-process-gating-aliexpress-2026-09-04|原文存档]]