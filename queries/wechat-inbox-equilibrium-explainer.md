---
created: 2026-06-10
uid: wechat-inbox-equilibrium-explainer
title: 微信公众号 RSS 订阅的最佳实践
type: query
tags: [wechat, inbox, wiki-pipeline, best-practice]
related:
  - "[[WORKFLOW|wiki-pipeline]]"
  - "[[wechat-mp-rss-extractor]]"
updated: 2026-06-30
---

# 微信公众号 RSS 订阅的最佳实践是什么？

基于 wechat-inbox 的运行机制，以下是微信公众号 RSS 订阅的最佳实践：

## 核心机制：动态平衡不是故障

wechat-inbox 固定在 188 篇是**自然饱和点**，不是 bug：

| 指标 | 值 |
|------|-----|
| 每账号 RSS 返回 | 固定 10 篇 |
| 监控账号数 | 20 |
| 潜在总量 | 200 篇 |
| inbox 实际存量 | **188 篇**（≈ 94% 容量） |

三个机制共同维持平衡：
1. **固定 RSS 窗口**：we-mp-rss 每账号每次最多返回最近 10 篇
2. **已读标记**：`POST .../read?is_read=true` 后，该文章不再出现在后续 RSS 中
3. **异步全文补抓**：已读标记在全文抓取完成后才执行，确保内容不丢失

## 最佳实践

### 1. 监控账号数量的选择
- 20 个账号是当前最优配置——既能覆盖主要来源，又不超过 RSS API 的返回限制
- 账号过多会导致单账号获取量下降（每账号固定 10 篇上限）

### 2. 处理频率与吞吐量
- 每 20 分钟扫描周期，每次获取约 9 篇新文章
- 全文写入 inbox 后才标记已读——确保不丢失内容
- 平衡逻辑：新文章发布 → 填补已读空缺 → 总量维持 188

### 3. 质量过滤时机
- 使用 inbox-screener 在 promote 阶段评分（score ≥ 49 才入库）
- 14 天未 promote 的文章自动清理——保持 inbox 高质量流转
- 不达标但有独特洞察的文章可作为引用素材直接删除

### 4. 避免的做法
- **不要**手动清空 inbox——会破坏已读标记机制，导致重复抓取
- **不要**增加单账号的抓取频率——RSS API 有固定窗口限制
- **不要**积累过多待处理文章——超过 14 天会被自动清理

## 相关概念
- [[concepts/hermes-agent-skill]]
- [[concepts/agent-orchestration-patterns]]

## 相关实体

## See Also
- [[wechat-mp-rss-extractor|wechat-mp-rss-extractor]] — 微信公众号 RSS 提取器 skill
- [[WORKFLOW|wiki-pipeline]] — 整体 wiki 采集 pipeline
- [[inbox-screener|inbox-screener]] — inbox 评分入库 skill
