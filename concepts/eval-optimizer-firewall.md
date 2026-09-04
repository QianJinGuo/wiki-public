---
title: "评测防火墙（Eval–Optimizer Information Isolation）"
created: 2026-08-29
updated: 2026-09-03
type: concept
tags: [concept, evaluation, verifier, anti-leakage, anti-gaming, benchmark, harness, quality-gate]
sources: [entities/self-harness-shanghai-ai-lab-agent-improves-harness]
confidence: 0.75
provenance_state: merged
---

## 定义

评测防火墙：当系统存在一个会针对评测信号做优化的 optimizer（训练循环、自改进 agent、meta-harness 优化器），评测器与 optimizer 之间必须建立**显式的信息与执行隔离**——评测面的内容、轨迹与锚点不进入 optimizer 的上下文，评测的执行环境与被评对象物理分离，评测函数本身按「被游戏后失真」来设计。它是 [[concepts/verifier-driven-development|verifier 驱动开发]] 在「optimizer 存在」前提下的加固层：verifier 不仅要存在，还必须**对 optimizer 保密到必要的程度**。

## 为什么 verifier 不够

[[concepts/verifier-driven-development|Verifier 驱动开发]] 已经回答了「verifier 要多层」「LLM-as-judge 有 bias」「谁验证 verifier」。但它隐含假设 verifier 是静态裁判席。一旦引入会学习的 optimizer，新的失效模式出现：optimizer 对 eval 过拟合（goodhart）、eval 内容泄漏进训练/提议上下文（泄漏）、agent 篡改评测面或基线（作弊）、评分函数区分度不足使「平庸」与「优秀」不可分（压团）。2026 年的自主科研/工程系统开始把这四种失效当成显式架构约束来处理。

## 四个机制（附一手证据）

**1. 信息隔离（eval 对 optimizer 保密）**。AutoDesign 的外层 MetaHarnessOptimizer 接受候选的依据是「training set 提升 且 独立 development set 不降」，且 **development set 的执行轨迹不暴露给更新提议器**——不是隐藏分数，是隐藏轨迹 。faibench 把 tests/（test.sh + compute_reward.py + anchors）**永不烘焙进镜像**、评分时只读挂载，镜像内 manifest 故意不含真 anchor：不挂载 tests/ 就大声失败，而不是给出可疑分数 。

**2. 执行分离（评测环境与被评物物理分离）**。faibench 评分永远 `--network=none`、vendored 挖掘树用 oracle.patch 正向可 apply / 反向不可 apply 双向自证 provenance、同 upstream 家族任务互相持有对方 scope 文件于挖掘态（防一个任务的镜像里藏着另一个任务的答案）。

**3. 激励函数设计（把「被游戏」写进曲线）**。faibench 的 oracle-zero 对数曲线 `reward = min(1, ln(speedup/ref)/ln(ref))` 把「打平 oracle」的分数大团整个压缩为 0，整个 [0,1] 留给超越 oracle 的幅度；配六条零分硬门与「宁可拒评不出可疑分」的拒评语义，`hard_fail`（运行无效/作弊）与「曲线得 0」（只是没打过 oracle）严格分账 。

**4. 成本分层的确定性先行（让 LLM 只做代码做不了的最后一步）**。Polaris 的 Sextant 按成本排序 check 链：`observation.error` → self_check → 结构化确定性 checks（先行、短路）→ `llm_rubric`（唯一一次模型调用）；失败字符串刻意写得可注入（`[metric] accuracy = 0.62, does not satisfy >= 0.8`），被复用为重试参数与 replanning prompt 的输入 。spark-to-paper 的 `draft_lint` 只把**语境无关**的规则交给代码硬 fail（AI 套话、proposal 模式的具体数字禁令），语境相关的判断留给模型——防止 lint 误伤产生「逗号汤」。

## 谱系与定位

[[entities/self-harness-shanghai-ai-lab-agent-improves-harness|Self-Harness]] 的 held-in/held-out 双门是本概念的先声：harness 修改提案必须在 held-out 任务上回归验证，实质是「提议器不得见过验收面」。不同在于 Self-Harness 把它用作**合并门**，本概念把它上升为**评测系统的一等架构属性**。与 [[concepts/verifier-driven-development|verifier 驱动开发]] 的关系：那是「verifier 必须存在」，这是「verifier 必须对 optimizer 有边界」；与 [[concepts/agent-self-improvement-loops|自改进循环]] 的关系：自改进越自动，防火墙越必要——人类 gate 是最后的防火墙，但不能是唯一的（gate 会形式主义化，见 [[drafts/wiki-emergent-viewpoints-2026-07|2026-07 涌现观点·验证者悖论]]）。

## 本 wiki 的自反

本 wiki 的 quality-loop（LLM 首轮评审 16 篇、keep/refresh/archive）同样存在 optimizer（ingestion pipeline 的打分调整）与 eval（LLM 评审）共谋的风险：评审 prompt 若引用了 ingestion 侧的评分锚点历史，评分锚点就会自我强化。防火墙设计问题同样适用：**评审上下文应当只含被评页面与锚点定义，不含生产侧的调分记录**。参见 [[concepts/evaluation-harness-design|evaluation harness 设计]]、[[moc/evaluation-and-benchmarks|评测 MOC]]。

## 参见

- [[concepts/verifier-driven-development|verifier 驱动开发]] — 上位概念：verifier 必须存在
- [[entities/self-harness-shanghai-ai-lab-agent-improves-harness|Self-Harness]] — held-in/held-out 双门的先声
- 待补公开原文 — 防火墙机制最完备的样本
- [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 涌现观点·观点二]] — 概念来源

## 所属 MOC

- [[moc/evaluation-and-benchmarks|Evaluation and Benchmarks]]
- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
