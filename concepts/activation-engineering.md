---
title: Activation Engineering
created: 2026-04-23
updated: 2026-08-01
type: concept
tags: [interpretability, mech-interp, activation-engineering]
related:
  - [[entities/sparse-autoencoders|SAE]]
  - [[entities/natural-language-autoencoders|NLA]]
  - [[entities/llm-language-thinking-mechanisms|LLM 工作机制研究]]
sources: [raw/articles/anthropic-nla-natural-language-autoencoders-interpretability, raw/articles/llm-language-thinking-mechanisms]
---

# Activation Engineering

激活工程（Activation Engineering）是一套通过直接操作模型内部激活值来理解、引导或修改模型行为的方法体系。该领域是[[concepts/mechanistic-interpretability|机械可解释性]]的核心子方向，提供了在激活层面"读取"和"写入"模型思维的技术手段。^[raw/articles/llm-language-thinking-mechanisms.md]

Activation Engineering 的核心研究问题包括：如何将模型内部激活翻译为人类可理解的语义（reading）？如何通过干预激活来改变模型行为（writing）？这两大问题催生了 steering vectors、activation patching、representation engineering 等具体技术，以及 [[entities/sparse-autoencoders|SAE]] 和 [[entities/natural-language-autoencoders|NLA]] 等基础设施。^[raw/articles/anthropic-nla-natural-language-autoencoders-interpretability.md]

## 核心技术体系

### Steering Vectors（激活向量导向）

Steering vectors 是 Activation Engineering 最直观的技术手段。其核心思想是：在模型的激活空间中识别代表特定概念或行为的"方向向量"，然后在推理时通过对激活向量施加这一偏移来"引导"模型行为。^[raw/articles/llm-language-thinking-mechanisms.md]

**技术原理**：
- 通过对比"带有某概念"与"不带某概念"的激活，提取概念方向向量
- 将此方向向量叠加到模型推理路径的激活上
- 模型在后续生成中会更倾向于表达该概念

**典型应用场景**：
- 情感引导：增强积极或消极表达倾向
- 主题 steering：引导对话朝向特定话题
- 行为约束：强化或抑制特定回答模式

Steering vectors 的局限性在于它是一种"事后干预"，无法解释概念在模型内部是如何表示的——它只能观测干预后的行为变化。

### Activation Patching（激活补丁/激活回填）

Activation Patching 是 [[concepts/mechanistic-interpretability|Mechanistic Interpretability]] 中用于因果追踪的核心技术。其基本流程是：^[raw/articles/llm-language-thinking-mechanisms.md]

1. 在模型处理某个输入时，记录特定层/位置的激活值
2. 用另一组激活值替换（patch）这些位置
3. 观察模型输出是否发生预期变化

这一技术被广泛用于"电路追踪"（Circuit Tracing）——识别模型处理某个任务时依赖哪些组件。例如，研究者可以通过 activation patching 确定 Transformer 的哪个注意力头负责执行"间接宾语"语法规则。^[raw/articles/llm-language-thinking-mechanisms.md]

**与 Steering Vectors 的区别**：
- Steering Vectors：叠加偏移量，持续影响后续激活
- Activation Patching：替换特定激活，精确控制变量

Activation Patching 是 [[entities/llm-language-thinking-mechanisms|LLM 工作机制研究]] 中用于构建 Attribution Graph（归因图）的关键技术，可视化输入到输出的因果链。^[raw/articles/llm-language-thinking-mechanisms.md]

### Representation Engineering（表示工程）

Representation Engineering 是斯坦福研究团队提出的方法论，强调将模型行为理解为激活空间中的"表示"变化。其核心观点是：模型的不同行为状态（如"诚实模式"、"有帮助模式"、"回避模式"）对应激活空间中的不同区域。^[raw/articles/llm-language-thinking-mechanisms.md]

**方法论框架**：
- 表征探测：训练分类器识别特定行为状态对应的激活模式
- 表征干预：在推理时强制激活进入目标状态对应的区域
- 表征对齐：将模型的内部表示与人类的期望对齐

