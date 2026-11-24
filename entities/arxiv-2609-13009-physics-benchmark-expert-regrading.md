---
title: "How Good Are Frontier Models at Physics? Expert Re-Grading Reveals Broken Evaluations"
created: 2026-09-22
updated: 2026-11-14
type: entity
tags: [evaluation, benchmark, physics, llm, grading]
sources: [raw/articles/arxiv-2609-13009-physics-benchmark-expert-regrading]
confidence: 0.7
---

# How Good Are Frontier Models at Physics? Expert Re-Grading Reveals Broken Evaluations

## 摘要

Yale 团队联合多位物理学教员与研究生，对 6 个广泛使用的物理 benchmark 做了专家重评分审计：他们逐题审查题面、参考解与模型回答，把模型的真实错误与评分器故障、错误参考解、含糊题面区分开。结果显示，大多数被初判为"错误"的回答其实源于 benchmark 自身缺陷而非模型物理推理失能；修正评分后 GPT-5.6-Sol 的 mean@4 在 HLE-Physics 上从 47.3% 升至 78.7%，在 CMT-Benchmark 上从 61.0% 升至 87.2%，CritPt 保留子集上的 pass@4 达到 94.4%——主流物理 benchmark 已接近饱和，且严重低估前沿模型的真实能力。^[raw/articles/arxiv-2609-13009-physics-benchmark-expert-regrading.md]

## 核心要点

- 审计范围：HLE-Physics、CMT-Benchmark、CritPt、UGPhysics、PRISM-Physics、PHYBench 六个常用物理 benchmark，聚焦有可验证最终答案的纯文本题目。
- 失分主因错位：多数被初评为错误的案例，反映的是 benchmark 自身问题（评分器错误、参考解错误、题目歧义或欠约束），而不是模型的物理推理错误。
- 修正幅度巨大：GPT-5.6-Sol 的 mean@4 在 HLE-Physics 上从 47.3% → 78.7%，在 CMT-Benchmark 上从 61.0% → 87.2%；修正后的 pass@4 在 CritPt 的 54 道保留题上达 94.4%。
- 修正分数是在专家审查后保留的评估子集上计算的；UGPhysics、PRISM-Physics、PHYBench 的被审计子集修正后同样大幅上升，说明评分故障是跨 benchmark 的系统性现象。
- 之前榜单（含 Artificial Analysis Intelligence Index 2026 采用的物理基准）给出的低分，与领域专家日常使用这些模型的真实体感明显不符——低分是测量问题，不是能力问题。
- 闭式（closed-ended）题目已近饱和，继续刷这些榜的边际价值极低；论文呼吁构建更难、经专家验证的评估。
- 方法论要点：按物理子领域匹配教员/研究生专家，逐一核对题面、参考解与模型回答，区分"真错"与"判错"，再修正错误参考解、修复或剔除坏题。

## 深度分析

### 评估基础设施失效是系统性风险，不是个别榜单的瑕疵

这篇论文最重要的发现不是"模型其实更强"，而是：六个彼此独立的 benchmark 同时存在大比例的评分侧故障。故障来源可分成三类——评分器判分逻辑错误、参考解本身就错、题目表述含糊或欠约束导致无法唯一作答。当三类问题同时存在于多个榜单时，说明这不是某次出题事故，而是当前"快速造榜、直接发布"这一评估生产流程的通病：题目由人快速产出，参考解未经独立验证，评分管线也没经过专家校准。评估基础设施的可靠性，正在成为整个 eval 生态中最薄弱的一环。

### "低分"与专家体感的背离，暴露榜单叙事的脆弱

论文的动机来自一个矛盾：物理 benchmark 榜单上前沿模型分数很低，仿佛它们还不具备高等物理推理能力；但真正用这些模型做科研的物理学家，日常体验并非如此。作者没有停留在争论，而是用逐题审计把矛盾拆解到具体原因上——绝大多数"看起来不会"的案例，拆开看是"判错了"。这提示所有依赖单一榜单做结论的人（包括各类 Intelligence Index）：榜单分数是"模型 × 题目 × 参考解 × 评分器"四个变量的合成结果，任何一环坏了，分数就会系统性偏移，而读榜者无从察觉。

### 专家在环的修正流程：把评估当科学工作来做

这篇论文的方法值得当成模板：不是用另一个模型或自动管线去复评，而是按物理子领域匹配真正的领域专家（教员与研究生），对每个被评错的问题逐一审查题面、参考解与模型回答，区分模型真实错误与评估侧错误；然后做两件修复——把错误的参考解改对，把无法修复的坏题剔除或重写。修正后的分数在"保留子集"上重新计算，作者明确说明这一点以保证口径透明。这实际上是把科学界的同行评审机制引入了 benchmark 质量控制，成本高但置信度完全不同。

### 近饱和之后：闭式 benchmark 的边际价值归零

修正后 HLE-Physics 78.7%、CMT-Benchmark 87.2%、CritPt 保留子集 pass@4 94.4% 的水平意味着：这类有唯一可验证答案的闭式物理题，前沿模型已经接近做完。继续在此类题目上刷分，既区分不出模型差异，也测不出真实科研能力，反而会放大评分侧噪声对结论的影响。论文给出的方向是"更难、经专家验证的评估"——例如需要多步推导、开放性建模或与真实实验对照的题目。这与评估方法论的一个普遍趋势一致：闭式 benchmark 的角色正在从"能力测量"退化为"下限检查"。

## 实践启示

1. 不要把物理（以及类似的高难理科）benchmark 低分直接解读为"模型不会物理"——先问评分管线和参考解是否被独立审计过。
2. 引用任何榜单分数时注明口径：是原始全集还是修复后的保留子集，两者的数字可能相差 30 个百分点以上，不可混用比较。
3. 自建评估时，把"参考解正确性"当作与"题目难度"同级的质量门槛；上线前至少做一轮领域专家抽检，否则评分噪声会淹没模型差异。
4. 闭式可验证题已近饱和的领域，应把评估预算转移到开放式、多步、专家验证的题目上，而不是继续收集更多同类题。
5. 构建 benchmark 时可采用本文的审计动作集：区分真错/判错 → 修正参考解 → 修复或剔除坏题 → 在保留子集上重算分数，并公开子集口径。
6. 评估基础设施的可靠性本身就是需要持续投入的资产——修评分器的 ROI，往往高于换模型重跑一遍。

## 相关实体

- [[concepts/agent-evaluation-benchmark-frameworks]]
- [[concepts/evaluation-harness-design|评估 Harness 设计]]
- [[concepts/eval-surface-rotation|评估面轮换]]
- [[entities/model-evaluation-from-benchmark-worship-to-self-built-evals|从 Benchmark 崇拜到自建评估]]
- [[entities/multimodal-pretraining-physics-knowledge-flow-arxiv-2608-05000|物理知识流多模态预训练]]

→ [[raw/articles/arxiv-2609-13009-physics-benchmark-expert-regrading|原文存档]]
