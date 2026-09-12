---
type: entity
title: d-OPSD —— 扩散语言模型的在线自蒸馏框架
created: 2026-07-09
updated: 2026-09-11
tags: [diffusion-llm, self-distillation, on-policy, post-training, knowledge-distillation]
sources: [raw/articles/d-opsd-第一个针对扩散语言模型的在线自蒸馏学习]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# d-OPSD —— 扩散语言模型的在线自蒸馏框架

## 摘要

d-OPSD（On-policy Self-distillation for Diffusion LLMs）是马普所联合清华大学提出的首个面向扩散语言模型（dLLMs）的在线自蒸馏后训练范式，论文为《Learning from the Self-future: On-policy Self-distillation for dLLMs》。它让学生先在线采样，再把学生自己生成的"未来"（self-future）随机保留、回灌给教师作为特权信息，把 OPSD 的密集监督彻底 on-policy 化，并将监督粒度从 token 提到扩散解码的 step。在四个数学推理 benchmark 上，它以约 RL 十分之一的训练步数取得更优表现，且全面优于传统"参考解注入"式 OPSD。^[raw/articles/d-opsd-第一个针对扩散语言模型的在线自蒸馏学习.md]

## 核心要点

- **填补空白**：OPSD 在自回归 LLM 上已成熟，但在 dLLMs 上此前完全空白，d-OPSD 是首个成体系的方案。
- **密集监督胜过稀疏奖励**：OPSD 依靠逐 token 的密集监督，效果与训练效率被报告为超越 RL，这是它被引入 dLLM 后训练的根本理由。
- **摆脱参考解注入**：传统 OPSD 把参考解塞进教师 prompt，会诱发"参考解幻觉"——学生默认答案已被提供，自己反而答不对；d-OPSD 取消参考解，改由学生自我采样产生特权信息。
- **self-future 作教师特权信息**：利用 dLLM 任意顺序生成的特性，把学生采样轨迹中后续的"未来"片段随机保留并反馈给教师。
- **on-policy 贯彻到底**：以往 OPSD 只有轨迹生成由学生驱动、特权信息仍是 off-policy 静态参考解；d-OPSD 让特权信息本身也由学生生成。
- **监督层级升到 step**：针对迭代式解码特性，把监督粒度从单 token 提到一次解码步，提供更贴合生成过程的密集信号。
- **教师-学生差异调到合适区间**：教师恰好握有学生不知道的新知识，又与学生共享足够共同语言，避免参考解注入下 overlap 过大、压缩蒸馏上限。
- **效率与效果双赢**：RL 需上千步收敛，d-OPSD 仅需数百步（约 1/10），并在四个数学推理 benchmark 的大部分任务上推理表现更好。

## 深度分析

### 扩散语言模型为何是后训练的特殊战场

自回归 LLM 的生成是串行左到右的：一次前向产出一个 token 的分布，监督信号天然对齐到"位置"。扩散语言模型（dLLMs）把文本当作可并行去噪的离散序列，在若干解码步内反复精化整条序列，token 之间没有强制的左到右顺序。这种并行去噪、双向上下文、非左到右生成的特性，让 dLLM 拥有"先粗后细、可重排"的自由度，却也让后训练失去清晰的"位置即监督"映射轴：RL 的轨迹级奖励不知该归因给哪一步去噪，蒸馏的逐 token 分布又对应不上一轮并行生成的多个位置。d-OPSD 的立足点，正是承认这种结构差异并为之重设监督轴。^[raw/articles/d-opsd-第一个针对扩散语言模型的在线自蒸馏学习.md]

### On-policy 自蒸馏：用密集监督取代稀疏奖励

OPSD 相对 RL 的吸引力在于信号密度。RL（尤其 RLVR 一类）通常在整条回答结束后才给一个标量奖励，梯度要穿越整个生成过程回传，信号稀疏、方差大、样本效率低，因而常需上千步收敛；在线自蒸馏则能在每个生成单元上给出教师-学生的分布差异，把"整条答案对不对"降解为"这一步该怎么走"。d-OPSD 继承了这一效率优势，却拒绝继承自回归 OPSD 的特权信息构造方式：它不提供参考解，而让学生自己的采样结果充当教师"看见未来"的信息来源，使蒸馏目标从模仿外部答案变为拟合模型自身在更优条件下的输出。^[raw/articles/d-opsd-第一个针对扩散语言模型的在线自蒸馏学习.md]

