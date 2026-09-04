---
title: Vault Evolution Dashboard
created: 2026-06-10
updated: 2026-09-05
type: query
tags: [dashboard, quality, metrics, evolution, emergence]
---

# Vault Evolution Dashboard

> 最后更新: 2026-09-05 | 数据源: vault-metrics(latest.json) + wiki-contradiction-scan + 目录实测
> 口径: entities 总数含 ~124 个 type:redirect 重定向壳；concepts 含 learning/ 子目录 25 页

## 📊 规模快照 (2026-09-05)

| 层 | 数量 | 2026-06-10 基线 | 变化 |
|------|------|------|------|
| raw/articles | 4154 | 1832 | +127% |
| entities | 4178 | 2201 | +90% |
| concepts | 164 | 74 | +122% |
| comparisons | 32 | ~20 | +60% |
| queries | 48 | 52 | -8% |
| moc | 45 | 45 | 0 |
| drafts | 24 | 20 | +20% |

## 🧭 层级模型与当前瓶颈

涌现 = 让知识从下层向上层移动。各层当前状态：

| 层 | 承载 | 状态 | 瓶颈 |
|------|------|------|------|
| L1 事实层 | raw + entities | 厚（4154 + 4178） | 孤儿率 26.4%（1071 页 0 入链）、dup_suffix 618 |
| L2 概念层 | concepts + comparisons | 薄（164 + 32 vs 4178 entity） | **概念缺位**：高入链主题无 concept hub 收拢 |
| L3 张力层 | [!contradiction] + 裁决队列 | 机制已接通 | 队列消费速度：60 对待四裁，依赖每日 LLM 环节 |
| L4 原理层 | 涌现稿（design axes） | 偶发→轮换制 | 透镜轮频率，见 [[queries/emergence-lens-rotation]] |
| L5 叙事层 | drafts / 月报 / 周报 | 事件驱动 | 选题应改从 L3 队列 + L4 涌现稿取材 |

**判断（2026-09-05）**：摄入不再是瓶颈（日均 ~10+ raw 入库），L2 概念缺位与 L3 队列消费是当前两大瓶颈。每日质量闭环（vault-metrics → daily-checkup 三裁）只作用于 L1，向上合成依赖矛盾升级线与透镜轮。

## ⚡ 矛盾升级线（2026-09-05 接通）

```
wiki-contradiction-scan.py（每 12h）
  ├─ callout 校验：23 个 [!contradiction] 全部解析（2026-09-05 修复 13 断链 + 18 误报）
  ├─ 同 raw 立场冲突检测
  └─ 裁决队列：同标签簇 × 评分差≥3，窄标签过滤 → metrics/contradiction-queue.json
        ↓
daily-checkup.py 日报「矛盾四裁队列」节（每日 LLM 环节消费）
  ├─ debate        → 建 comparisons/ 对抗页 + 双向 contradicted_by
  ├─ complementary → 互加 cross-link，记录 metrics/contradiction-dismissed.jsonl
  └─ false         → 记录 dismissed（含孪生重复页对，转交去重）
```

当前队列：候选 1746 对 → 过滤后 60 对（已排除：已标记 / comparisons 已同台 / 已裁定）。

## 📏 质量指标 (vault-metrics 2026-09-05)

| 指标 | 数值 | 说明 |
|------|------|------|
| review_coverage | 66.3% | 有 review_value 的实体占比 |
| share ≥7 分 | 84.9% | 已评分实体的分值分布 |
| orphan_entities | 1071 (26.4%) | 0 入链口径（entities/concepts/moc 入链） |
| median in/out links | 2 / 6 | entity 层 |
| dup_suffix 孪生页 | 618 | `-N` 后缀疑似重复 |
| copy_paste_fingerprint | 1374 | 逐句脚注密度高的实体 |
| book_quarantined | 1957 | 公开投影隔离 |
| news_stale_90d | 138 | 新闻类过期 |

### 标签簇 Top 8（矛盾候选空间）

| 标签 | 实体数 | | 标签 | 实体数 |
|------|--------|-|------|--------|
| agent | 1293 | | aws | 390 |
| ai | 660 | | security | 350 |
| llm | 637 | | architecture | 348 |
| harness | 281 | | memory | 279 |

## 📜 历史里程碑

- **2026-06-09/10 五阶段建设**：mode-3 深度分析覆盖、修复 598 断裂 raw 引用、零链接 944→0、292 孤儿 raw 转 entity、302 stubs 扩写。详见 git log 与 log.md 2026-06 段。
- **2026-07 涌现观点①**：[[drafts/wiki-emergent-viewpoints-2026-07|结构信号透镜]]（矛盾扫描 + 链接统计撞库）。
- **2026-08 涌现观点②**：[[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|phd 七系统透镜]]，产出 [[concepts/eval-optimizer-firewall|评测防火墙]] 等补页与"经验抽象度"一等轴。
- **2026-09-05 升级线接通**：扫描器修复（callout 块解析 + 目标跨目录解析 + 评分差配对），四裁接入每日 checkup 日报，13 断链 callout 修复 + 5 双向回执。

## 🎯 下一步（按杠杆排序）

1. **四裁队列消费**：每日 LLM 环节产出 debate/complementary/false 判定，debate 落 comparisons 页（已在 checkup 日报中，无需新流程）
2. **透镜轮第一轮**：按 [[queries/emergence-lens-rotation]] 跑"时间切片"透镜
3. **概念缺位检测**：从链接图推导候选 concept 页清单，更新 [[moc/wiki-pending-concepts-roadmap]]
4. **prediction ledger**：抽取带时间预测的断言做到期回访（时间透镜的常量化）
5. **孪生页清理**：618 dup_suffix 与裁决队列暴露的重复对，走既有 `--archive-file` 流程

## 🔍 快速导航

- [[Dashboard|Wiki 主页]]
- [[queries/research-frontier-map|研究前沿地图]]
- [[queries/engineering-practice-backlog|工程实践待办]]
- [[queries/emergence-lens-rotation|涌现透镜轮换]]
- [[queries/wiki-quality-dashboard|质量仪表板]]
