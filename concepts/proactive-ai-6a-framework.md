---
title: "主动性的 6A 框架（Proactive AI）"
created: 2026-10-22
updated: 2026-10-22
type: concept
tags: [agent, proactivity, autonomy, alignment, long-running, harness, evaluation, concept]
description: "把主动性拆成 Awareness/Anticipation/Agenda/Arbitration/Action/Adaptation 六项能力，并与智能、对齐、自主程度区分"
sources:
  - raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026
  - entities/amap-proactive-agent-lbs-spacetime
  - entities/claude-code-routines-proactive-agent
  - entities/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026
confidence: 0.7
---

# 主动性的 6A 框架（Proactive AI）

**主动性（proactivity）** 指智能体在没有新指令时自行发现"什么值得做"，并判断"是否、何时、怎样介入"。UIUC、谷歌、斯坦福、UC Berkeley、CMU 的联合综述《Proactive AI for Superintelligent Agents》把它提为与**智能**、**对齐**并列的第三个评价轴，并用 6A 框架拆解其能力构成。^[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026.md]

## 为什么主动性是独立的一维

主动性不是"更自主"。综述用「剩余决策空间（residual decision space）」刻画：一项委托里总有一部分判断没有被预先规定——参会日期、预算上限可以写明，但"转机风险与票价如何取舍""住宿无法延长时怎么调整""哪些已有安排值得保留"往往留待执行时决定。主动性处理的是**发现并界定**这些剩余决策，自主程度处理的是**获准独立执行到哪一步**，两者可以分开配置：助手可以主动发现行程冲突并准备改签方案，而把付款留给用户确认。^[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026.md]

这条划分对工程有直接含义：一个系统的"主动性"上去了，不等于它的执行权限也要跟上；反之授权范围大的系统也可能完全不主动。把两者混为一谈，会导致要么授权过宽、要么主动能力被权限设计压死。^[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026.md]

## 6A 能力分解

| 能力 | 回答的问题 | 典型机制 | 行程案例中的表现 |
|------|-----------|---------|-----------------|
| Awareness | 现在发生了什么、哪些状态变了 | 跨会话状态跟踪、信息获取、记忆管理 | 把新会议结束时间与机票/酒店订单关联，更新对行程的理解 |
| Anticipation | 变化会带来什么后果 | 用户模型 + 世界模型、行动与等待的后果比较 | 推断返程与最后一天会议冲突、延住影响预算、取消期限临近 |
| Agenda | 哪些事值得做、按什么优先级持续跟进 | 目标选择、待办维护（依据/进度/依赖） | 把改签、延住、费用对照组织成可推进的待办 |
| Arbitration | 此刻推进到哪一步 | 结合证据、收益、时间、风险与可撤回程度判断介入时机与强度 | 先查条件与整理方案，把需决策的事项提出，待确认后再动订单 |
| Action | 怎么执行并保持可继续 | 工具调用、权限约束、执行反馈 | 在获授权后完成改签，未授权部分留待确认 |
| Adaptation | 结果与反馈如何修订理解 | 把执行结果与人的反馈回写目标/记忆/优先级 | 费用或余票推测被查询结果修正；新任务到来时重排优先级 |

6A 的价值在于它把"主动性"从形容词变成可检查的清单：任一环节缺失，主动性就会退化为某一种具体的失败形态——没有 Awareness 是幻觉式主动，没有 Anticipation 是噪音式提醒，没有 Agenda 是一次性建议，没有 Arbitration 是越权执行。^[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026.md]

## 与 harness 的关系

综述明确把 harness 列为主动性的使能层：LLM/多模态模型负责处理文本、图像与情境信息，而 **agent harness 提供的记忆、工具、技能与执行管理**才让系统能接入外部应用、保存进度、跨会话继续任务。换言之，主动性不是靠更好的 prompt 得到的，而是靠"状态能不能存住、工具能不能调通、权限能不能分级"这三件工程事撑起来的。^[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026.md]

这与 wiki 既有的长任务结论一致：跨会话续接的难点是状态一致性而非模型能力，见 [[concepts/long-running-agent-architecture|长任务 Agent 架构]] 与 [[concepts/harness-long-running-task|长任务 Harness]]。记忆侧的 6A 前两项（Awareness/Anticipation）本质上要求记忆不仅能检索，还要能**按当前目标判断哪些变化相关**，这与 [[concepts/agent-memory-system-design|Agent Memory 系统设计]] 里的写-组织-读生命周期是同一问题的两个入口。

## 已有落地形态对照

wiki 里三个既有实体分别覆盖了 6A 的不同子集，可以当作框架的实例校准：

- [[entities/amap-proactive-agent-lbs-spacetime|高德主动式 Agent（LBS+时空）]] — Awareness/Anticipation 强（时空情境推断需求），Agenda 偏弱（场景触发而非长期待办）。
- [[entities/claude-code-routines-proactive-agent|Claude Code Routines]] — Agenda/Action 强（把重复工作固化为可触发的例行程序），Anticipation 弱（由人定义触发条件）。
- [[entities/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026|Remember When It Matters]] — Awareness/Adaptation 强（长程记忆的写入与召回时机），Arbitration 弱（不涉及介入强度判断）。

三者合起来仍缺"Arbitration（介入时机与强度的显式判断）"这一环——这也是综述指出的开放问题：主动系统最难的不是发现需求，而是决定**现在是否值得打扰人**。^[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026.md]

## 评测与研究缺口

综述覆盖 13 个应用领域并梳理评测方法，但主动性的评测本身有结构性困难：漏报（该主动时没主动）与误报（不该打扰时打扰）代价不对称，且"正确时机"没有静态标注可依。可操作的评测设计至少要区分三件事——发现率（是否找到值得做的事）、介入质量（时机与形式）、以及放弃判断（是否正确地选择不介入）。^[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026.md]

工程上更现实的路径是把主动性当作可分级能力逐档放开：先做"发现但只提示"，再做"准备方案待确认"，最后才是"在授权范围内直接执行"。评测指标随档位变化，而不是用一个统一的"主动性分数"横跨三档。这与 [[concepts/evaluation-harness-design|评测 Harness 设计]] 里"评测要贴合能力档位"的思路一致。

## 局限

本页内容基于二手综述报道（新智元转述 arXiv 综述），6A 的原始定义、13 个领域的完整清单与评测细则以原论文为准；wiki 尚未收录该论文一手全文，因此具体数字与实验结论不宜引用本页。^[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026.md]

## 相关概念

- [[concepts/long-running-agent-architecture|长任务 Agent 架构]] — 主动性是长程任务能持续推进的前提
- [[concepts/harness-long-running-task|长任务 Harness]] — 状态、工具、权限三件支撑
- [[concepts/agent-memory-system-design|Agent Memory 系统设计]] — Awareness/Anticipation 的记忆底座
- [[concepts/evaluation-harness-design|评测 Harness 设计]] — 分级评测的工程方法

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
- [[moc/memory-context-systems|Memory & Context Systems]]

→ [[raw/articles/proactive-ai-superintelligent-agents-6a-framework-uiuc-google-2026|原文存档]]