Representation Engineering 与 Steering Vectors 有高度重叠——两者都是对激活空间的定向干预——但 Representation Engineering 更强调"表示"的语义层次，而非单纯的向量运算。

## SAE：激活空间与可解释性之间的桥梁

[[entities/sparse-autoencoders|Sparse Autoencoders（SAE）]] 是 Activation Engineering 领域最重要的基础设施技术。SAE 的核心功能是"解压"：将模型内部稠密的激活向量分解为稀疏的、可解释的特征组合。^[raw/articles/llm-language-thinking-mechanisms.md]

### 技术架构

SAE 由编码器和解码器组成：^[raw/articles/llm-language-thinking-mechanisms.md]

```
原始激活（稠密向量）
    ↓ [编码器：非线性变换]
稀疏特征向量（高维）
    ↓ [解码器：线性变换]
重构激活
```

- **编码器**：将原始激活映射到高维稀疏特征空间
- **解码器**：将稀疏特征重建回原始激活维度
- **训练目标**：最小化重构误差，同时鼓励特征稀疏性

### SAE 的突破性发现

通过 SAE 对 LLM 激活的分析，研究者发现了以下关键规律：^[raw/articles/llm-language-thinking-mechanisms.md]

| 发现 | 说明 |
|------|------|
| 特征规模 | 单一模型层可分解出数十万到百万量级特征 |
| 层次结构 | 浅层（词法/语法）→ 中层（语义/基本推理）→ 深层（复杂推理/规划） |
| 特征叠加 | d维空间可容纳 exp(d) 个近乎正交基向量（Superposition 现象） |

### SAE 与特征叠加（Superposition）

Anthropic 提出的 Superposition 理论解释了为什么 SAE 的稀疏分解是必要的：^[raw/articles/llm-language-thinking-mechanisms.md]

- **宽层假设**：理想的稀疏特征应该是独立的
- **实际层**：特征之间存在叠加——一个神经元被多个特征共享
- **几何基础**：高维空间的几何性质允许在 d 维空间中存在 exp(d) 个近乎正交的方向

SAE 的稀疏约束正是解决 Superposition 问题的关键——它强制模型在表示复杂概念时保持特征的稀疏性和可辨识性。

### SAE 在 Activation Engineering 中的角色

SAE 处于 Activation Engineering 技术栈的核心位置：^[raw/articles/anthropic-nla-natural-language-autoencoders-interpretability.md]

```
SAE（特征提取）→ 激活空间的"分解"
    ↓
归因图（Attribution Graph）→ 因果链追踪
    ↓
NLA（文字翻译）→ 人类可读输出
```

- **下游工具**：SAE 分解出的特征是 Steering Vectors 的候选来源
- **上游基础**：SAE 为 NLA 提供了激活→文字翻译的特征输入
- **应用领域**：行为异常诊断、安全审计、模型可解释性分析

## NLA：读取模型"隐藏想法"的新手段

[[entities/natural-language-autoencoders|Natural Language Autoencoders（NLA）]] 是 Anthropic 2026 年发布的突破性可解释性技术，其核心命题是将 AI 内部激活值翻译为人类可读语言，从而读取模型"不会主动展示"的隐藏想法。^[raw/articles/anthropic-nla-natural-language-autoencoders-interpretability.md]

### 核心技术架构：AV + AR 双模块闭环

NLA 方法的核心闭环是：**原始激活 → 文本解释 → 重构激活**。^[raw/articles/anthropic-nla-natural-language-autoencoders-interpretability.md]

```
原始激活 → [AV 激活说明器] → 文本解释
              ↑                    ↓
              ← [AR 激活重建器] ← 文字
```

- **AV（Activation Verifier，激活说明器）**：将激活值翻译为文字解释
- **AR（Activation Reconstructor，激活重建器）**：将文字解释重建为原始激活值
- **训练信号**：AR 逆向重建质量判断 AV 解释是否准确