### 核心机制：self-future 特权重构

d-OPSD 的机制可概括为"教师构建方式的重构"：学生先在线采样得到轨迹，从中随机保留一部分学生自我的"未来"片段，再把这段未来作为特权信息回灌进教师输入。教师因此站在"已看到学生将要写什么"的位置上给出更可靠的分布，学生则在仅有 prompt 的条件下向教师对齐。该设计把 dLLM"任意顺序生成"的特性变成优势——因为不存在固定左到右因果顺序，未来片段可在不破坏因果一致性的前提下被切出回灌，"看到未来"在 dLLM 里是自然操作，在自回归模型里几乎自相矛盾。由此教师多出的正是学生尚不具备的那部分信息，而非外部强模型带来的、与学生语言习惯脱节的全新分布。^[raw/articles/d-opsd-第一个针对扩散语言模型的在线自蒸馏学习.md]

### 从 token 到 step：扩散解码的信用分配问题

把监督层级从 token 抬到 step 是 d-OPSD 的明确贡献。对 dLLM 而言，一次解码步并行确定多个位置，逐 token 监督既要处理并行性，又要面对"同一步内 token 谁对谁错"的信用分配难题：一个错误 token 可能源于上一步的错误假设，而非本步决策失误。以解码步为单位组织蒸馏信号，等于让监督轴与生成机制对齐——每个 step 各自对齐师生分布，信用归属单位与模型真正做决策的单位一致，密集信号因此更精准、更少被噪声稀释。这也是它能在数学推理这类对中间步骤高度敏感的任务上取得增益的结构性原因。^[raw/articles/d-opsd-第一个针对扩散语言模型的在线自蒸馏学习.md]

## 实践启示

1. **dLLM 后训练优先考虑蒸馏而非 RL**：任务有可验证答案且对中间步骤敏感时（数学推理是典型），step 级密集监督的效率远优于稀疏奖励，先跑 OPSD 往往比先调 RL 划算。
2. **警惕参考解注入的副作用**：把标准答案当特权信息喂给教师会让学生默认"答案已被给出"；若在自回归 OPSD 中观察到这类退化，可改用学生自我采样构造特权信息。
3. **特权信息不必来自外部**：self-future 表明教师的相对优势可完全由学生自身轨迹构造，无需更强外部教师，也无需额外蒸馏数据管线。
4. **教师-学生差异是需调优的超参数**：overlap 过大会限制蒸馏上限，完全脱节又会让教师"说学生听不懂的话"；随机保留比例即是调节这一落差的直接旋钮。
5. **按生成机制决定监督粒度**：并行去噪用 step 级监督，因果串行用 token 级监督；把监督单位对齐到模型决策单位，是可迁移的第一原则。
6. **仍待回答的问题**：本次存档未展开蒸馏目标的数学形式（如与 MaxEnt RL、reverse-KL 的关联），也缺向更大规模 dLLM 的扩展验证与推理步数预算-质量权衡结论。

## 相关实体

- 论文：<https://arxiv.org/abs/2606.18195>；代码：<https://github.com/xingzhejun/d-OPSD>
- 方法对照：[[entities/on-policy-distillation-vs-offline-distillation-loster|On-policy vs Offline 蒸馏]]、[[entities/xopd-on-policy-distillation-landscape-banana-2026|xOPD 蒸馏全景]]、[[entities/exploring-self-distilled-reasoning-for-supervised-fine-tunin|自蒸馏推理与 SFT]]
- 扩散语言模型：[[entities/llada2-2-agentic-diffusion-model-ant-2026|LLaDA2.2]]、[[entities/diffusiongemma-4x-faster-text-generation-google-2026-06|DiffusionGemma]]、[[entities/acl-2026-diffusion-lm-block-size-reasoning-t-star|扩散 LM 块大小与 T*]]、[[entities/lave-lookahead-then-verify-diffusion-lm-constrained-decoding-issta-2026|LAVE 约束解码]]、[[entities/diffusion-model-consistency-framework-2026-survey|扩散一致性综述]]
- 概念：[[concepts/model-distillation-compression|模型蒸馏与压缩]]、[[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 可验证推理]]、[[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026|LLM RL 算法演进]]、[[concepts/harness-engineering-framework|Harness Engineering]]
- → [[raw/articles/d-opsd-第一个针对扩散语言模型的在线自蒸馏学习|原文存档]]
