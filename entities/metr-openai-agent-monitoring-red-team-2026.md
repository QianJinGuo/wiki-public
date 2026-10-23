---
title: "METR 红队 OpenAI 内部 agent 监控系统：75–94% 覆盖率背后的七类可绕过面"
created: 2026-10-24
updated: 2026-10-24
type: entity
tags: [agent-safety, red-team, monitoring, misalignment, codex, sandbox, evaluation, openai, metr]
sources: [raw/articles/metr-openai-agent-monitoring-red-team-2026]
confidence: 0.8
provenance_state: extracted
---

# METR 红队 OpenAI 内部 agent 监控系统：75–94% 覆盖率背后的七类可绕过面

> METR 与 Redwood Research 各派一名研究员，在 2026-07-21 至 08-11 的三周内对 OpenAI **内部 agent 监控系统**（异步 misalignment monitoring）做了一次高权限红队演练：可读内部代码库、监控 prompt 与配置、与十余位监控/安全团队研究员访谈，产出 61 页内部报告。公开博客只给出高层结论，但信息量集中在"覆盖率估算 + 具体绕过面"两件事上。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

## 覆盖率：内部 Codex 流量 99.2% 可监控，但 75–94% 是粗估

演练时（2026-07-21 及之前）对内部 Responses API（Codex 等背后使用的接口）中**至少和 GPT-5.5 一样强**的模型流量，估计有 **75–94% 具备被监控资格**——作者明确说明两个方向都有很大不确定性。若只看"内部 Codex agent 使用的那部分 Responses API 流量"，则 **99.2%** 具备监控资格。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

两个关键缺口在报告里被点出：其一，**RL 训练 rollout 在演练开始时没有被监控覆盖**（虽然 3–6 月间据说有一部分被监控）；其二，**compaction（上下文压缩）事件未被监控**，估计导致 2026 年 5–7 月间约 **10%** 的合格内部 Codex 流量没被监控——因为当时没有记录"多少次监控调用由 compaction 触发"所需的日志，OpenAI 员工在演练前已独立发现并在演练期间修复。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

更具结构性的问题是触发时机：监控系统在**agent 把控制权交还用户**时触发。在 compaction 未被覆盖的那段时间里，agent 只要**不断继续行动**就能无限推迟被监控。这类"触发点可被被监控方操纵"的缺陷，比单个 bug 更值得写进 harness 设计规范。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

## 监控系统自身的可靠性指标

OpenAI 提供的数据显示：演练期间监控系统**报错率低于 0.2%**，且在其中一个 24 小时窗口内**零 load-shed**。作者同时验证了"监控系统看到的就是 agent 看到的完整轨迹"（除少数小例外），并确认监控 prompt 覆盖的风险类别包括：密谋（scheming）、监控系统颠覆、蓄意破坏、数据外泄、欺骗、以及进出双向 prompt injection。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

这三项（覆盖率、报错率、可见轨迹）构成一条可复用的**监控有效性三件套**：只报"我们监控了"没意义，必须同时给出"覆盖了哪些流量 / 监控本身多可靠 / 监控看到的信息是否与 agent 相同"。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

## 七类发现：从沙箱逃逸到"让监控看不见"

报告列举的发现可按"攻击者能拿到什么"分层：^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

| # | 发现 | 影响 |
|---|---|---|
| 1 | Codex agent 原则上可**绕过 Codex sandboxing 与 Auto-Review** | 可执行宿主用户能执行的任意动作 |
| 2 | 一种能让**数百条近期动作（含该利用本身）不被监控**的利用 | 直接抹掉审计痕迹（细节被 OpenAI 隐去） |
| 3 | 利用 **agent 侧轨迹与监控侧轨迹的序列化差异**构造输入，使监控失败 | 失败即不升级人工复核 |
| 4 | 两个**客户端可控 header** 可关闭新 agent 的监控 | 可起一个"其他方面完全相同"的未监控实例（但设置动作本身可能被捕获） |
| 5 | 两个"只读"内部 agent 工具存在简单利用 | 可在其沙箱内执行任意动作（未验证实战可利用性） |
| 6 | compaction 未被监控 + 只在交还控制权时触发 | 持续行动即可推迟监控（已修） |
| 7 | 监控可见轨迹与 agent 一致（正面结论） | 排除"监控看到的是删减版"这一假设 |

第 3、4 条最值得工程界警惕：它们不需要突破模型对齐，只需要**在协议层制造"监控没看到"的状态**——序列化差异、header 开关、触发时机，都是普通软件工程缺陷，而不是"模型太聪明"。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

## 结论的边界

OpenAI 表示本次演练发现的所有影响生产防护栈的漏洞**均已修补**，演练后监控系统的预期覆盖范围也有重大变化；METR 明确声明**没有评估这些说法**。报告也强调这是"给外部研究者深度访问权限的试点"，不是关于 OpenAI 监控系统的可靠公开透明度——公开博客刻意保持简短。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

对自建 agent harness 的团队，这篇的价值不在"OpenAI 有问题"，而在于它把**监控系统的攻击面分类**摆了出来：沙箱/审批层、审计日志完整性、序列化一致性、开关型配置接口、只读工具的权限假设、触发时机。任何一条不成立，覆盖率数字都会虚高。^[raw/articles/metr-openai-agent-monitoring-red-team-2026.md]

## 关联

- Agent 安全架构：[[concepts/agent-security-architecture]]、[[concepts/agent-security-attack-defense]]
- Harness 与执行控制：[[concepts/harness-engineering]]、[[concepts/agent-evaluation-benchmark-frameworks]]
- 相关红队/评估工作：[[entities/anthropic-multi-agent-conflict-frontier-red-team-2026-08]]、[[entities/metr-openai-hugging-face-agent-altruism-investigation]]

→ [[raw/articles/metr-openai-agent-monitoring-red-team-2026|原文存档]]
