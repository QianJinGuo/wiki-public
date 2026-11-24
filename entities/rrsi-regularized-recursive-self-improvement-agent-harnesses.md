---
title: "RRSI: Regularized Recursive Self-Improvement of Agent Harnesses"
created: 2026-09-24
updated: 2026-11-21
type: entity
tags: [agent, harness, self-improvement, agent-harness, evaluation, benchmark]
sources: [raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses]
confidence: 0.7
provenance_state: extracted
---

# RRSI: Regularized Recursive Self-Improvement of Agent Harnesses

## 核心主张

递归自改进（RSI）的 Agent harness 演化会**过拟合其被评分的 benchmark**——现有方法在 evolve-split 上收益巨大，但换到 out-of-distribution benchmark 后收益缩水甚至消失，其中两个方法的 OOD 成绩甚至低于演化起点。RRSI 是唯一一个 **OOD 收益随演化增长的方案**，held-out 成绩比此前方法平均高出最多 **22.9%**。^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md]

## 三角色演化循环 + 四条正则化规则

循环：proposer（读完整 ledger，花费递减的 edit budget 提出修改）→ critic（评分前先做 leakage 审查）→ gate（候选必须同时满足噪声底线与 token 成本偿付）。

正则化规则：
1. **Leakage 拒绝**：任务名、实体、答案、benchmark-specific logic 在评分前即被拒绝
2. **噪声底线**：收益必须超过 unchanged base harness 上测得的方差
3. **成本偿付**：额外 inference tokens 必须由可测收益支付
4. **组件裁剪**：停止"自付"的组件被标记删除

^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md]

## 演化粒度控制

早期轮次允许捆绑少量协调修改以发现机制；后期轮次强制单次一个可归因修改。每个候选都记录 hypothesis、diff、score、cost change，proposer 据此复用有效改动、停止重测失败项。停滞在噪声带内时，budget 重定向到从未触及的组件。^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md]

## 评测设计

- 每个领域只在一个 suite 上演化，然后 harness 原封不动跑其他所有 benchmark
- 任务类型：coding suites（pass rate）、Harvey LAB / JobBench / APEX-Agents（model-judged rubric）、GDPval（对人类专家 win rate）、EngDesign / Frontier-Eng（frozen simulators）
- RRSI 是最轻的演化 harness（policy tokens per trial 最低），也是唯一超过未演化 baseline H 的

^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md]

## 与 wiki 已有内容的关联

- 补充 [[concepts/agent-self-improvement-loops]]：现有内容覆盖 RSI 循环设计，RRSI 提供防 benchmark 过拟合的正则化机制这一缺失维度
- 与 [[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026]]、[[entities/aide2-recursive-self-improvement-weco-2026]]、[[entities/ai-recursive-self-improvement-nanogpt-prime-intellect]] 同属 RSI 谱系——RRSI 的差异化在于"正则化优先"而非"搜索能力优先"
- harness 组件可编辑性设计与 [[concepts/agent-harness-engineering-paradigm]] 的 harness-as-artifact 视角直接相关

## 深度分析

### Benchmark 过拟合是 harness 演化的结构性风险，而非工程失误

harness 演化的本质是一个以 benchmark 得分为适应度的搜索过程：每轮候选的去留由评分决定，演化压力天然指向"在这套评分上拿更高的分"。当演化集与评测集不分离时，搜索找到的必然是 benchmark 特定的捷径——任务名提示、答案泄漏、针对特定判分逻辑的 prompt 技巧——这些改动在 evolve-split 上收益巨大，换到 OOD benchmark 便立刻失效。raw 数据中最刺眼的信号是有两个先验方法的 OOD 成绩**低于未演化的起点**：搜索不仅没有泛化，反而把原本健康的 harness 改坏了^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:13-14]。这说明过拟合不是偶发的 bug，而是任何"评分即选择压力"的循环在没有隔离机制时的默认结局。

由此可以概括 RRSI 的核心洞察：**不要约束被编辑的 artifact（harness 组件始终保持可编辑），要约束编辑它的循环**——哪些改动允许被提出、哪些收益允许被记账、账要付多少成本^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:17-18]。这把问题从"如何搜索出更好的 harness"转换为"如何给搜索施加纪律"，也解释了为什么它的收益能随演化在分布外持续增长。

### 四条正则化规则分别封堵一种具体的失效模式

把 RRSI 的四条规则逐条对应到失败模式，可以看出这是一套针对搜索动力学的防御，而非针对模型能力的调优：

