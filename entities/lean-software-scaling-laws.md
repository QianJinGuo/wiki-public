---
title: "Lean Software Scaling Laws"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/lean-software-scaling-laws]
provenance_state: extracted
---

> -> [[raw/articles/lean-software-scaling-laws.md|原文存档]]

sha256: 7d18a440117de0f1a01d54b8555ff36660db0a8095620c987a516b8fda7125a4 ^[raw/articles/lean-software-scaling-laws.md]

## 摘要

这是 Gwern 的一篇研究提案：通过实证测量编码 LLM 的困惑度（perplexity/BPC）随代码库上下文规模增长的 scaling 曲线，来估计不同编程语言的"可预测性"指数，并以此判断形式化语言是否值得大规模投入 ^[raw/articles/lean-software-scaling-laws.md]。核心假设是：Python 这类弱约束语言在小规模（数千行）下容易预测但 scaling 指数更差，而 Lean 虽然因现有语料稀少而有很高的基线常数和更大的总损失，却应拥有更好的 scaling 指数，最终会在某个代码库规模上交叉反超 ^[raw/articles/lean-software-scaling-laws.md]。作者认为关键洞察在于：设计良好的代码库"越看越可预测"，而坏代码"越看越不可预测"，因此 LLM 预测准确率是系统可预测性的弱代理指标 ^[raw/articles/lean-software-scaling-laws.md]。文章给出了一套学生或 MATS 项目即可完成的廉价测量方案，并论证若交叉点假设成立，将为大规模用 Lean 重写软件、提升全球网络安全水平提供依据 ^[raw/articles/lean-software-scaling-laws.md]。

## 关键要点

- 测量流程五步：按语言构建单一语料文件 → 用冻结的预训练 LLM 测量全上下文窗口内逐 token 位置的困惑度（按字节归一化以消除 tokenizer 和行长度差异） → 按位置平均 → 每语言拟合 scaling 曲线 → 外推寻找交叉点
- 交叉验证手段包括：向上下文注入细微 bug（缺边界检查、符号错误、静默 dtype 转换等）观察 LLM 困惑度是否升高；人为限制上下文看多少采样 rollout 仍能编译并通过测试；检验 LLM 能否仅凭 Lean 模块签名预测出正确实现
- 作者预测：弱类型语言在数千行规模更易预测，但在数十万至百万行规模会被 Haskell 等强约束语言交叉超越；Lean 因常数项过高可能不会在任何合理长度上于绝对损失上反超，但会拥有最好的 scaling 指数
- 偏差控制方案：先按主题配对比较代码库；更强的做法是合成受控对比——用两种语言写同一 spec 到同等实测质量再测困惑度（如原版 C zlib 对 Lean zlib）
- 最可能的失效模式是生态成熟度与语料先验效应主导语言级不变量；若如此，则说明至少现阶段工具、约定和文档比形式化语言属性对 Agentic 编程更重要

## 来源

- 原文：[[raw/articles/lean-software-scaling-laws.md|Lean Software Scaling Laws]]