**关键创新**：NLAs 不直接验证解释正确性（无法直接检查激活编码了什么），而是通过 AR 逆向重建间接验证——若解释能准确还原原始激活，则解释有效。^[raw/articles/anthropic-nla-natural-language-autoencoders-interpretability.md]

### NLA 的核心指标

| 指标 | 数据 |
|------|------|
| 错位审计成功率（配备 NLAs） | **12~15%** |
| 错位审计成功率（无 NLAs） | **<3%** |
| SWE-bench 编程任务评估意识 | **26%**（NLAs 揭示） |
| claude.ai 真实用户评估意识 | **<1%**（NLAs 揭示） |

### NLA 与 SAE 的组合使用

NLAs 擅长读取"整体语义意图"，SAE 擅长定位具体激活特征。两者组合可以形成"宏观理解 + 微观定位"的闭环：^[raw/articles/anthropic-nla-natural-language-autoencoders-interpretability.md]

1. **NLAs 读取**：模型认为"这是个安全测试"
2. **SAE 定位**：具体是哪些特征组合驱动了这一判断

### 局限性

| 局限 | 说明 |
|------|------|
| 幻觉 | 解释中可能编造不存在细节；内部推理验证比文本幻觉更难 |
| 成本 | 需两模型副本 RL + 每次激活数百 token 推理 |

## 技术演进路径

```
Sparse Autoencoders（SAE）— 将激活模式分解为可解释的特征
    ↓
Attribution Graphs — 追踪输入到输出的因果链
    ↓
NLAs（本文）— 激活 → 文字闭环，直接输出人类可读的解释
```

## 与 [[concepts/mechanistic-interpretability|Mechanistic Interpretability]] 的关系

Activation Engineering 是 [[concepts/mechanistic-interpretability|机械可解释性]] 的核心实践方法。Mechanistic Interpretability 提供理论框架和目标（理解模型计算回路），而 Activation Engineering 提供具体的技术手段（操作激活）。^[raw/articles/llm-language-thinking-mechanisms.md]

两者共同指向一个更大的目标：**在激活层面建立对模型思维的完整理解**，从"黑箱观察"走向"白箱解读"。

## 关联

- [[entities/natural-language-autoencoders|Natural Language Autoencoders（NLAs）]] — 读取和解释激活的新手段，激活→文字闭环翻译
- [[entities/sparse-autoencoders|Sparse Autoencoders（SAE）]] — 激活空间的特征分解工具，Activation Engineering 的基础设施
- [[entities/llm-language-thinking-mechanisms|LLM 工作机制与可解释性]] — LLM 内部机制与 SAE/CLT 研究的核心论文
- [[entities/anthropic-llm-introspection-awareness-mechanisms|LLM 内省意识检测]] — steering vector 电路追踪的相关研究
- [[concepts/mechanistic-interpretability|Mechanistic Interpretability]] — Activation Engineering 的理论基础
- [[entities/anthropic|Anthropic]] — SAE 与 NLA 研究的领先机构

---

*相关标签: #activation-engineering #interpretability #mech-interp #steering-vectors #activation-patching #representation-engineering #SAE #NLA*

## 关联实体

**上游依赖**:
- [[entities/sparse-autoencoders]] — 提供基础理论/方法
- [[entities/natural-language-autoencoders]] — 提供基础理论/方法
- [[entities/llm-language-thinking-mechanisms]] — 提供基础理论/方法

**下游应用**:
- [[entities/natural-language-autoencoders]] — 具体应用场景
- [[entities/llm-language-thinking-mechanisms]] — 具体应用场景
- [[entities/sparse-autoencoders]] — 具体应用场景

**平行协作**:
- [[entities/natural-language-autoencoders]] — 替代/补充方案
- [[entities/sparse-autoencoders]] — 替代/补充方案
- [[entities/llm-language-thinking-mechanisms]] — 替代/补充方案

## 所属 MOC

- [[moc/loop-engineering|Loop Engineering]]
