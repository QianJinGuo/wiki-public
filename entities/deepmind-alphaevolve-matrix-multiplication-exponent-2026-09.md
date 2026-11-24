---

title: "DeepMind AlphaEvolve 再破纪录：矩阵乘法指数 40 年最低"
created: 2026-09-01
updated: 2026-11-22
type: entity
tags: [deepmind, alphaevolve, matrix-multiplication, optimization, ai-for-math, theoretical-cs, gemini]
sources: [raw/articles/deepmind-alphaevolve-matrix-multiplication-exponent-2026-09]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

## 概述

Google DeepMind 与理论计算机科学家联手，借助 Gemini 驱动的编码智能体 AlphaEvolve，将矩阵乘法指数上界刷新到 ω < 2.371177。这是 AI 第一次以实质性方式介入了理论计算机科学最经典、最顽固的开放问题之一。 ^[raw/articles/deepmind-alphaevolve-matrix-multiplication-exponent-2026-09.md]

## 核心突破

### 矩阵乘法指数

矩阵乘法是现代计算的"暗物质" —— 驱动从图形渲染到大规模神经网络训练的一切。速度由指数 ω 定义：两个 n×n 矩阵相乘的渐近复杂度是 O(n^ω)。 ^[raw/articles/deepmind-alphaevolve-matrix-multiplication-exponent-2026-09.md]

- 教科书算法：ω = 3
- 1969 年 Strassen：ω ≈ 2.807
- 此后半个多世纪：逐渐推到 2.371339
- **本次突破：ω < 2.371177**（改进幅度约 1.62×10⁻⁴，与过去 40 年的多数进步相当）

### 三板斧方法论

**第一板斧：重构**
- 组合损失分析（combination loss analysis）是激光法的精炼版本
- 重新形式化问题，支持递归层级 ℓ*=4，参数规模从 2.5 万跳到约 700 万 ^[raw/articles/deepmind-alphaevolve-matrix-multiplication-exponent-2026-09.md]

**第二板斧：机器学习设计的优化器**
- 梯度下降（Adam）+ Softmax 参数化 + Sinkhorn-Knopp 算法
- 在 700 万参数非凸空间里搜索，贡献约 0.97×10⁻⁴ 的改进 ^[raw/articles/deepmind-alphaevolve-matrix-multiplication-exponent-2026-09.md]

**第三板斧：AlphaEvolve 进化**
- 把优化器本身交给 AlphaEvolve 进化
- 单 GPU 大约 5 小时跑出结果，额外贡献约 0.63×10⁻⁴ 的改进 ^[raw/articles/deepmind-alphaevolve-matrix-multiplication-exponent-2026-09.md]

### 人机协作模式

- 人类提出框架、划定问题边界、设计验证路径
- AI 负责在巨大搜索空间中高效探索并进化解决方案
- 前纪录保持者 Josh Alman、Virginia Vassilevska Williams 共同署名

→ [[raw/articles/deepmind-alphaevolve-matrix-multiplication-exponent-2026-09|原文存档]] ^[raw/articles/deepmind-alphaevolve-matrix-multiplication-exponent-2026-09.md]

## 深度分析

### 不发明新框架，而是把优化核心推到极限

原文最反直觉的一点是：这次进步"没有发明全新的数学框架"。组合损失分析（激光法的精炼版本）本身早已存在，真正的障碍是规模——此前最好的结果只能处理递归层级 ℓ*=3、约 2.5 万个参数。团队的贡献在于重新形式化问题，配合张量表示、幽灵节点和分阶段并行，把一个原本难以扩展的非凸计算改造成 GPU 上可高效求解的形态，参数规模直接跳到约 700 万。这说明"问题的可计算形态"本身就是创新的载体：同一个数学框架，能否规模化往往比是否新颖更关键。

### 三层能力栈：重构 → 优化器 → 进化

突破可以拆解为一个递进的能力栈：第一层是重构（ℓ*=4、700 万参数的可行表示）；第二层是梯度下降（Adam）+ Softmax 参数化 + Sinkhorn-Knopp 最大熵计算 + 隐式微分构成的优化器，在非凸空间贡献约 0.97×10⁻⁴；第三层才轮到 [[entities/alphaevolve-deepmind-discovery-agent|AlphaEvolve]]——它进化的不是解，而是优化器本身，单 GPU 约 5 小时再贡献约 0.63×10⁻⁴。层级顺序很重要：没有前两层把搜索空间整理好，第三层的进化就没有可作用的"产品"。这提示我们，进化式智能体最大的杠杆点往往在流水线的上游环节，而非直接生成最终答案。

### "必须靠全新数学思想"这一判断需要重新校准

原文提出一个尖锐的追问：如果优化能力本身成为瓶颈，那么"必须靠全新数学思想才能突破"的判断是否需要重新校准？迷宫隐喻很准确——人类先设计出一座巨大的迷宫，AI 既学会在迷宫里找路，又学会改造迷宫本身的结构。过去 40 年里 ω 上界的多数进步幅度都在 10⁻⁴ 量级（本次约 1.62×10⁻⁴ 与之相当），这类"渐进式"进步很可能长期被低估：它们检验的其实是现有方法族的扩展性上限，而这正是搜索类 AI 的主场。

### 人机分工的边界在哪里

作者名单本身是信号：DeepMind 团队与前纪录保持者 Josh Alman、Virginia Vassilevska Williams 共同署名，成果被纳入人类学术验证体系而非另立门户。原文的冷静结论是：AlphaEvolve 没有发明新的激光法，真正的创造性框架依然由人类提出；[[entities/gemini-ai|Gemini]] 驱动的智能体是在人类划定的疆域内以前所未有的速度推进边界。但边界也是动态的——推动现有方法逼近天花板的过程已经不可逆地变成 AI 的主场，而真正接近 ω=2 的跃迁仍需人类提出全新思想。这一"渐进归 AI、跃迁归人类"的分野，与 [[concepts/scientific-method-ai-research|AI 驱动的科学研究方法论]] 中的分工模型高度一致，其外溢影响可参考 [[entities/alphaevolve-impact-deepmind|AlphaEvolve 的影响]]。

## 实践启示

1. **先问"是否已到规模极限"，再考虑发明新方法。** 面对顽固难题，检查现有框架的优化核心是否还能通过重新形式化（如 ℓ* 层级提升、参数规模跃升、GPU 化改造）获得收益——这往往是比全新框架性价比高得多的路径。
2. **把问题形式化成 AI 可作用的形态，是协作的先决条件。** 人类的价值不在与 AI 比拼搜索，而在提出框架、划定边界、设计验证路径，并把计算改造为张量表示 + 幽灵节点 + 分阶段并行这类"机器友好"的表示。
3. **把优化器本身当作可进化对象。** 当手动调优到瓶颈时，让进化式智能体去改进优化器（而非直接改进解），既能带来额外可测的改进（约 0.63×10⁻⁴），成本也出奇地低（单 GPU 约 5 小时）。
4. **渐进式数字同样有战略意义。** 1.62×10⁻⁴ 的改进与过去 40 年的多数进步相当——不要因数字"小"而低估其方法论含量：它验证的是整个方法族的扩展上限。
5. **AI 定位为"最强实验助手"而非替代者。** 在理论科学中，署名结构（前纪录保持者共同署名）表明 AI 成果应嵌入人类的验证与同行评审体系，而非绕开它。
6. **区分两类目标的归属。** 推动现有方法逼近天花板：已可交给 AI 主导；寻找范式级跃迁（如真正接近 ω=2）：仍需人类提出全新思想——资源配置应据此分轨。