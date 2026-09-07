---

title: "DeepMind AlphaEvolve 再破纪录：矩阵乘法指数 40 年最低"
created: 2026-09-01
updated: 2026-09-07
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