---
title: 涌现透镜轮换
created: 2026-09-05
updated: 2026-09-05
type: query
tags: [meta, emergence, synthesis, dashboard]
confidence: high
---

# 涌现透镜轮换（Emergence Lens Rotation）

> 目的：把"透镜撞库"从偶发事件变成周期仪式。全库最深的两个产物——[[drafts/wiki-emergent-viewpoints-2026-07|2026-07 结构信号透镜]]与[[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 phd 七系统透镜]]——都是同一方法的产物：**拿一个透镜去撞存量，记录哪里撞出火花、哪里撞出矛盾、哪里两边都是空的**。本页是透镜清单与轮换状态，供 [[queries/vault-evolution-dashboard|进化仪表板]] 的 L4 原理层调度。

## 方法合约（每轮固定）

1. **选一个透镜**（不混用，一轮只透一个）
2. **撞库**：透镜 × 相关知识簇，重点记录三类发现——火花（新连接）、矛盾（页面间未对话的对立立场）、空白（两边都没有）
3. **定额产出**：1 篇 `drafts/wiki-emergent-viewpoints-YYYY-MM-<透镜名>` + 2~3 个 concept/comparison 补页
4. **验收**：涌现稿必须至少包含一条"本 wiki 此前不存在 X 页面/X 一等轴"的陈述并补齐；lint 0 error
5. **登记**：在下方轮换状态表记录日期与产量，log.md 留痕

## 透镜目录（7 个，按启动成本排序）

| # | 透镜 | 撞什么 | 预期火花 | 启动成本 |
|---|------|--------|----------|----------|
| 1 | **时间切片** | "2026-Q1 的判断，Q3 还成立吗"：取 90 天前同主题页 vs 近 90 天页 | 过时结论、被推翻的预测、 supersedes 链 | 低（纯存量） |
| 2 | **概念缺位** | 链接图：in-link 密集但无 concept hub 收拢的主题；跨 ≥5 entity 的窄标签簇 | 候选概念页清单（喂 [[moc/wiki-pending-concepts-roadmap]]） | 低 |
| 3 | **反方立场** | 让"最强批评者"读一个 MOC，写出 steelman 反方 | 每个 hub 页的立场光谱、缺失的"何时不要 X"页 | 低 |
| 4 | **跨簇迁移** | A 簇机制搬到 B 簇（如评测防火墙 → security / memory） | "X 簇的 Y 机制在 Z 场景是否成立"的 comparison 页 | 中 |
| 5 | **预测到期** | 带时间谓词的断言（"X 将在 Y 前取代 Z"）集中回访 | prediction ledger + 对/错判定（时间透镜的常量化） | 中 |
| 6 | **嵌入相似度** | qmd 嵌入找"语义近但无互链"的页对 | 该连未连的边、误判的重复页 | 中（需跑 qmd） |
| 7 | **外部系统** | 外部 repo/产品合集当探针（[[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|phd-lens 模式]]） | 一等设计轴（如"经验抽象度"）、wiki 缺的整页概念 | 高（需外部素材） |

## 轮换状态

| 日期 | 透镜 | 产出 | 备注 |
|------|------|------|------|
| 2026-07 | 结构信号（前身） | [[drafts/wiki-emergent-viewpoints-2026-07]] | 透镜制之前，方法验证轮 |
| 2026-08-29 | 外部系统（phd） | [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens]] + [[concepts/eval-optimizer-firewall]] 等 | phd 七仓库透镜 |
| 2026-09-05 | **① 时间切片** | [[drafts/wiki-emergent-viewpoints-2026-09-time-slice]] + [[concepts/claim-half-life]] + [[queries/prediction-ledger]]（种子 7 条） | 172 条时间断言撞库；首测半衰期：术语 30d / 代际 59d / 发布预期 109d；supersedes 空转定性 |
| 2026-09-05 | **② 概念缺位** | [[drafts/wiki-emergent-viewpoints-2026-09-concept-gap]] + [[concepts/world-models]] + [[concepts/agent-sandbox]] + 工具 concept-gap-detector.py | D1:4/D2:229/D3:133/D4:106；伪 hub 与跨目录孪生、标签-概念断线两结构发现；台账中期检视附带完成 |
| 2026-09-05 | **③ 反方立场** | [[drafts/wiki-emergent-viewpoints-2026-09-adversarial]] + [[concepts/when-not-to-harness-engineering]] + [[comparisons/model-capability-vs-harness-engineering]] + [[queries/negative-results-registry]] | harness 簇检验：厂商指控被驳回（1%）、同温层 49%、衰减推论缺席；5 条负结果入册；跨目录孪生同步清理 |
| 2026-09-05 | **④ 跨簇迁移** | [[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster]] + [[concepts/channel-enumeration-criterion]] + [[comparisons/information-isolation-vs-execution-isolation]] | 评测防火墙 × 沙箱：通道枚举判据合成；判分 2 成功 1 半成功 1 空白；对抗页三问登记台账 #8-#10 |
| 2026-09-05 | **⑤ 概念缺位·复测（度量轮）** | [[drafts/wiki-emergent-viewpoints-2026-09-metrology]] + 检测器升级（delta/history/D2 排除）+ 仪表板六指标基线 | D3 133→128（覆盖 5：实覆盖 2/重标签 2/借道 1）；新增缺口 0；合约 v2（度量轮免定额）；节奏分化决议 |
| — | （待跑：内容轮——外部系统透镜需备素材，或 roadmap 在册候选批量走 quality 轮） | | 下一轮占位 |

## 与其他机制的关系

- **矛盾升级线**（[[queries/vault-evolution-dashboard|见仪表板]]）是 L3 张力层的常驻管道；透镜轮是 L4 原理层的周期脉冲。四裁判为 `debate` 的对子可以直接成为透镜轮的素材。
- 透镜轮属于 WORKFLOW Phase 3 Evolve 的 **Frontier Track** 实例："vault 缺的不是更多笔记而是更好的研究议程"。
- L5 输出取材：月报/周报选题优先从最近的涌现稿与 debate 页取，形成"摄入→张力→原理→叙事"的完整梯度。
