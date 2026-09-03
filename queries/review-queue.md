---
title: Review Queue
created: 2026-06-10
updated: 2026-06-10
type: query
tags: [review, quality, queue]
---

# Review Queue

> 需要复审/改进的实体和内容。按优先级排列。参见 [[ai-coding-agent-quality-defense-five-control-mechanisms]] 的质量控制方法论。

## 🔴 高优先级: 模板化扩写需精炼

以下 302 篇实体由模板自动扩写，需要基于原文内容进行更深入的技术分析：

### 识别标准
- 深度分析部分包含通用模板句式 ("涉及X领域的核心技术议题")
- 核心观点提取不够具体
- 实践启示过于泛化

### 改进方法
1. 阅读对应 raw 全文
2. 提取 3-5 个具体技术洞察
3. 补充与相关实体的深度对比
4. 添加领域特定的实践建议

## 🟡 中优先级: 标签清理

- `uncategorized` 标签实体需重新分类
- 标签粒度不统一 (如 `agent` vs `agentic-ai`)
- 缺少领域标签的实体

## 🟢 低优先级: 内容增强

- 添加更多交叉引用 (当前平均 5.8 links/entity)
- 补充代码示例和架构图
- 增加与 wiki-life 的跨库关联

## 📋 自动化建议

1. 创建 cron job 定期扫描新 raw 文章
2. 自动生成 v×c 评分并低于阈值时加入 review queue
3. 每周生成质量报告