- **Leakage 拒绝**封堵 reward hacking 的入口：任务名、实体、答案、benchmark-specific logic 在评分之前就由 critic 拒绝。前置于评分是关键——污染分数一旦进入 ledger，就会被 proposer 当作有效信号继续放大和堆叠^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:22]。
- **噪声底线**封堵把方差当信号：候选收益必须超过在 unchanged base harness 上实测的方差，否则一次幸运的采样也会被记为"改进"，并在后续轮次被复用^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:23]。
- **成本偿付**封堵无界复杂度增长：额外 inference tokens 必须由可测收益支付，且在提案阶段就拒绝"不付钱的增长"^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:24]。
- **组件裁剪**封堵棘轮效应：已入库的组件若后来停止"自付"，会被标记删除。它处理的是成本规则覆盖不到的时间维度问题——提案时划算、运行中退化的改动^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:25]。

值得单独强调的是，成本拒绝与事后裁剪这两条没有任何先验方法同时具备^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:29]：多数 RSI 工作只优化"能否找到更好的改动"，而不优化"改动的账要不要付"。

### Ledger 与演化粒度：把开放式搜索变成可归因的实验序列

RRSI 对搜索过程本身还有两处关键设计。其一是**演化粒度调度**：早期轮次允许捆绑少量协调修改以发现机制（此时搜索空间大、单点改动信号弱），后期轮次强制单次一个可归因修改（此时需要精确记账每个改动的贡献）^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:19]。这相当于把粗粒度探索与细粒度利用在时间轴上分开，避开"大改无法归因"与"小改找不到机制"的两难。其二是 **ledger 作为实验记录**：每个候选记录 hypothesis、diff、score、cost change 四元组，使 proposer 能复用有效改动、停止重测已证伪的方向；当进度停滞在噪声带内时，budget 重定向到从未触及的组件，等价于一次受控的探索重启^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:20-21]。这套机制让 RSI 循环从黑盒优化变成可审计的实验序列，与 [[concepts/agent-harness-engineering-paradigm]] 中 harness-as-artifact 的可检查性诉求一脉相承。

### 与既有 RSI 谱系的分野：正则化优先 vs 搜索能力优先

正文已指出 RRSI 与 [[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026]]、[[entities/aide2-recursive-self-improvement-weco-2026]]、[[entities/ai-recursive-self-improvement-nanogpt-prime-intellect]] 同属 RSI 谱系，差异化在于"正则化优先"。放进更大的坐标系：[[entities/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104]] 一类综述按搜索与记忆机制给 RSI 方法分类，而 RRSI 补充的是一条正交的轴——**收益的准入标准**。[[entities/dream-rsi-history-replay-recursive-self-improvement-google-2026]] 以历史回放强化经验复用，[[entities/memento3-huawei-rsi-arc-agi3]] 在 ARC-AGI-3 上验证技能库式自进化，这些方法都可以在保留自身搜索机制的同时叠加 RRSI 的四条规则——正则化层与搜索机制是可组合的，而非互相竞争的方案。评测结果也支持这种定位：RRSI 是最轻的演化 harness（policy tokens per trial 最低），也是唯一 OOD 成绩超过未演化 baseline H 的方法^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:26-28]，说明防过拟合的纪律本身几乎不增加运行成本。

## 实践启示

1. **先隔离，后搜索**：任何自改进循环开跑之前，先把 evolve 集与 held-out 评测物理分离；每轮候选在打分前过一遍 leakage 审查。污染分数一旦入账，proposer 会将其放大，事后很难清除。
2. **给收益建立噪声底线**：先在 unchanged baseline 上实测评分方差，任何低于该方差的"改进"一律按噪声丢弃——这是防止 ledger 被运气污染的最便宜手段。
3. **让每个改动付租金**：为新增组件和额外 tokens 建立成本偿付规则，并在提案阶段就执行；对已入库组件定期复测，停止自付的组件坚决裁剪，防止 harness 单调膨胀。
4. **按阶段控制演化粒度**：探索期允许捆绑修改以发现机制，收敛期强制单变量修改保证归因；不要让两种粒度混在同一轮里，否则 ledger 里的归因会失效。
5. **把 ledger 当实验记录而非日志**：每条候选必须携带 hypothesis、diff、score、cost change，否则后续轮次既无法复用有效改动，也无法停止重测失败项。
6. **评估自改进系统看 OOD 收益曲线，而非 evolve-split 峰值**：收益只在演化集上增长是过拟合的直接证据。RRSI 的评估协议——每域只在一个 suite 上演化，随后 harness 原封不动跑其余所有 benchmark^[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses.md:26]——可以直接搬用为自改进系统的验收标准。

## 相关链接

- 来源：[[raw/articles/rrsi-regularized-recursive-self-improvement-agent-harnesses|原文存档]]
- 项目页：https://regularized-rsi.com/
- 论文：arXiv:2609.24972（Xia, Peng 等 14 人）

