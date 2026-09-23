---
title: "阿里Qwen开源 Skill-SP：自博弈实现模型和Skill协同进化新范式"
created: 2026-07-28
updated: 2026-09-24
type: entity
tags: ['auto-harvested', 'self-play', 'skill-evolution', 'search-agent', 'curriculum-learning']
sources:
  - raw/articles/qwen-skill-self-play-hyman-2026
  - raw/articles/sesa-self-evolving-search-agents-xhs-2026-08-03
  - raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/qwen-skill-self-play-hyman-2026.md|原文存档]]

一句话讲清楚👉🏻 阿里 Qwen 大模型应用团队开源 Skill Self-Play （ Skill-SP ）：用会进化的 skill 库同时管「出什么题」和「怎么自动判对错」，让自博弈既能覆盖开放任务，又能挡住假题；工具调用最高抬 42.9 分，逻辑推理上也能把几乎起不来的弱模型拉回正轨。 ^[raw/articles/qwen-skill-self-play-hyman-2026.md]

## 来源

- 原文: [[raw/articles/qwen-skill-self-play-hyman-2026.md|阿里Qwen开源 Skill-SP：自博弈实现模型和Skill协同进化新范式]]
- 原始链接: : https://mp.weixin.qq.com/s/czQ1AnCD5qwswhKmutLGgQ

## SESA：搜索场景的 Self-Play + Skill 进化（Supplementary）

**SESA（Self-Evolving Skill-Augmented Agent）** ——《Self-Play Meets Skill Evolution: Self-Evolving Search Agents that Pose, Solve, and Remember》（arXiv 2607.29468，第一方作者 XHS 发布，2026-08-03）将 Skill-SP 同源范式落地到开放域/多跳问答搜索场景：Proposer 出题 → Solver 解题 → 将**有价值的失败轨迹提炼成可复用的 Skill Card 存入持续更新的 Skill Bank**，形成「失败 → 技能 → 能力提升 → 更难问题 → 新失败」的闭环自进化。^[raw/articles/sesa-self-evolving-search-agents-xhs-2026-08-03.md]

**与 Skill-SP 的机制同源**：两者核心都是用进化的 skill 库驱动自博弈——Qwen Skill-SP 用 skill 库同时管「出题」和「判对错」（覆盖开放任务+挡假题），SESA 用 Skill Bank 沉淀失败经验供下一轮进化。差异在场景与证据：SESA 给出 7 个开放域/多跳问答 Benchmark 上相比 Search Self-Play 平均 +1.2~3.2 点的量化提升。^[raw/articles/qwen-skill-self-play-hyman-2026.md, raw/articles/sesa-self-evolving-search-agents-xhs-2026-08-03.md]

**关键消融证据（不可替代维度）**：SESA 推理时**关闭 Skill Bank** 后模型仍保留大部分能力增益——说明技能不仅是提示词注入，而是真正参与并影响了模型训练，为「skill 库驱动自博弈」路线提供了训练级（而非 prompt 级）增益的直接证据。^[raw/articles/sesa-self-evolving-search-agents-xhs-2026-08-03.md]

## 论文原文补强（SUPP 2026-08-06，arXiv 第一手来源）

> 用户提供论文原文 PDF（arXiv:2607.22529，30 页），补充二手解读缺失的形式化定义、完整分项数据与局限未来。^[raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06.md]

### 形式化目标（gated curriculum reward 防 reward hacking）

可验证任务形式化为元组 (𝒙, 𝒄)：𝒙 为 solver 可见 prompt，𝒄 为隐藏机器可读验证契约（单元测试/参考答案），环境返回 Rsolve ∈ [0,1]。proposer 目标 = 𝟙{(𝒙,𝒄) is valid} · (1 − 2|vsolve − 0.5|)——瞄准 solver 学习前沿（50% 正确率），**二元质量过滤器显式 gate proposer 奖励**：防止 proposer 合成 ill-posed/不可解契约伪造人工难度（reward hacking）。外层目标联合优化 skill 库与 proposer，持续合成 valid + frontier-targeted 任务。^[raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06.md]

### 完整分项数据

**Qwen3-4B-Inst**（60.2 → 66.7，+6.5）：API-Bank L1 +6.5/L2 +12.2/L3 +8.4；BFCL JS +7.7/Py +4.2/Java +3.7/Live +2.9。**Qwen3-8B** 69.4 基线各分项均正向。**Ministral-3-8B** 20.7 → 63.6（+42.9，Unguided SP 几乎无进步——技能库提供标准化出题模板让训练信号启动）；**Ministral-3-14B** 22.2 → 64.5（+42.3）。ZebraLogic：Qwen3-4B +1.4、Qwen3-8B +8.8、Ministral-3-14B 整体 +12 点/简单谜题 +35.3；四档复杂度网格谜题验证生成的课程帮助 solver 学习更难推理模式（非仅局部格式改善）。^[raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06.md]

