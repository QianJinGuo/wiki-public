---
title: 主题聚类快照
created: 2026-06-15
updated: 2026-11-25
type: query
tags: [query, topic-cluster, meta, analysis, taxonomy]
---

# 主题聚类快照

> 2026-11-25 按 `entities/` frontmatter 实测。3,163 个实体至少有一个标签；其余实体主要靠标题、正文和 MOC 归类。这里是结构信号，不是完整分类法。

## 高频标签

| 实体数 | 标签 | 主要含义 |
|-------:|------|----------|
| 1,034 | `agent` | Agent 架构、运行时、工作流与产品 |
| 453 | `ai` | 泛 AI 产业与应用 |
| 446 | `llm` | 模型能力、训练与推理 |
| 313 | `aws` | AWS 服务、部署与案例 |
| 302 | `harness` | Agent harness 与运行时 |
| 242 | `architecture` | 系统架构与工程结构 |
| 229 | `security` | 攻击面、防御、供应链 |
| 220 | `evaluation` | 评测、基准与验证 |
| 213 | `claude-code` | Claude Code 生态 |
| 182 | `memory` | 记忆、上下文与状态 |
| 181 | `multi-agent` | 多 Agent 协作 |
| 180 | `anthropic` | Anthropic 产品与战略 |
| 180 | `wechat` | 微信公众号来源 |
| 172 | `open-source` | 开源项目与生态 |
| 165 | `harness-engineering` | Harness 工程方法 |

## 高频共现

| 共现数 | 标签对 | 判读 |
|-------:|--------|------|
| 203 | `agent` + `llm` | Agent 文章默认以模型能力为背景 |
| 201 | `agent` + `harness` | Harness 已成为 Agent 工程的主导框架 |
| 151 | `agent` + `architecture` | 说明“架构”仍是最常用的问题入口 |
| 136 | `agent` + `memory` | 记忆是 Agent 子主题中连接最强的一支 |
| 129 | `agent` + `evaluation` | 评测与 Agent 工程高度绑定 |
| 129 | `ai` + `wechat` | 来源标签与话题标签混用，需注意 |
| 122 | `agent` + `aws` | 企业部署案例集中在 AWS |
| 108 | `aws` + `bedrock` | Bedrock 是 AWS AI 文章的主轴 |
| 98 | `agent` + `security` | Agent 安全已成为独立问题域 |
| 93 | `agent` + `harness-engineering` | Harness 从标签升级为方法 |

## 结构判读

1. **`agent` 仍是过宽标签**：1,034 个实体共用一个标签，无法单独承担导航。应通过 MOC 和更窄标签（`harness`、`memory`、`evaluation`、`multi-agent`）分流。
2. **来源标签污染话题统计**：`wechat`、`newsletter`、`article` 描述来源而非主题，不应与前排技术标签同权。
3. **概念缺口集中在标签已成形但无 concept hub 的主题**：候选包括 knowledge management、inference optimization、computer vision 等，详见 [[moc/wiki-pending-concepts-roadmap|概念补建路线图]]。
4. **MOC 比标签更适合作为导航**：标签只回答“出现过什么”，MOC 回答“先读什么、后读什么”。

## 使用方式

- 找主题入口：[[moc/wiki-master-map|知识库总导航]]。
- 找概念缺口：[[moc/wiki-pending-concepts-roadmap|概念补建路线图]]。
- 看质量与孤儿：[[queries/wiki-quality-dashboard|Quality Dashboard]]。
- 看层级瓶颈：[[queries/vault-evolution-dashboard|Vault Evolution Dashboard]]。

## 复算

当前统计按实体 frontmatter 的 `tags:` 字段去重计算，不做同义词合并。标签本身没有统一别名表，因此 `harness` 与 `harness-engineering`、`claude` 与 `claude-code` 仍分开计数。后续若引入标签别名，应先把映射写入 SCHEMA，再重算本页。
