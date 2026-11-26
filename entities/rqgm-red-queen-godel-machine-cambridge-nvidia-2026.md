---
title: "Red Queen Gödel Machine (RQGM)：共进化 Agent 与评估器的递归自改进框架"
created: 2026-06-28
updated: 2026-11-26
type: entity
tags: [self-improvement, godel-machine, co-evolution, red-queen, evaluator, agent-as-judge, recursive-self-improvement, cambridge, nvidia, arxiv-2606.26294]
sources: [raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026]
confidence: 0.9
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Red Queen Gödel Machine (RQGM)：共进化 Agent 与评估器的递归自改进框架

Cambridge + NVIDIA (2026-06-24) 提出的进化框架，解决自改进 Agent 的根本性局限：评估标准假设是固定的，但进化要求评估器也一起进化。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md]

## 核心问题

现有自改进 Agent 假设评估标准是平稳的（stationary）：固定的 verifier、benchmark、标注数据集。但这违背了进化的核心规律：物种不是在静止环境中优化自己，而是和不断变化的环境一起改变。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md]

## 历史脉络

| 阶段 | 方案 | 局限 | ^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md]
|------|------|------|
| Gödel Machine (2003, Schmidhuber) | 数学证明自修改有益 | 计算成本不可行 |
| Darwin/Huxley Gödel Machine | 进化替代证明（突变→测试→生存） | 评估器静态不变 |
| **RQGM (2026)** | 评估器和 Agent 共进化 | 本文贡献 |

## 核心机制：受控效用进化 (Controlled Utility Evolution)

RQGM 将搜索组织为 epoch：
1. **epoch 内**：评估器冻结 → 信号稳定 ^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md]
2. **epoch 边界**：效用可以更新
3. **新评估器**必须在留出的锚点数据上统计意义上打败老评估器
4. **选择性擦除**：只丢掉被替换评估器打过的分，保留其他证据

这确保了自改进保证在每个 epoch 内成立，同时目标跨 epoch 演化。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md]

## 实验结果

**代码编写 (Polyglot)**：
- 通过率：69.9% → 71.7%（+1.8% over SOTA）
- Token 效率：省 1.35-1.72x
- 机制：agent-as-judge 代码审查信号（比多轮测试便宜）

**科学论文写作**：
- 接收率：21.8% → 40.5%（1.78-1.86x 提升）
- 共进化写手 + 多样化 agent-as-judge 评审团

**奥赛级数学证明**： ^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md]
- 评分器准确率：比静态基线高 9%
- 搜索成本：降低 3x

**论文评审（偏见修正）**：
- 问题：最强基线审稿人过度接受 AI 论文（1.91x 人类接受率）
- 解法：对抗性目标发现对 AI 和人类同样严格的审稿人
- 结果：平等对待，同时保持 80% 真值准确率

## 关键意义

1. **Novel 框架**：首次将评估纳入递归自改进循环
2. **实用结果**：代码、论文、数学多领域强改进
3. **偏见修正**：修复 LLM 过度接受 AI 生成内容的倾向
4. **效率提升**：通过更智能的评估显著节省 token
5. **理论基础**：用进化原则扩展 Gödel Machine 概念

## 深度分析

### 从静态评估器到"军备竞赛"：为什么共进化是必然

现有自改进 Agent 的搜索方法普遍假设评估标准是平稳的（stationary）：固定的 verifier、benchmark 或标注数据集在 agent 不断变强的过程中依然有效。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:21-21] 但这恰恰是最大的隐患——评估器一旦固定，就会成为整个改进循环中的瓶颈和被"攻破"的目标：agent 会优先学会迎合固定裁判的偏好，而不是真正提升能力，这正是 reward hacking / evaluator gaming 的温床。RQGM 借用 Red Queen 假设（Van Valen 1973）重新组织这一问题：物种不是在静止环境中优化自己，而是与不断变化的环境协同演化，因此评估必须被纳入改进循环本身。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:21-23]^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:63-63] 这一视角与 [[concepts/verifier-paradox]] 所刻画的"验证器既是引擎又是枷锁"的张力一致，也与 [[entities/agent-self-evolution-evaluator-bottleneck]] 中"评估器成为自进化瓶颈"的观察相互印证。

### 受控效用进化的精巧设计：epoch 冻结 + 锚点检验 + 选择性擦除

