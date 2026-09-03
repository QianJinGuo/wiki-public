---
title: LLM Artifact Optimization — 文本/制品进化优化
created: 2026-05-28
updated: 2026-08-01
type: concept
tags: [llm-evolution, artifact-optimization, skill-evolution, self-improvement, asi, pareto-search, validation-gated, microsoft-research]
related:
  - [[entities/gepa-optimize-anything|GEPA optimize_anything]]
  - [[entities/darwin-skill-2-huashu|达尔文.skill 2.0]]
  - [[concepts/skill-formal-theory-survey|Skill 形式化理论]]
confidence: high
---

# LLM Artifact Optimization — 文本/制品进化优化

## 核心命题

LLM Artifact Optimization（文本/制品进化优化）研究如何让 LLM 自主改进其生成的文本制品——包括代码、prompt、Skill、架构配置等。与传统代码编译器不同，这里的"编译"对象是自然语言或结构化文本，优化的目标是让制品在特定 evaluator 下达到更优表现。

## 两大范式对比

| 维度 | GEPA optimize_anything | 达尔文.skill 2.0 |
|------|------------------------|-----------------|
| **优化对象** | 通用文本制品（代码/prompt/架构/配置） | Agent Skill（任务描述） |
| **核心机制** | ASI（Actionable Side Information）+ Pareto 前沿搜索 | SkillOpt 验证回滚 + SkillLens 评分 + 多评委共识 |
| **Human-in-loop** | ❌ 全自动 | ✅ CHECKPOINT 人工介入 |
| **验证门控** | 无（基于评分和 Pareto 选优） | ✅ 验证不通过坚决拒绝 |
| **评委机制** | 单一 evaluator | 两个独立评委 + 共识才通过 |
| **典型效果** | ARC-AGI Agent 32.5%→89.5% | 近 30 skill 平均 +15 分 |

## 技术共性

尽管实现不同，两者的核心设计哲学高度一致：

1. **诊断反馈 > 纯评分**：传统进化框架返回单一标量分数，告诉你"打几分"但不告诉你"怎么改"。GEPA 的 ASI 和达尔文的多评委本质都是将诊断信息升维——让 proposer 知道失败原因和修复路径，而非仅知道好坏。
2. **帕累托意识**：两个系统都拒绝简单的平均分聚合。GEPA 用 Pareto frontier 保留各维度非劣解，达尔文的多评委独立审查防止单一评委的偏见掩盖真实问题。
3. **验证作为守门机制**：SkillOpt 将"验证必须通过"作为硬约束，这与数值优化中"梯度必须降低 loss"的原则对称。优化的本质不是让分数变高，而是让制品在真实环境中更有效。

## 相关概念

- [[concepts/skill-formal-theory-survey|Skill 形式化理论]] — Skill 的表示、执行、评估与进化的理论基础
- [[entities/ai-skill-evolution-framework|AI Skill 进化框架]] — Skill 沉淀与持续优化的系统框架


## TODO
Think about User's Trajectory in data eval level
eval data -> knowledge domain 
And add scenario to User's hook

## 新增关联实体
- [[entities/lossy-self-improvement]]

## 关联实体

**上游依赖**:
- [[entities/gepa-optimize-anything]] — 提供基础理论/方法

**下游应用**:
- [[entities/darwin-skill-2-huashu]] — 具体应用场景

**平行协作**:
- [[entities/ai-skill-evolution-framework]] — 替代/补充方案
- [[entities/lossy-self-improvement]] — 替代/补充方案

## 深度分析

### 1. 优化目标的可验证性差异

LLM Artifact Optimization 与传统数值优化的根本区别在于「可验证性梯度」是否存在。传统 ML 优化中,loss 函数的梯度方向是明确的——你可以通过反向传播得到「往哪个方向调参能让 loss 下降」。但在 LLM 制品优化中,「改进」本身就需要 LLM 来判断——「这个 prompt 是否更好」「这个 skill 描述是否更清晰」这些判断的 ground truth 是模糊的。这导致两个系统的核心挑战完全不同：传统优化挑战是「找到全局最优」,LLM 制品优化挑战是「先定义什么叫更好」。

GEPA 的 ASI 和达尔文的 SkillOpt 本质上都在尝试构造可验证信号——前者通过失败案例提炼「修哪里」,后者通过 CHECKPOINT 阶段人工介入补足可验证性。这两种思路代表 LLM 制品优化的两个分支：**自动构造可验证信号** vs **人工 gate 兜底**。前者扩展性强但可信度依赖 evaluator 质量,后者可信度高但规模受限。

### 2. 制品类型与优化策略的对应关系

不同类型的 LLM 制品需要不同的优化策略,GEPA 的"optimize_anything"定位和达尔文的"skill 进化"定位本质上是在不同制品类型上做优化:

| 制品类型 | 代表 | 关键优化机制 | 评估成本 |
|---------|------|------------|---------|
| **Prompt** | GEPA 通用 prompt | Pareto 选优,无需 gate | 低(可批量 evaluator) |
| **Code** | AlphaEvolve | 执行验证 + 性能 profiling | 中(需 sandbox) |
| **Skill 描述** | 达尔文.skill 2.0 | SkillOpt + 多评委共识 | 高(需专家领域知识) |
| **Agent 架构配置** | OpenAI 自演进 | sandbox 内 end-to-end 跑通 | 极高(完整任务评估) |
| **RAG 检索策略** | Anthropic RAG 优化 | 离线 eval set 评分 | 中(需 ground truth) |

制品类型决定了 evaluator 的可用性,反过来决定优化策略的天花板。Code 制品有天然的执行反馈,优化空间大;Skill 描述的「好」与「不好」没有客观标准,优化空间被 evaluator 能力限制。

### 3. 与传统编译器的范式差异

把 LLM Artifact Optimization 类比编译器是常见的比喻——但这个比喻有重要边界:**传统编译器是 deterministic + verified,LLM 优化是 stochastic + heuristic**。传统编译器的输出是 bit-exact reproducible 的(同一段代码编译两次结果相同),而 LLM 优化每次跑可能得到不同结果(因为 evaluator 本身可能是 LLM 评判)。

这个差异的实际影响是:LLM 制品优化需要**重复性验证**——同一个制品多次评估,确认分数稳定。如果分数方差很大,优化本身就没有意义。达尔文.skill 2.0 的多评委共识机制本质上是降方差策略——用多个独立判断的平均降低单次评估的随机性。

### 4. 优化与创作的张力

LLM Artifact Optimization 的一个深层矛盾是「优化」与「创作」的张力。当我们对 prompt 做 Pareto 选优,我们倾向于保留那些 evaluator 评分高的版本——但 evaluator 评的是「在某些指标上表现好」,而不是「有创造性」。这导致 LLM 制品优化可能倾向于「平均化的好」而非「有特色的好」。

达尔文.skill 2.0 的 CHECKPOINT 机制有一个隐含价值:它在优化过程中保留人工 gate,而人工 gate 不仅是验证手段,也是创造性的兜底——人能在「evaluator 评分高但实际很平庸」时否决。完全自动化的 LLM 制品优化可能在长尾上失去多样性,这是值得警惕的范式风险。

## 所属 MOC

- [[moc/ai-skill-design|Ai Skill Design]]
