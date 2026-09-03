---
title: Agent / Skill 评估方法对比
created: 2026-05-01
updated: 2026-05-01
type: comparison
tags: [comparison, meta, benchmark]
sources:
  - raw/articles/openai-evaluation-best-practices
  - raw/articles/anthropic-demystifying-evals-for-ai-agents
  - raw/articles/langsmith-evaluation-concepts
  - raw/articles/langsmith-trajectory-evals
  - raw/articles/anthropic-multi-agent-research-system
confidence: high
---
# Agent / Skill 评估方法对比
## 用途
这页是 vault 中**三个核心 Skill 的统一评估语言参考表**。它不是给某一 Skill 打分，而是定义每套评估方法论中的组件分别映射到哪个 Skill 的具体评估场景。
三 Skill 各自的评估页面：
- [[queries/wiki-evolver-skill-evaluation-methods]]（已落地：benchmark + scorecard + trajectory）
- [[queries/llm-wiki-evaluation]]（待落地）
- [[queries/web-content-reviewer-evaluation]]（待落地）
---
## Common Ground（所有 Skill 共享的评估原则）
1. 评估必须 task-specific，不是通用 benchmark
2. 真实失败样本需不断沉淀成测试集
3. 自动打分不够，需要 periodic human calibration
4. 结果（outcome）与过程（trajectory）都值得评估
5. offline benchmark 与 online monitoring 应形成闭环
---
## 统一评估组件对照表
以下 8 个组件构成 vault 的统一评估语言。每个 Skill 从中选取自己需要的子集。
| # | 组件 | 定义 | wiki-evolver | llm-wiki | web-content-reviewer |
|---|------|------|-------------|----------|---------------------|
| 1 | **Task definition** | 评估场景的 Prompt + 输入 + 合格标准 | ✅ 4 个 benchmark task | 待定（建议 3 个） | 待定（建议 4 个） |
| 2 | **Pass/fail gate** | 一票否决的门控条件 | ✅ 每个 task 有最低标准 | 待定 | 待定 |
| 3 | **Outcome rubric** | 对交付物质量的 0-2 分维度评分 | ✅ 5 维满分 10 | 待定 | 待定 |
| 4 | **Trajectory checklist** | 对执行过程的通过/失败检查 | ✅ 5 项 | 待定 | 待定 |
| 5 | **A/B comparison** | With/without Skill 的对照基线 | ✅ pilot 已跑 | 待定（可复用自然基线） | 待定（建议随机 3 篇抽样） |
| 6 | **Scorecard table** | 各任务分数 + 各维明细 | ✅ pilot 结果已录 | 待定 | 待定 |
| 7 | **Known weaknesses** | 当前已知弱点 + 缓解计划 | ✅ 4 项 | 待定 | 待定 |
| 8 | **Regression rules** | 新增失败案例如何回流 | ✅ 维护规则已定义 | 待定 | 待定 |
---
## 方法论来源 → 组件映射
这个表回答 "OpenAI 的方法贡献了这套评估语言的哪个部分"。
| 来源 | 贡献的组件 | 关键提取 |
|------|-----------|---------|
| **OpenAI eval best practices** | 1 (Task definition), 3 (Outcome rubric), 5 (A/B comparison) | Pairwise / criteria-based scoring, objective→dataset→metrics→loop |
| **Anthropic agent evals** | 1, 4 (Trajectory checklist), 6 (Scorecard), 7 (Weaknesses) | Task/trial/grader/transcript/outcome/harness 结构，结果与过程区分 |
| **LangSmith evaluation concepts** | 1, 8 (Regression rules) | Offline vs online 区分，curated examples 优先 |
| **LangSmith trajectory evals** | 4 (Trajectory checklist) | LLM-as-judge trajectory eval，适合灵活 agent |
| **Anthropic multi-agent research** | 7 (Weaknesses), 5 (A/B comparison) | 允许不同路径达同样好结果，早期小样本启动 |
---
## 评估目标分层
所有 Skill 的评估目标按粒度排列：
```
最细粒度：工具调用级正确性（当前 vault 不考虑）
    ↓
过程级：trajectory checklist（wiki-evolver 已用）
    ↓
交付物级：outcome rubric（wiki-evolver 已用）
    ↓
用户价值级：长期使用效果（当前 vault 不评估）
最粗粒度：业务指标影响（当前 vault 不评估）
```
当前 vault 聚焦在**交付物级 + 过程级**，跳过工具调用级和更粗的长期效果层。
---
## 怎么用这个表
**场景 1：跑完一次 wiki-evolver cycle → 用 wiki-evolver-skill-evaluation 的 rubric + checklist 打分**
**场景 2：跑完一次 llm-wiki ingest → 用 llm-wiki-evaluation 的 checklist 检查**
**场景 3：跑完一次 web-content-reviewer → 用 web-content-reviewer-evaluation 的 rubric 评分**
**场景 4：想给新的 Skill 建立评估方案 → 从这个表选取组件，在 comparison 页注册新行**
---
## 当前缺口
| 缺口 | 影响 | 优先级 |
|------|------|--------|
| llm-wiki 和 web-content-reviewer 还没有各自的评估页面 | 三 Skill 评估语言不统一，无法汇总比较 | P0（本 cycle 修复） |
| vault-evolution-dashboard 还没有评估汇总视图 | 无法一眼看到三 Skill 的评估状态 | P1 |
| 三 Skill 之间没有 cross-evaluation：llm-wiki 入库质量是否受 wiki-evolver 影响 | 缺少纵向比较数据 | P2 |
---
## 相关
- [[queries/wiki-evolver-skill-evaluation-methods]] — wiki-evolver 的具体评估实现
- [[queries/llm-wiki-evaluation]] — llm-wiki 的评估（待创建）
- [[queries/web-content-reviewer-evaluation]] — web-content-reviewer 的评估（待创建）
- [[queries/vault-evolution-dashboard]] — 汇总仪表盘
## 相关实体
- [[entities/hermes-agent-deep-dive|Hermes Agent]] — Self-Evolving Skill 机制
- [[entities/claude-code-architecture|Claude Code]] — Progressive Disclosure 渐进式披露机制
- [[entities/openclaw-leveraging-nova-mme-s3-vector-implement-skill|OpenClaw]] — ClawHub 社区 Skills 生态
## 相关概念
- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测框架]] — pass@k、评分器设计、CI/CD 集成
- [[concepts/skill-framework-writing-patterns|Skill 框架写作模式]] — Skill 设计影响评估方法论