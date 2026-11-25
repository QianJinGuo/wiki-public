---
title: 知识库全景导航
type: moc
created: 2026-06-18
updated: 2026-11-25
tags: [moc, navigation, index, meta, health]
description: 知识库的规模快照、导航入口、维护债务与整理优先级。
---

# 知识库全景导航

> 2026-11-25 快照。这里描述“库现在是什么样”，不是知识结论本身。逐页目录见 `index.md`，推荐阅读顺序见 [[moc/wiki-master-map|知识库总导航]]。

## 规模

| 层 | 页面数 | 说明 |
|----|-------:|------|
| `entities/` | 4,495 | 文章型实体、产品、公司、人物与案例 |
| `concepts/` | 178 | 跨文章复用的概念与方法 |
| `comparisons/` | 36 | 明确对峙或选型对比 |
| `queries/` | 55 | 仪表盘、专题判断与维护视图 |
| `moc/` | 45 | 有顺序的阅读路径 |
| `drafts/` | 38 | 对外输出与综合叙事 |
| `raw/articles/` | 4,749 | 原始来源存档 |
| 合计 | 9,596 | lint 的活跃页面口径 |

## 怎么逛

| 目标 | 入口 | 说明 |
|------|------|------|
| 从零建立全局感 | [[moc/wiki-master-map|知识库总导航]] | 先看分层，再按主题下钻 |
| 系统学习 | [[moc/learning-path|AI 知识体系学习路径]] | 按学习顺序而不是按文件夹浏览 |
| 找某个主题 | `index.md` → 对应 MOC | `index.md` 是全量目录，MOC 是精选路径 |
| 查当前质量 | [[queries/wiki-quality-dashboard|Wiki Quality Dashboard]] | lint、v×c、孤儿页与告警 |
| 处理待办 | [[queries/review-queue|Review Queue]] | 复审、补来源、重写与归档候选 |
| 看演化瓶颈 | [[queries/vault-evolution-dashboard|Vault 演化仪表板]] | L1 事实层到 L5 叙事层 |

## 当前结构信号

以下数字来自 2026-11-25 的 lint 与日常质检，只作为整理优先级，不代表页面价值：

| 信号 | 当前值 | 判读 |
|------|-------:|------|
| Lint errors | 0 | 结构底线正常 |
| Lint warnings | 98 | 主要是历史页面的推断段落未附来源 |
| True orphans | 1,994 | 其中 raw 1,503、entities 491；不等于都应删 |
| 实体中位入链 | 2 | 长篇实体多，但网络连接仍偏中心化 |
| 复审队列 | 245 | 需要逐页三裁，不宜批量处理 |
| 矛盾四裁队列 | 60 | 需要逐对判 debate / complementary / false |
| `-N` 后缀疑似重复 | 750 | 需要去重与 redirect 治理 |
| Redirect stubs | 1,324 | 属于迁移成本，不是知识增长 |
| 公开投影隔离 | 2,716 | 发布边界，不等于质量差 |

## 整理优先级

1. **先修导航**：MOC 的断链、过期数字和空入口，比继续增加页面更影响可用性。
2. **再处理矛盾**：优先消费 `metrics/contradiction-queue.json`，把真正对立的主张升级为 comparison。
3. **最后清孤儿**：优先给高价值实体补上游 MOC 或 concept 回链；低价值新闻页按复审流程归档，不做机械全连。
4. **不追求“全库无警告”**：历史推断段落应随重写自然下降，不应为了清零而批量添加伪引用。

## 导航边界

- `index.md` 是完整性清单，不承担阅读顺序。
- `moc/` 只放“顺序有意义”的路径。
- `queries/` 放可验证的判断、仪表板与维护视图。
- `drafts/` 放已经综合到可以对外表达的内容。
- `raw/articles/` 是不可变来源，不应为了降低孤儿率而反向改写。