### 技能库进化统计

5 轮迭代后 **Active skills 86 套、Effective（≥1 accepted record）46 套**，从初始十几套扩张——持续拓宽任务类型覆盖。^[raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06.md]

### 局限与未来（一手声明）

局限：①发现全新任务模式需基础模型最低基础能力 ②极复杂领域初期需少量人工演示 jumpstart ③依赖固定启发式（静态混合比例 α、预定义难度边界，新任务族需经验调参）。未来：可学习动态课程调度器取代固定路由；直接从原始环境交互全自动 co-induce 生成规则与可执行验证器；**跨模型架构迁移演化技能库**（强模型引导小模型，可扩展民主化对齐）；拓展多模态/长流程 Agent 场景。^[raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06.md]
## 深度分析

### Gated curriculum reward：把「防 reward hacking」做进奖励函数里

Proposer 的奖励被写成 `1{(x,c) is valid} * (1 - 2|v_solve - 0.5|)`——乘在难度分前面的二元有效性过滤器意味着：格式混乱、校验契约不可执行、参考答案对不上的题直接拿零分，proposer 无法靠合成 ill-posed 契约伪造「高难度」来刷分。难度分本身在 solver 成功率 50% 处最大，把出题稳定压在学习前沿附近。这个设计的巧妙之处在于它不靠事后审查，而是让 proposer 的最优策略与「出可验证、且刚好半对半错的题」完全对齐。^[raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06.md]

### Skill 库是训练级增益，不是 prompt 级技巧

SESA 的消融给出关键证据：推理时关闭 Skill Bank，模型仍保留大部分能力增益——说明 skill 沉淀在训练循环里真实参与了策略更新，而非只在推理时注入提示词。Skill-SP 的对应消融同向印证：冻结 skill 库 -2.3、冻结出题器 -2.1、去 skill 编排 -2.6，增益来源是「skill 驱动的课程质量」，可迁移到训练侧。对 self-play 路线而言，这把 skill 库从「工程辅助」提升为「训练信号的结构化来源」。^[raw/articles/sesa-self-evolving-search-agents-xhs-2026-08-03.md]

### 增益不对称：越弱的模型受益越大

Ministral-3-8B 从 20.7 涨到 63.6（+42.9），Unguided SP 几乎原地踏步；而较强的 Qwen3-4B-Inst 只 +6.5。原因在于 skill 库的标准化出题模板为弱模型 bootstrap 出了最初的有效训练信号——没有它，弱模型连一条可学习的数据都造不出来。这暗示 skill 库本质上降低了 self-play 的「启动门槛」，其价值随基础能力下降而上升；论文也把它列为未来方向：强模型演化出的技能库可迁移给小模型，实现民主化对齐。^[raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06.md]

### 局限的深层含义：启发式参数是场景税

静态 skill 流/探索流混合比例 α 与预定义难度边界都是固定启发式，新任务族必须经验调参——这说明当前框架把「课程超参」的复杂度从标注转嫁到了每个新域的配置上。配套的约束还有两条：发现全新任务模式需要基础模型有最低基础能力，极复杂领域初期仍需少量人工演示 jumpstart。可学习动态课程调度器被列为替代方案，若落地将消除这一场景税。^[raw/articles/qwen-skill-self-play-paper-arxiv-2026-08-06.md]

## 实践启示

1. **自博异数据管线先建质量门再谈多样性**：像 gated reward 一样，先给 proposer 的产出加二元有效性过滤（格式、可执行校验、参考一致性），否则 reward hacking 会在第 1 轮就污染训练池。
2. **难度调度瞄准 50% 成功率**：`1 - 2|vs - 0.5|` 形式的难度奖励可直接抄；训练池按难度排序取题，避免全难或全易的无效梯度。
3. **弱模型冷启动优先上标准化模板**：当被训模型基线很低（如 Ministral-3-8B 的 20.7），无引导自博弈几乎无效；用结构化 skill 模板先造出可用训练信号，再逐步放开探索。
4. **双流出题防模板塌缩**：保留一条无 skill 约束的探索流专挖新模式，同时按难度配比混入训练池（各取一半），否则整库会塌进少数高频模板。
5. **验证「skill 是否真的有用」要看训练侧消融**：评估 skill 库时，别只看带 skill 推理的分数——做一次 Skill-Bank-off 消融（SESA 式），确认增益是训练级而非 prompt 级，否则可能只是过拟合了提示词。
6. **固定启发式参数按域登记**：α 混合比例、难度边界这类超参在换域时要重新调；迁移到新任务族前先小规模网格搜索，别假设上一域的取值可直接复用。
