---
title: "ReHarness — 给 Agent 记忆装上反向传播：Recurrent Harness for Agentic Continual Learning"
created: 2026-10-13
updated: 2026-10-13
type: entity
tags: [agent-memory, continual-learning, harness, backward-revision, reflection, distribution-shift, stability-plasticity, mit, meta, yale]
value: 9
confidence: 0.85
sources: [raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13]
provenance_state: extracted
---

# ReHarness：给 Agent 记忆装上反向传播（Recurrent Harness）

MIT / Meta MRS Research / Yale《Recurrent Harness for Agentic Continual Learning》全文精读（AGI研习社，技术报告级解读档）：现有 Agent Harness 只做前向读写，过度泛化/过时的反思在分布漂移下持续误导决策——ReHarness 增加一条**反向修订（Backward Revision）**通路，相当于给纯文本记忆系统装上反向传播。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md]

## 三步 Recurrent 循环

约束=实例只过一次/无验证集/黑盒 API → 学习完全锁定在文本空间：①**前向读**（检索 top-k 反思辅助决策）②**反向修订**（修订 Agent 结合源轨迹+当前实际效果改写被检索反思；硬约束=当前轨迹只作反馈、新内容必须有据可溯来自源轨迹防泄漏；只做三件事：补忽略的证据/纠无支撑假设/收紧适用条件）③**前向写**（记忆 Agent 以修订后反思为示范生成新条目，鼓励互补视角——示范消融 0.217→0.191）。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md]

## 主结果

严格流式协议 × 3 Benchmark（CL-Bench 6 任务/MemoryBench 5 写作/MemoryArena 5 领域）× 3 Backbone（Claude Opus 4.8/GPT-5.5/Gemini 3.7 Flash）：平均 normalized gain **0.240**，9 组合 8 个第一，比最强全历史基线 Full Raw（0.164）高约 46%，Wilcoxon（Bonferroni）全基线显著；token 仅 Full Raw 约 1/3（41.57M/$254 vs 142.12M/$314，压缩反思省输入）。原始上下文基线强于多数结构化记忆方法（与 CL-Bench 原文一致）；纯反思方法（RBank/DoT/ACE）部分负增益；GEPA-Online 在 MemoryBench×Gemini 崩塌 -0.134。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md]

## 四个递进分析

- **泄漏分析（FActScore 式）**：grounded 率 95.3-98.7%、泄漏率 0.8-2.1%、修订前后几乎不变——backbone 遵守"只反馈不抄素材"约束。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md]
- **反事实重放**（违反在线协议仅作诊断）：0.335 > ReHarness 0.217 > 无修订 0.144——修订确实从源轨迹挖出"本该被记下"的信息。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md]
- **定性两案例**：纠正过度泛化（盲频谱"填补静默空隙"→标推测性，0.297→0.219→0.367）；漂移后更新过时经验（schema 漂移 prc→prc_v2，"不预设哪列被评分、枚举所有价格类列"，0→0.867）。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md]
- **稳定性×可塑性**：Mem0/RBank/DoT/ACE/GEPA-Online 平均稳定性全为负（纯前向记忆漂移时反而有害）；ReHarness 双第一（0.015/0.241）。top-k 扫描甜点 k=5（过大→反思漂移）；热力图显示修订随漂移动态转移（漂移后旧反思大幅修订→集中到近期）。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md]

## 边界条件

Poker 上失效——隐藏牌+运气使单局奖励与策略质量弱耦合，反向修订信号是噪声（Raw 类原样保留轨迹反而不受信用分配问题影响）；**ReHarness 最受益于反馈具有诊断性的任务**。局限：噪声奖励场景修订弱、需保留原始轨迹（存储开销）。意义：把"用下游反馈修正上游表示"移植到纯文本记忆空间——权重封藏 API 后，Harness 层循环适配可能是 Agent 持续学习主战场。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md]

## 深度分析

### 反向修订是文本空间里的反向传播

传统微调用梯度把下游损失回传修正上游权重；ReHarness 在黑盒 API、无验证集、实例只过一次的三重约束下，把同一结构映射进 Harness 状态：下游奖励当损失信号，修订 Agent 当梯度算子，被检索到的反思当被激活的参数。关键细节是只修订"本次被检索"的条目、未检索的保持不动——正如梯度只更新参与前向计算的权重，更新天然稀疏、开销被限制在真实使用过的记忆上。前向写与反向修订构成闭环后，记忆系统第一次拥有"错题订正"而非"错题累积"。这与 [[entities/agent-self-evolution-evaluator-bottleneck]] 的判断互补：自进化的瓶颈在评估器，而下游奖励是现成的评估信号，缺的只是回传通路。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:19-22]

