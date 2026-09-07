---
title: "SkillCorpus: 大规模社区 Skill 生态的筛选、评测与边界分析"
description: "SkillCorpus 从 82 万社区 SKILL.md 经 6 阶段流水线提纯为 96,401 标准化技能，配套三级检索系统。三基准两框架评测：SkillsBench +7.5%，提升由覆盖度边界和 Harness 边界共同决定。"
created: 2026-07-24
updated: 2026-09-07
type: entity
tags: [skill-corpus, skill-curation, agent-skill, open-source, evaluation, retrieval, skillecosystem, skillbench]
sources: [raw/articles/skillcorpus-arxiv-2607-15557, raw/articles/skillcorpus-skill-screening-framework-mozhi-2026]
review_value: 8
review_confidence: 8
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# SkillCorpus: 大规模社区 Skill 生态的筛选、评测与边界分析

> 首个端到端框架：聚合开源 SKILL.md 生态，提纯为 96,401 标准化技能，在真实 Agent 任务上评测社区技能的实际价值并界定其边界。^[raw/articles/skillcorpus-arxiv-2607-15557.md]

## 概览

SkillCorpus 是由 EverMind、盛大集团与北京大学联合提出的框架，将松散的开源 SKILL.md 生态（~821,000 原始文件）经多层流水线提纯为 96,401 份合规、高质量、可商用的标准化技能，并配套微调检索排序堆栈，在真实 Agent 任务上评测了社区技能的实际增益与边界条件。^[raw/articles/skillcorpus-arxiv-2607-15557.md]

## 六阶段提纯流水线

1. **结构/格式检查**：标准 SKILL.md 格式 + 合理长度过滤
2. **两阶去重**：精确指纹去重（169,465 合并）+ 语义嵌入去重（cosine 0.90 阈值，LLM 裁决 66,751 边界对）→ 合计去除 64%
3. **三维质量打分**：LLM-as-judge 从 Utility（实用性）、Robustness（鲁棒性）、Safety（安全性）三维度输出 0-10 分
   - 综合分 = 0.85·content_q + 0.15·prior_src（安全薄弱时衰减）
4. **安全硬门禁 + 许可证过滤**：5 条硬规则（prompt_injection/cmd_injection/unsafe_exec/auth_bypass/csam_risk）→ 分数归零；仅保留 OSI 兼容许可证（去除 3,795 条）
5. **归类入库**：16 类分类法（Dev 22.4%, Data 14.1%, Writing 8.2%, DevOps-Infra 7.8%...），1024 维检索嵌入

## 三级检索排序堆栈

- **粗召**：Qwen3-Emb-0.6B（在去重后语料上微调），3000 字符检索字段
- **精排**：Qwen3-Rank-0.6B 微调排序模型
- **LLM 选择门**：阅读完整 Skill 正文，返回 0-2 条注入
- **可选查询改写**：领域术语规范化 ^[raw/articles/skillcorpus-arxiv-2607-15557.md]

## 评测结果

### 主实验（407 任务，24 配置 × 3 轮 = 74 次端到端运行）

| 框架 × 模型 | SkillsBench | GDPVal | QwenClawBench | 均值 |
|---|---|---|---|---|
| OpenClaw × Qwen-27B | +4.2 | +1.9 | +1.5 | +2.5 |
| OpenClaw × Qwen-397B | +5.8 | +1.8 | +1.3 | +3.0 |
| Raven × Qwen-27B | +6.5 | +1.2 | +3.9 | +3.9 |
| Raven × Qwen-397B | **+13.4** | +1.2 | +4.4 | **+6.3** |
| Claude Opus 4.7 | +8.0 | — | — | — |

全部配置正向增益，无净负均值（no-harm attachment）。最强单元（Raven × Q-397B）在 SkillsBench 上从失败中救回 19 个任务、损害 2 个（McNemar 检验 p<0.001）。

### 两个边界条件

**Harness 边界**：Raven 执行完整「推理→运行脚本→校验→修正」闭环，提升远超 OpenClaw（写代码后即终止、不校验）。Harness 执行逻辑直接影响 Skill 的落地效果。^[raw/articles/skillcorpus-arxiv-2607-15557.md]

**覆盖度边界**：高检索匹配 → 平均 +25.1%；中匹配 → +6.2%；低匹配 → +2.2%。Skill 库覆盖度直接调节增益幅度。^[raw/articles/skillcorpus-arxiv-2607-15557.md]

### 关键洞察

- **流程适配度 > 质量分数**：单任务成败取决于 Skill 流程与任务结构的匹配度，而非综合质量分
- **Skill 可能帮倒忙**：PPT 内嵌 Excel 修改任务中，通用 Skill（"打开 .xlsx"）无法处理 OLE 内嵌对象，反而比无 Skill 基线更差
- **高基线任务天花板**：写作等任务模型本身能力强，Skill 提升空间有限（GDPVal 仅 +1.2-1.9pp）
- **上下文隔离 > 并行**：规划器-执行者拆分的主要扩展优势来自上下文隔离，而非并行执行

## 局限与未来方向

- 质量评分依赖 LLM 文本判断，无沙箱执行验证
- 仅英文评测，中文场景尚未覆盖
- 静态快照（2026 Q2），无动态更新机制
- 高基线任务受限天花板效应 ^[raw/articles/skillcorpus-arxiv-2607-15557.md]

## 相关实体

- [[entities/skill-os-learning-skill-curation-self-evolving-agents|SkillOS: Learning Skill Curation for Self-Evolving Agents]]
- [[entities/skillcomposer-generative-skill-composition-agent|SkillComposer: 生成式技能组合]]

→ [[raw/articles/skillcorpus-arxiv-2607-15557|论文原文]] | [[raw/articles/skillcorpus-skill-screening-framework-mozhi-2026|中文解读]] | [PDF](assets/skillcorpus-arxiv-2607-15557.pdf)
