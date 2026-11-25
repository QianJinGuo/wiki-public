---
title: Review Report
created: 2026-06-10
updated: 2026-11-25
type: query
tags: [review, quality, report, dashboard]
---

# Review Report

> 本页原先保存 2026-06 的一次性低质量实体清单，已不适用于当前 9,596 页的规模。保留为复审入口，具体名单以每日质检报告和实时质量仪表板为准。

## 当前快照

| 指标 | 数值 | 口径 |
|------|-----:|------|
| 活跃页面 | 9,596 | lint tracked pages |
| 实体页 | 4,495 | `entities/` |
| 有 review 覆盖 | 62.0% | fresh-checkup coverage |
| 已评分且 v×c ≥ 7 | 94.5% | 以已评分实体为分母 |
| 复审队列 | 245 | `reports/checkup-2026-11-25.md` |
| 矛盾四裁队列 | 60 | 同标签、评分差 ≥3 的候选对 |
| 结构错误 | 0 | lint errors |

## 怎么复审

1. 打开 [[queries/wiki-quality-dashboard|Wiki Quality Dashboard]] 看整体结构是否漂移。
2. 打开 `reports/checkup-2026-11-25.md` 的“复审队列”，按页面证据逐项三裁：keep / refresh / archive。
3. 需要补概念页时查 [[moc/wiki-pending-concepts-roadmap|概念补建路线图]]，不要重复造页。
4. 处理对立主张时先看 [[queries/negative-results-registry|负结果登记簿]] 与 [[queries/prediction-ledger|预测台账]]，再决定是否建 comparison。
5. 归档只走 `daily-checkup.py --archive-file`，保留 redirect 与回滚路径。

## 优先顺序

- **先修影响导航的错误**：断链、空入口、索引漂移。
- **再处理高价值但证据不足的页面**：优先补来源与边界，不先追求字数。
- **最后处理历史低分页**：新闻型、重复型和过期页面按批次裁决。

## 相关入口

- [[queries/review-queue|Review Queue]]
- [[queries/wiki-quality-dashboard|Quality Dashboard]]
- [[moc/wiki-panorama-navigation|知识库全景导航]]
- [[moc/wiki-master-map|知识库总导航]]
