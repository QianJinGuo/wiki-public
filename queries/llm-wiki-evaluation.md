---
title: LLM-Wiki Evaluation
created: 2026-05-01
updated: 2026-05-21
type: query
tags: [meta, benchmark, wiki]
sources:
  - raw/articles/openai-evaluation-best-practices
  - raw/articles/anthropic-demystifying-evals-for-ai-agents
confidence: high
---

# LLM Wiki 的评测方法有哪些？

LLM Wiki 的评估方案分为四个层次，评测的是**一次 wiki ingest 的质量**，不评估整个 vault 的健康度：

## 一、Benchmark Suite（任务级评测）

### Task 1: Standard Article Ingest
- **输入**：标准格式 web 文章（HTML 可见正文、单作者、明确来源）
- **产出**：raw/article.md + entity/concept 页 + index/log 更新 + lint 通过
- **合格标准**：raw 文件有 source_url + ingested + sha256；合成页面有完整 frontmatter；最少 1 个出站 wikilink；index + log 已更新；lint 0 errors

### Task 2: Non-Standard Source
- **输入**：格式缺失源（图片为主的微信公众号，或纯 API 返回的 JSON）
- **产出**：同 Task 1，但记录 extraction_method
- **合格标准**：正确判断源是否可提取；raw 文件添加 extraction 字段；失败时写 log 不创建空文件

### Task 3: Duplicate / Boundary Check
- **输入**：vault 已有相关内容的文章
- **产出**：确认是否创建新页或合并到已有页
- **合格标准**：入库前搜索 vault 确认无重复；高度重合→合并；部分重叠→创建+cross-link

## 二、Scorecard（0-2 每维，满分 10）

| 维度 | 0 | 1 | 2 |
|------|---|---|---|
| **Raw source quality** | 缺少 sha256 或 source_url | 全字段但 extraction 未记录 | 全字段 + extraction_method |
| **Synthesis quality** | 没有合成页 | 有但缺 cross-links 或 sources | 完整 frontmatter + ≥2 cross-links + sources |
| **Navigation completeness** | index 或 log 未更新 | 只更新了其中之一 | 两者都更新 + lint 通过 |
| **Boundary accuracy** | 入库与现有内容冲突/重复 | 边界正确但 cross-link 缺失 | 边界正确 + 恰当 merge/cross-link |
| **Extraction correctness** | 从不可提取源创建空文件 | 正确判断但记录不全 | 正确判断 + 完整记录 |

**总分解释**：9-10 完美入库 / 7-8 良好 / 5-6 可用 / 0-4 需重做

## 三、Trajectory Checklist（通过/失败）

| # | 检查项 |
|---|--------|
| 1 | 入库前是否 orient（读 SCHEMA/index/log）？ |
| 2 | 是否先搜索 vault 确认无重复实体页？ |
| 3 | 是否按 layer model 写入了正确目录（raw vs entities vs concepts）？ |
| 4 | 是否完成了 governance（index + log + lint）？ |

## 四、自然基线（Without Skill）

llm-wiki 的基线是"文件的原始问题记录在 raw/ 中，从未分类入库"：
- 没有 sha256 → 文件漂移无法检测
- 没有合成页面 → 跨文章 insights 不会交叉
- 没有 index.md 条目 → 不知道自己看过什么
- 没有 log → 不知道 vault 变动历史

**天然基线几乎为 0/10**——它是 vault 存在的操作性前提。

## 五、与兄弟 Skill 的关系

- `` 调用 llm-wiki → llm-wiki 入库质量直接影响 wiki-evolver 可用的 vault 状态
- web-content-reviewer 调用 llm-wiki → 评分质量与入库质量互相依赖
- 三者共同构成统一评估语言 → [[comparisons/agent-skill-evaluation-methods]]

## 相关概念
- [[concepts/agent-evaluation-benchmark-frameworks]] — Agent 评测框架与 Benchmark 设计
- [[concepts/harness-engineering-7-layers-framework]] — Harness 工程七层框架
- [[concepts/skill-formal-theory-survey]] — Skill 形式化理论：表示、执行、评估与进化

## See Also
- Wiki Evolver Skill Evaluation（已删除） — wiki-evolver 的评估
- [[queries/web-content-reviewer-evaluation]] — web-content-reviewer 的评估
- [[comparisons/agent-skill-evaluation-methods]] — 统一评估语言
- [[queries/paper-backlog]] — 哪些 AI/ML 论文最值得深入研读
