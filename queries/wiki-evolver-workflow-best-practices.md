---
title: Wiki Evolver 的工作流程与最佳实践？
created: 2026-05-21
updated: 2026-05-21
type: query
tags: [meta, workflow, skill, evaluation]
sources:
  - wiki/queries/wiki-evolver-cycle-2026-05-01
confidence: high
---
# Wiki Evolver 的工作流程与最佳实践？
## 核心问题
Wiki Evolver 如何通过 cycle 机制实现知识库的可持续改进？其工作流程与最佳实践是什么？

## 当前瓶颈判断
**最高杠杆瓶颈不再是 ingest，而是 evaluation。**

当前知识库已具备：
- ingest / index / log / lint 闭环
- topic map / review queue / health dashboard
- frontier / paper / practice / dashboard 四层控制面
- `wiki-evolver` 首版 Skill 规范

**缺失**：如何判断 Skill 比"没有 Skill 的一般执行"更好 → 需升级为"可验证、可回归、可持续改进"

## Frontite Decision（最高优先级）
### 问题
如何把 Skill / Harness 的评估从 prompt-level judgment 提升到 system-level evaluation？

### 选择理由
- vault 已有足够多关于 Skill、Harness、Context、Governance 的材料
- 缺少统一方法评估这些能力是否真的产生增量
- 这一层补上后，`wiki-evolver`、`web-content-reviewer`、`llm-wiki` 都能受益

## Paper Decision
### 主题
**Skill 是 procedural memory，而 evaluation 是其 governance layer**

### 选择理由
- 直接连接 `agent-skill-writing`、`skill-craft`、`wiki-evolver`
- 能把"Skill 不只是 Prompt"推进成更完整的工程论点
- 有明确实践落点，不会停留在纯概念层

## Practice Decision
### 主题
为 `wiki-evolver` 建立离线 evaluation harness 与 regression suite

### 最小可执行集合
1. 4 个当前 vault 任务构成的 benchmark suite
2. with/without skill 的对照条件
3. outcome rubric
4. lightweight trajectory checklist
5. 每次 Skill 变更后的回归运行

## Dashboard Decision
### 增长指标转变
不应以"新增多少页"为主指标，而应关注：
- 能否稳定选出高杠杆 frontier
- 能否从 frontier 推出 paper candidate
- 能否把 paper / principle 转成 practice / Skill
- 能否通过 eval 确认 Skill 真的在变强

## Next Round 规划
1. 先把 `wiki-evolver-skill-evaluation` 的 benchmark suite 固化成长期 regression set
2. 再把同样的方法扩展到 `llm-wiki` 与 `web-content-reviewer`
3. 回头补 `research-frontier-map` 里排在第二梯队的问题（context-as-working-set、harness-as-policy）

## 相关概念
- [[concepts/agent-evaluation-benchmark-frameworks]]
- [[concepts/harness-engineering-framework]]
- [[concepts/skill-formal-theory-survey]]
- [[concepts/hermes-agent-skill]]

## 相关实体

## 相关知识
- Wiki Evolver Skill Evaluation（已删除）
- [[queries/research-frontier-map|Research Frontier Map]]
- [[comparisons/agent-skill-evaluation-methods|Agent Skill 评估方法对比]]
