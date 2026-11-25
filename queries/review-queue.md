---
title: Review Queue
created: 2026-06-10
updated: 2026-11-25
type: query
tags: [review, quality, queue, maintenance]
---

# Review Queue

> 当前权威队列由每日质检生成，不再手工维护 2026-06 的 302 篇模板化清单。实时结论见 `reports/checkup-2026-11-25.md`，整体健康见 [[queries/wiki-quality-dashboard|Wiki Quality Dashboard]]。

## 当前状态

| 项目 | 数量 | 说明 |
|------|-----:|------|
| 复审候选 | 245 | 每日质检的三裁队列：keep / refresh / archive |
| 前 150 项中的 bypass 项 | 5 | 评分低于 Gate，先重评分再裁决 |
| book-quarantined 项 | 其余大多数 | 公开投影隔离，不等于内容无价值 |
| 矛盾四裁 | 60 | 同标签、评分差 ≥3 的候选对 |
| 结构错误 | 0 | 链接、索引、citation 与 frontmatter 基线 |

## 处理顺序

1. **先看结构错误**：0 errors 是底线，出现断链先修。
2. **再处理 `bypass_score_below_gate`**：评分不可信，先重评 v×c。
3. **处理 `book_quarantined`**：核对公开投影边界，区分“来源不可公开”与“内容质量差”。
4. **最后做 archive**：只有明确重复、过期、无复用价值时才归档，并保留 redirect。
5. **矛盾对单独处理**：按 debate / complementary / false 四裁，不混入普通质量队列。

## 页面级判断

- **keep**：有明确复用计划、结构完整、来源可核验。
- **refresh**：主张有价值，但证据、时效、边界或结构不足。
- **archive**：重复、过期、无受众、无复用路径，且归档后不损失独有信息。
- **debate**：两页对同一问题给出相反判断，升级为 comparison 并双向回填。

## 入口

- [[queries/review-report|Review Report]] — 当前复审快照与流程。
- [[queries/wiki-quality-dashboard|Quality Dashboard]] — lint、v×c、孤儿与结构债务。
- [[moc/wiki-pending-concepts-roadmap|概念补建路线图]] — 该补的 concept，不要用新 entity 重复覆盖。
- [[queries/negative-results-registry|负结果登记簿]] — 有对照的失败证据优先保留。
- [[queries/prediction-ledger|预测台账]] — 过期预测需要回访，不进入普通归档。

## 边界

- 不为降低队列数字而批量删页。
- 不为清零 warning 而批量添加伪引用。
- 不把 `book_quarantined` 等同于低质量。
- 不绕过 `daily-checkup.py --archive-file` 手工移页。
