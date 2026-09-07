---
title: "阿里Qwen开源 Skill-SP：自博弈实现模型和Skill协同进化新范式"
created: 2026-07-28
updated: 2026-09-07
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
