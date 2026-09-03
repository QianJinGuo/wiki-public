---
title: Web 内容评审的标准与流程是什么？
created: 2026-05-01
updated: 2026-05-21
type: query
tags: [meta, benchmark, review]
sources:
  - raw/articles/openai-evaluation-best-practices
  - raw/articles/anthropic-demystifying-evals-for-ai-agents
confidence: high
---
# Web 内容评审的标准与流程是什么？

## 核心研究问题

如何系统性地评估 Web 内容并做出入库决策？评审流程中的质量标准如何定义？

## 一、Benchmark Suite（4 个核心任务）

### Task 1: Qualifying article（正确入库）
- **条件**：评分乘积 ≥ 49
- **产出**：Phase 1 评分 + Phase 2 入库（raw + entity + index/log + lint）
- **最低合格标准**：
  - raw 文件包含 source_url + ingested + sha256
  - entity 页面包含 review metadata（value/confidence/recommendation/stars）
  - index/log 更新 + lint 0 errors
- **典型失败**：≥49 但未入库、入库了但没有 review metadata、没有 strategic alignment check

### Task 2: Below-threshold article（正确拒绝）
- **条件**：评分乘积 < 49
- **产出**：Phase 1 拒绝 + 一行原因
- **最低合格标准**：
  - 明确说"未落库" + 原因
  - 不创建任何 wiki 文件
  - 原因具体（不是"价值不够"而是"新闻综述为主，缺技术细节/源码/架构"）

### Task 3: Non-extractable source（正确失败）
- **条件**：无法提取（body 为空，仅图片）
- **产出**：提取失败报告
- **最低合格标准**：
  - 明确报告 extraction failed，不评分
  - 不创建空文件
  - 如果尝试了 browser console 提取仍不行，说明原因

### Task 4: Strategic alignment（有对齐的入库）
- **条件**：文章与 vault frontier 对齐
- **产出**：入库 + frontier/backlog cross-link 更新
- **最低合格标准**：
  - 入库时检查了 strategic alignment
  - 如果对齐 → entity 页添加 strategic_context frontmatter + 更新 control page

## 二、评分维度（Scorecard 0-2 每维，满分 10）

| 维度 | 0 | 1 | 2 |
|------|---|---|---|
| **1. Scoring accuracy** | 评分与内容明显不匹配 | 评分基本合理但缺少论证 | 评分合理 + 论证具体（引用了内容细节） |
| **2. Threshold compliance** | ≥49 未入库 或 <49 入库了 | 阈值遵守但缺少明确状态声明 | 阈值遵守 + 明确状态声明（入库/未落库）+ 原因 |
| **3. Extraction correctness** | 从不可提取源创建空文件 | 正确判断但未记录方法 | 正确判断 + 记录 extraction_method |
| **4. Review metadata** | 未写 review metadata | 写了但字段不完整或格式错误 | 完整：value/confidence/recommendation/stars |
| **5. Strategic context** | 未检查 strategic alignment | 检查了但如果对齐未更新 control page | 对齐 → 更新 control page；不对齐 → 正常入库 |

**总分解释**：
- **9-10:** 完美审稿
- **7-8:** 良好，小缺陷
- **5-6:** 可用但缺关键项
- **0-4:** 需重做

## 三、Trajectory Checklist（通过/失败）

| # | 检查项 |
|---|--------|
| 1 | 提取是否成功尝试了二次（snapshot → console）？ |
| 2 | Phase 1 是否先评分后决策（乘积计算正确）？ |
| 3 | 如果 ≥49，是否运行了 strategic alignment check？ |
| 4 | Phase 2 是否完成入库 + 验证（bytes_written > 0）？ |
| 5 | 是否回复了 Phase 1 + Phase 2 的完整 closeout？ |

## 四、自然基线（Without Skill）

web-content-reviewer 的 "without skill" 基线：看到一篇好文章 → 凭感觉决定是否收藏 → 没有评分 → 没有入库规范 → 没有 cross-link。

**基线评分大约 2-3/10**（凭感觉可以做对一些，但不可复现，不可验证，不可累积）。

## 五、已知弱点与缓解

| 弱点 | 影响 | 缓解计划 |
|------|------|---------|
| scoring 依赖人工 judgment，不同 session 可能有偏差 | rubrics 不一致 | 用 scoring-rubric.md 标准化，定期校准 |
| extraction 流程对微信公众号依赖浏览器 | 有失败风险 | 备用提取方案（console + 视频链接） |
| strategic alignment check 只在 ≥49 时触发 | 可能漏掉边界值 | 已在 scoring-rubric 中明确，"49+必须检查" |

## 六、与兄弟 Skill 的关系

- 调用 llm-wiki 做实际入库 → 评估质量受到 llm-wiki ingest 质量的下限影响
- 被 wiki-evolver 作为来源采集层 → 评分质量决定了 vault 输入端的信噪比
- 三者共同构成统一评估语言 → [[comparisons/agent-skill-evaluation-methods]]

## 相关概念
- [[concepts/agent-evaluation-benchmark-frameworks]] — Agent 评测框架与 Benchmark 设计
- [[concepts/harness-engineering-framework]] — Harness 工程框架
- [[concepts/skill-formal-theory-survey]] — Skill 形式化理论：表示、执行、评估与进化

## 相关实体


## 相关
- [[comparisons/agent-skill-evaluation-methods]] — 统一评估语言
- Wiki Evolver Skill Evaluation（已删除） — wiki-evolver 的评估
- [[queries/llm-wiki-evaluation]] — llm-wiki 的评估