### 三件事约束：防泄漏同时防能力漂移

修订被硬约束为只做三件事——补入源轨迹中被忽略的证据、纠正无支撑假设、收紧适用条件——且当前轨迹只作反馈信号、新内容必须可溯源到源轨迹。泄漏分析（FActScore 式）给出实证：grounded 率 95.3-98.7%、泄漏率 0.8-2.1%、修订前后几乎不变。这套约束同时防住两类失败模式：信息泄漏（用未来信息伪造过去经验，污染在线评估）与过度改写（修订 Agent 自由发挥，把记忆变成二度创作）。记忆可以重建但必须有据——与 [[entities/mragent-memory-reconstructed-not-retrieved-nus-icml2026]] 的"重建而非检索"命题在同一方向上。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:21,33]

### 原始上下文悖论：记忆的主要矛盾是"错了谁来改"

Full Raw（0.164）强于多数结构化记忆方法，纯反思方法（RBank/DoT/ACE）部分负增益，且五条纯前向基线（Mem0/RBank/DoT/ACE/GEPA-Online）平均稳定性全为负——分布漂移下只增不改的记忆不仅无用而且有害。这说明 Agent 记忆的主要矛盾不在"存什么、存多少"，而在过时与过度泛化的条目无人订正；[[entities/agent-memory-main-contradiction-context-scheduling]] 指向上下文调度，本文在其上补出一条反向通路。反事实重放 0.335 > ReHarness 0.217 > 无修订 0.144 证明修订确实从源轨迹挖出了前向写漏掉的信息。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:30,34,36]

### 修订强度的杠杆：top-k 甜点与诊断性反馈边界

top-k 扫描显示 k=1→5 增益急升、k=5 平稳、更大反而下降——过度修订引入"反思漂移"，修订本身成为新噪声源；热力图还显示修订量随分布漂移动态转移：漂移前旧反思修订减弱，漂移后先大幅修订旧反思、再集中到近期。Poker 失效划出边界：隐藏牌+运气使单局奖励与策略质量弱耦合，反向修订信号退化为噪声，而 Raw 类原样保留轨迹反而免疫信用分配问题。反向通路的适用前提是反馈具有诊断性——评估信号质量决定修订上限，与 [[entities/agent-self-evolution-evaluator-bottleneck]] 在信号层面同构。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:31,36,37]

## 实践启示

1. 给记忆系统补反向通路的优先级高于继续优化前向写入：先让"被检索过的反思在用坏之后被改写"，再谈增加记忆条目或结构。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:15]
2. 把修订操作收敛为三件事（补证据/纠假设/收条件）并以源轨迹为唯一素材来源，是修订可以安全自动化的前提；放开改写自由度会同时放大泄漏与漂移两类风险。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:21,33]
3. 判断任务是否适合自修订记忆，先看反馈的诊断性：奖励与决策质量强耦合（写作评分、数据探索、预测类任务）收益大；高噪声奖励场景（含运气成分的对局类）应退回保留原始轨迹。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:31]
4. 检索宽度 top-k 是修订强度的杠杆：k 过大修订面过宽会引入反思漂移，k=5 附近是甜点，迁移到自有场景时值得重新扫描而非照搬。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:37]
5. 压缩与可溯源不冲突：压缩反思把单 rollout token 从 142.12M 降到 41.57M（$314→$254），省的是重复读取原始轨迹的输入成本；原始轨迹仍需保留作修订素材库——被压缩的是读取，不是存储。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:32]
6. 权重封藏在 API 之后，Harness 层的"前向读写+反向修订"循环适配是无需梯度、无需验证集的持续学习可行形态，也是作者指出的主战场方向。^[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13.md:41]


## 相关实体

- [[entities/agent-memory-main-contradiction-context-scheduling]]
- [[entities/mragent-memory-reconstructed-not-retrieved-nus-icml2026]]
- [[entities/agent-self-evolution-evaluator-bottleneck]]

→ [[raw/articles/reharness-recurrent-harness-backward-revision-agiyanxishe-2026-10-13|原文存档]]
