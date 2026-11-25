---
title: Vault Evolution Dashboard
created: 2026-06-10
updated: 2026-11-25
type: query
tags: [dashboard, quality, metrics, evolution, emergence]
---

# Vault Evolution Dashboard

> 2026-11-25 快照。数据来自 lint、`reports/checkup-2026-11-25.md` 与目录实测。这里的“层”是知识成熟度，不是文件夹层级。

## 规模

| 层 | 当前数量 | 作用 | 主要债务 |
|----|---------:|------|----------|
| L1 来源与实体 | 4,749 raw + 4,495 entities | 记录事实、案例、产品与人物 | 491 个实体孤儿、93 个推断段落 warning |
| L2 概念与对比 | 178 concepts + 36 comparisons | 把重复现象压缩成可复用判断 | 概念/实体比约 1:25，仍偏薄 |
| L3 张力与裁决 | 60 对候选 | 把冲突从“各说各话”变成可裁决问题 | 队列消费依赖逐对阅读 |
| L4 维护与查询 | 55 queries | 仪表板、路线图、负结果与预测 | 多个 6–9 月视图已过期 |
| L5 对外叙事 | 38 drafts | 综合、发布与复用 | 只占活跃页面 0.4%，输出仍偏少 |

## 当前瓶颈

1. **导航曾比内容更差**：MOC 中大量 wikilink 被历史编辑截断，Lint 因只统计成对链接而没有报错。2026-11-25 已修复全部 MOC，并新增 malformed wikilink 检查。
2. **概念层没有跟上实体层**：4,495 个 entities 只有 178 个 concepts。大量文章型实体在扮演概念页角色，重复解释同一机制。
3. **孤儿主要是来源层**：1,994 个 true orphan 中，1,503 个是 raw sources、491 个是 entities。前者说明“抓到了但未合成”，后者说明“合成了但没接入网络”。
4. **复审覆盖仍不足**：review coverage 62.0%，意味着约四成实体没有足够评分依据进入质量判断。
5. **重复与 redirect 有历史成本**：750 个 `-N` 后缀疑似重复、1,324 个 redirect stubs。它们影响检索和链接图，但不能靠批量删除解决。
6. **推断与来源混杂**：98 个 lint warnings 中绝大多数是历史页面的未引用推断段落，应随重写自然下降。

## 健康判断

- **结构健康**：0 lint errors；索引、frontmatter、broken links、citations 全部通过。
- **导航健康**：45 个 MOC 已无截断 wikilink，moc/ 与 queries/ 无孤儿。
- **内容健康**：可用，但概念层偏薄、复审覆盖不足，不能把 0 errors 当成“知识都已验证”。
- **发布健康**：公开投影有 2,716 个 book-quarantined 项，说明发布边界已被认真执行，而不是全库裸奔。

## 2026-11-25 已完成

- 重建 [[moc/wiki-master-map|知识库总导航]]，从单页清单改为按学习顺序和主题域进入。
- 重建 [[moc/wiki-panorama-navigation|知识库全景导航]]，用当前规模和维护债务替代 6 月数字。
- 修复 19 个 MOC 页面中的截断链接，并复核全部 45 个 MOC；补上关闭 `]]` 与错误目录前缀。
- 把 `review-queue`、`review-report`、`wiki-topic-clusters`、`vault-evolution-dashboard` 从历史快照更新为当前维护入口。
- 在 `scripts/wiki-lint.mjs` 增加 malformed wikilink 检查，防止同类问题复发。
- 把 Syncthing 冲突副本、`.stfolder/` 与一次性 `scripts/tmp_*` 加入 `.gitignore`。

## 下一轮优先级

1. **补概念层**：从 [[moc/wiki-pending-concepts-roadmap|概念补建路线图]] 选 5 个高入链缺口，而不是继续增加新闻型 entity。
2. **连接高价值孤儿实体**：优先给长篇、可复用、技术机制类页面补 1–2 个上下文入链。
3. **消费矛盾四裁**：60 对候选逐批处理，真正对立的升级为 comparison。
4. **清理重复后缀**：先做标题/正文指纹，再合并；不要只按 `-N` 机械删除。
5. **提升 review coverage**：优先补 2026 下半年新增的未评分页面。

## 导航

- [[moc/wiki-master-map|知识库总导航]]
- [[queries/wiki-quality-dashboard|Quality Dashboard]]
- [[queries/review-queue|Review Queue]]
- [[queries/wiki-topic-clusters|主题聚类]]
- [[moc/wiki-pending-concepts-roadmap|概念补建路线图]]