RQGM 的核心贡献不是"让评估器跟着变"这么简单，而是让目标演化与自改进保证可以同时成立。机制上，搜索被组织成 epoch：epoch 内评估器冻结、信号稳定；epoch 边界处效用可以更新；新评估器必须在留出的锚点数据上统计意义上打败老评估器；选择性擦除只丢弃被替换评估器打过的分、保留其他证据。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:49-53] 这样"自改进保证按 epoch 成立，而目标跨 epoch 演化"。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:55-55] 从历史脉络看这是对两条前人路线的综合：Gödel Machine（2003, Schmidhuber）要求执行前数学证明自修改有益，因计算成本不可行；Darwin/Huxley Gödel Machine 用"突变→测试→生存"替代证明，但评估器仍然静态。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:59-61] RQGM 的洞察在于：把"证明"降级为"epoch 内的受控统计检验"，就能在可行性与安全性之间取得平衡——锚点检验防止评估器退化（目标漂移），选择性擦除则避免换裁判时误删仍然有效的历史证据，本质上是一种证据治理设计。

### 对抗性目标发现：修正裁判偏见，而不只是把分数推高

论文评审实验揭示了评估器共进化更深远的价值：最强的基线审稿人以 1.91 倍于人类的比例过度接受 AI 论文——一个典型的人工 judge 系统性偏差。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:42-43] RQGM 用对抗性目标去发现"对 AI 和人类同样严格"的审稿人，最终实现平等对待 AI 与人类论文、同时保持 80% 真值准确率。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:44-45] 这说明共进化的目标函数不该只是"更难"，而应该是"更无偏"：当被评估对象是 AI 生成内容、而裁判本身也是 LLM 时，宽松裁判会奖励"针对裁判弱点优化"的产出，形成闭环的自我强化偏差。这也提示评估器演化需要显式的公平性/严格性约束，而不仅仅是强度约束。

### 多域验证：agent-as-judge 的成本—质量—多样性三角

RQGM 在三个域上的结果共同刻画了评估器设计中的三角权衡。代码编写（Polyglot）上，通过率从 69.9% 提到 71.7%（+1.8% over SOTA），token 效率提升 1.35-1.72x，其机制是用 agent-as-a-judge 的代码审查信号替代更贵的多轮测试。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:29-32] 科学论文写作上，共进化写手 + 多样化 agent-as-a-judge 评审团把接收率从 21.8% 拉到 40.5%（1.78-1.86x）。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:34-36] 奥赛级数学证明上，评分器准确率比静态基线高 9%，搜索成本反而降低 3x。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:38-40] 三条线索指向同一结论：评估器质量本身就是成本杠杆——更好的 judge 信号既能减少搜索开销，又能提高优化方向的正确性；而"多样化评审团"（而非单一裁判）是提升 judge 质量、同时降低被单点迎合攻破风险的关键设计，这与 [[entities/llm-as-a-judge-agent-eval-offline-huolala-2026]] 中离线 LLM-as-judge 实践及 [[entities/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134]] 的 harness-model 共进化思路一脉相承。

## 实践启示

1. **把评估器纳入改进循环，而不是只迭代 agent。** 只要 verifier/benchmark 固定，自改进系统的收益就会逐渐转化为"对裁判的迎合"。参考 RQGM 的做法：让评估目标随 agent 能力一起演化。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:21-23]
2. **用 epoch 化的"冻结—更新"节奏保证信号可用。** epoch 内冻结评估器以获得稳定梯度信号，只在 epoch 边界更新效用；不要在跑改进循环的中途随意换 benchmark。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:49-51]
3. **升级评估器必须过统计检验。** 新评估器要在留出的锚点数据上证明自己统计意义上优于旧评估器，并用选择性擦除保留未被替换裁判污染的证据，防止目标悄悄退化。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:52-55]
4. **用多样化 judge 面板替代单一裁判。** 多样化 agent-as-a-judge 评审团是论文写作接收率提升 1.78-1.86x 的直接机制，也降低了系统被单一裁判偏好劫持的风险。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:34-36]
5. **对裁判做对抗性校准以修正偏见。** 若 judge 对 AI 生成内容系统性宽松（基线达人类 1.91x 接受率），应把"对 AI 与人类同样严格"设为对抗性目标，并监控真值准确率（保持 80%）而非只看一致性。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:42-45]
6. **把评估器质量当作成本杠杆来投资。** 更聪明的评估信号（agent-as-judge 审查、更准的评分器）带来的回报是双向的：token 消耗降低 1.35-1.72x，搜索成本降低 3x，比单纯堆更多轮测试划算。^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:29-32]^[raw/articles/rqgm-red-queen-godel-machine-cambridge-nvidia-2026.md:38-40]

## 相关实体

- [[entities/ai-recursive-self-improvement-nanogpt-prime-intellect|AI 递归自改进]] — 自改进的更广泛背景
- [[entities/agent-self-improvement-six-mechanisms|Agent 自改进六机制]] — 自改进机制分类
- [[concepts/verifier-driven-development|验证器驱动开发]] — 验证器在 Agent 系统中的角色
