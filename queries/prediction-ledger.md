---
title: 预测对账台账
created: 2026-09-05
updated: 2026-09-05
type: query
tags: [meta, prediction, temporal, ledger, dashboard]
confidence: high
---

# 预测对账台账（Prediction Ledger）

> wiki 每天摄入带时间谓词的断言（"到 2026 年底将 X"），但记了不还账：质量闭环对账页面质量，没人对账断言真值。本页是时间闭环的容器：凡库内记录的、有可判定到期日的预测，登记在册，到期回访，判定写回。建立于 [[drafts/wiki-emergent-viewpoints-2026-09-time-slice|2026-09 时间切片透镜轮]]，背景与"supersedes 不适配此用途"的论证见该稿观点三、四。

## 台账

| # | 断言 | 记录页 | 入库 | 到期 | 判定 | 判定日 |
|---|------|--------|------|------|------|--------|
| 1 | DeepSeek V4"正面追赶闭源模型架构差距"，抹平与 Opus/GPT-5 最大模型的量级差 | [[entities/deepseek-v4]] | 2026-05-13 | 2026-08-30（正式版） | **partially-falsified**：交付为第二梯队平价（与 kimi-k3/Fable 5 持平），未及闭源量级；[[entities/连夜实测deepseek-v4-pro-正式版低于预期不推荐接入codex|实测]] | 2026-09-05 |
| 2 | Gartner：2026 年底 40% 企业应用嵌入任务专用 AI agent（2025 年 <5%） | [[entities/agentic-design-system-from-chatbot-to-orchestration]] | 2026-05-14 | 2026-12-31 | pending | — |
| 3 | METR 曲线外推：AI 自主工作时长 2026 年底达 100 小时 | [[entities/anthropic-联创2028-年实现-ai-自我构建的概率超过-60]] | 2026-05-21 | 2026-12-31 | pending | — |
| 4 | "2026 年可能成为 AI 发展的分水岭" | [[entities/2028-two-scenarios-for-global-ai-leadership]] | 2026-05-16 | 2026-12-31 | pending | — |
| 5 | AI 编程辅助能力边界在 6-12 个月内快速扩展 | [[entities/2028-two-scenarios-for-global-ai-leadership]] | 2026-05-16 | 2026-11 ~ 2027-05 | pending | — |
| 6 | Jack Clark：2028 年底 60%+ 概率出现可完全自主训练下一代 AI 的系统 | [[entities/alphaevolve交出一周年炸裂成绩单ai自我改进不再科幻]] | 2026-05-21 | 2028-12-31 | pending | — |
| 7 | AIDLC：3 年内 AI 原生与平台驱动数据团队人效差距拉大 5-10 倍 | [[entities/ai-驱动的大数据工程-从平台驱动到-aidlc-的范式迁移]] | 2026-05-21 | 2029-05-21 | pending | — |
| 8 | 对抗页三问①：长程任务上裸模型与 harness-agent 差距持续 ≥2 代模型不收敛（正方赢；一代内收敛 ≥50% = 反方赢） | [[comparisons/model-capability-vs-harness-engineering]] | 2026-09-05 | 2027-09-05（跨 2 代） | pending | — |
| 9 | 对抗页三问②：harness 工件平均存活期随模型升级**变长**（正方赢；变短 = 衰减假说成立）。证据口径：框架页"衰减"节记录链（现值 -38%/代）+ 后续同源记录追加 | [[comparisons/model-capability-vs-harness-engineering]] · [[concepts/harness-engineering-framework]] | 2026-09-05 | 2027-03-01（半年度） | pending | — |
| 10 | 对抗页三问③：同任务 harness 开销占比下降且绝对收益上升（正方赢；绝对收益同步下降 = 反方赢）。证据口径：成本可对账的任务页对（如 [[entities/computer-use-45x-more-expensive-than-structured-apis|Computer Use 45x]] 族） | [[comparisons/model-capability-vs-harness-engineering]] | 2026-09-05 | 2027-03-01（半年度） | pending | — |

## 中期检视（2026-09-05 · 非判定）

台账 #2-#5 距到期尚有 ~4 个月，按对账仪式提前记轨迹，不硬判：

- **#2 Gartner 40%**：`data-insufficient`。近 90 天簇内只有零散企业 agent 部署页（浪潮 OCP agent 规模基建、火山引擎 agentic 数据底座），无可对账的渗透率口径。12 月回访时需先定"企业应用嵌入"的计量来源。
- **#3 METR 100h**：`on-track`（弱证据）。[[entities/mirrorcode-long-horizon-benchmark-epoch-ai-metr|MirrorCode 长程基准]]（08-05）记录 Claude Opus 4.7 以 14 小时/$251 重写人类需 2-17 周的 1.6 万行生物信息工具包——长程任务时长在快速爬坡，但"自主工作时长"口径与该基准不同，只作方向性佐证。另见 [[entities/metr-openai-hugging-face-agent-altruism-investigation|METR OpenAI/HF 调查]]（08-28）。
- **#4 分水岭论**：`unfalsifiable-at-midterm`。主观判断无客观口径；[[drafts/2026-h1-agent-engineering-trends|2026-H1 趋势稿]]可作年终回访的对照基线。
- **#5 能力边界 6-12 个月**：`on-track`。#3 的 MirrorCode 证据 + 本轮撞库发现的 harness 工程化速度（矛盾升级线首日即产出 debate 页）与"快速扩展"方向一致。2026-11 起可进入判定。

## 对账仪式

1. **登记**：摄入或透镜轮发现带到期日的预测时追加行；无到期日的改进性断言不登记（那是 [[concepts/claim-half-life|半衰期]] 的范畴）。
2. **到期提示**：`daily-checkup` 或季度回访时检查到期行；到期项进入当日涌现/评审动作。
3. **判定词汇**：`fulfilled` / `partially-fulfilled` / `falsified` / `partially-falsified` / `unfalsifiable`（到期时无法客观判定则如实标注，不硬判）。
4. **判定要求**：引用库内交付侧证据页（如实测页、后续系统卡）；判定日写实际回访日。
5. **产出回流**：批量判定完成后写进当期涌现稿（[[drafts/wiki-emergent-viewpoints-2026-09-time-slice|范例]]），并在 [[queries/vault-evolution-dashboard|进化仪表板]] 记一笔。

## 关联

- [[queries/emergence-lens-rotation|涌现透镜轮换]] — 时间切片透镜的常量化容器就是本页
- [[concepts/claim-half-life|断言半衰期]] — 判定结果反哺半衰期测量表
- [[queries/vault-evolution-dashboard|进化仪表板]] — 下一步第 4 项即本页
