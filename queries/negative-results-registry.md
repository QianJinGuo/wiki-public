---
title: 负结果登记簿
created: 2026-09-05
updated: 2026-09-05
type: query
tags: [meta, negative-results, evaluation, dashboard, epistemics]
confidence: high
---

# 负结果登记簿（Negative Results Registry）

> 本库的正结果有完整的摄入-合成-索引管线，负结果（"干预 X 使 Y 变差"）却散落在页面段落里、没有任何容器汇总——2026-09 反方立场透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-adversarial|涌现稿]]）识别此为论证结构单向性的根源。本页补这个容器。收录标准：**有对照的变差证据**（同任务/同模型/可比较），纯观点否定不收。

## 在册负结果

| # | 干预 | 结果 | 对照证据 | 来源页 | 日期 |
|---|------|------|----------|--------|------|
| 1 | 强模型（Pro）挂载长 Skill | 60.1→50.7（-9.4） | 同任务无 Skill 基线 | [[entities/gene-gep-evomap-qinghua-strategy-genes-arxiv-2604-15097-2026|Gene-GEP]] | 2026-05 |
| 2 | Computer Use 执行 SaaS 操作 | 成本 ×45 | 结构化 API 同任务 | [[entities/computer-use-45x-more-expensive-than-structured-apis|45x 对比]] | 2026-06 |
| 3 | DeepSeek V4 挂载 Codex 主力位 | 4100 万 token 实测不适合 | kimi-k3/Fable 5 平价但更稳 | [[entities/连夜实测deepseek-v4-pro-正式版低于预期不推荐接入codex|连夜实测]] | 2026-08-30 |
| 4 | Cursor 远程隧道 + Shell builtin 沙箱策略 | 可被组合攻破（逃逸链） | 攻击复现 | [[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|NomShub]] | 2026 |
| 5 | 上一代模型的 harness 工件直接复用 | 成本节省优势 -38%/代（衰减） | Opus 4.5→4.6 对照 | [[concepts/harness-engineering-framework|框架页"衰减"节]] | 2026 |

## 登记与消费

- **登记**：透镜轮、矛盾四裁、透镜撞库发现的"变差证据"随时追加；每行必须有对照锚点（同任务/同模型基线），否则移入下方观察区。
- **消费**：[[concepts/when-not-to-harness-engineering|边界条件页]]的反向检查清单引用本簿；[[comparisons/model-capability-vs-harness-engineering|对抗页]]三问检验的负向证据来源；新摄入管线的评分环节应查本簿避免给已被证伪的做法打高分（[[concepts/claim-half-life|半衰期]]打折的负向版）。

## 观察区（有嫌疑、缺对照）

- entities/agent-memory-architecture 与 `-essence` 双页同标题并存（疑似重复入库，待 quality 轮裁决，见 twin-cleanup 2026-09-05 记录）
- `harness` 簇 517 页均分 7.73 高于全库 7.35（+0.38，n=347）——尚不能区分"簇质量高"与"同温层互评偏置"（49% 来源为单一渠道 mp.weixin.qq.com）

## 关联

[[queries/prediction-ledger|预测对账台账]]（正向前瞻）· [[concepts/claim-half-life|断言半衰期]]（时间衰减）· 本页（负向实证）——三者构成库内断言的三轴对账。
