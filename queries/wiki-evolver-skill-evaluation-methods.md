---
title: Wiki Evolver 的 Skill 评测方法？
created: 2026-05-21
updated: 2026-05-21
type: query
tags: [meta, benchmark, evaluation, skill]
sources:
  - wiki/queries/wiki-evolver-skill-evaluation
confidence: high
---
# Wiki Evolver 的 Skill 评测方法？
## 核心问题
如何建立可执行的评估方案来判断 wiki-evolver Skill 是否真正产生增量？

## 一、Benchmark Suite（4 个任务）
每个任务都是真实的 vault-level 场景，跑一次 wiki-evolver cycle 时自动激活。

### Task 1: Frontier prioritization
**Prompt**: "从当前 vault 里选出最值得继续追的一个研究问题，并说明为什么"

**最低合格标准**:
- ✅ 选出的 frontier 可直接追溯到 vault 中至少 3 个现有页面
- ✅ 每个 frontier 项至少命名 1 个具体的 next actionable question
- ✅ 优先化理由是"当前 vault 在这个方向积累最厚，再往外推一步的杠杆最大"

**典型失败模式**:
- ❌ 选出一个全新 topic，与现存 vault 积累无关
- ❌ 只列方向不列问题
- ❌ 优先化依据是通用认知而不是 vault 特异性

### Task 2: Paper prioritization
**Prompt**: "从当前 backlog 里选出最值得写的一个 thesis，并补足缺失证据"

**最低合格标准**:
- ✅ 选中的 thesis 给出清晰的 claim（不是"总结一下 X"）
- ✅ 至少列出 3 个需要补充的证据 gap
- ✅ 每个 gap 有具体的手找路径（哪个页面/哪个来源/哪篇论文）

### Task 3: Practice operationalization
**Prompt**: "把一个理论主题推进成可执行 playbook / checklist / template"

**最低合格标准**:
- ✅ 产物不是分析而是可执行的命令式清单
- ✅ 至少包含 1 个具体可复用的模板片段（YAML / markdown / script）
- ✅ 可用一句话回答"这个 playbook 什么时候应该有人跑"

### Task 4: Dashboard update
**Prompt**: "把这轮 cycle 的结果写回控制面，明确下一轮该往哪里长"

**最低合格标准**:
- ✅ `index.md` 和 `log.md` 已更新（或明确说明无需更新）
- ✅ 至少更新一个 control page（frontier / paper / practice / dashboard）
- ✅ 明确写出一句话描述下一轮的推荐 leverage track

## 二、Scorecard（0-2 每维，满分 10）
| 维度 | 0 | 1 | 2 |
|------|---|---|---|
| **1. Outcome completeness** | 只有分析，没有 durable artifact | 有 artifact 但不完整 | artifact 完整，含 frontmatter + cross-links + sources |
| **2. Specificity & actionability** | 结论是"这个方向很重要" | 有优先级，但下一步不具体 | 有优先级 + 明确下一步 + 可执行动作 |
| **3. Navigation & governance** | 未更新 index/log | 更新了 index 或 log 之一 | index + log 都更新了 + lint 通过 |
| **4. Evidence grounding** | 结论没有 vault 页面引用 | 引用了页面但未区分主证据与辅助证据 | 每个 main claim 都有对应 vault 页面引用 |
| **5. Reusability** | 结果是一次性回复，不可复现 | 有记录但需人工 reinterpret | 结果可用作下次 regression 的 baseline |

**总分解释**:
- **9-10:** 强，可直接提交
- **7-8:** 良好，有改进空间
- **5-6:** 勉强可用，缺关键项
- **0-4:** 未达标

## 三、Trajectory Checklist（通过/失败，每道门必须过）
| # | 检查项 | 说明 |
|---|--------|------|
| 1 | 是否先 orient（读 SCHEMA / index / log）？ | |
| 2 | 是否明确选择了 leverage track？ | |
| 3 | 是否产生了至少一个 durable outcome？ | |
| 4 | 是否更新了 control pages？ | |
| 5 | 是否完成了 governance（index + log + lint）？ | |

**规则**: 任一 Fail → cycle 不完整。必须明确标注 Fail 项并提出修复计划。

## 四、Regression Suite 维护规则
1. **新增失败案例**：每次子任务失败时，将 vault 状态快照和错误输出加入 regression set
2. **任务变更**：当 vault 结构或 Skill 行为发生根本变化时，重写或替换 benchmark task
3. **评分校准**：每 10 次 cycle 后抽查 3 次历史结果，确认 rubric 没漂移
4. **CI 集成**：如果未来 wiki-evolver 跑得更频繁，把 4 个任务脚本化

## 五、已知弱点与缓解计划
| 弱点 | 影响 | 缓解计划 |
|------|------|---------|
| 单轮 pilot，非多 trial | 不能证明稳定性 | Phase 2 加 3 次以上重复 |
| 无自动化 grader | 评分有人为偏差 | 先保留人工 rubric，等 repeatability 稳定后加 LLM judge |
| 无线上数据回流 | 没有真实使用反馈 | 目前手动触发，先不做 online monitoring |
| 兄弟 Skill 未覆盖 | 评估语言不统一 | Phase 4 扩展 |

## 相关知识
- [[comparisons/agent-skill-evaluation-methods|统一评估语言来源]]
- [[queries/vault-evolution-dashboard|汇总仪表盘]]

## 相关实体


## 相关概念

- [[concepts/agent-evaluation-benchmark-frameworks|Agent Evaluation Benchmark Frameworks]]
- [[concepts/skill-framework-writing-patterns|Skill Framework Writing Patterns]]
