---

title: "Anthropic Claude Fable 5.1 / Mythos 5.1 发布"
created: 2026-09-03
updated: 2026-11-07
type: entity
tags: [anthropic, claude, fable, mythos, model-release, benchmark, safety, protein-design, cybersecurity]
sources: [raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026]
confidence: 0.9
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Anthropic Claude Fable 5.1 / Mythos 5.1 发布

## 核心信息

Claude Fable 5.1 和 Mythos 5.1 是同一模型的不同安全等级版本。Fable 5.1 为通用版本，Mythos 5.1 仅通过 trusted access programs 提供，专门支持网络安全和生命科学研究。两者于 2026 年 9 月发布。 ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md]

**关键变化：**
- 价格降低 25%（典型工作负载），agentic 工作负载最高降低 45%（通过 cache read 定价优化）
- 企业级前沿安全防护（Enterprise Frontier Safeguards, EFS）：客户完全控制数据存储，等效零数据保留
- 安全误报减少 60%（网络安全领域）
- Fable 5.1 默认 High effort（Claude Code），Medium effort（Claude Cowork / Claude.ai） ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md]

## 基准测试结果

| 基准测试 | Fable 5.1 | Fable 5 | Opus 5 | GPT-5.6 Sol |
|---------|-----------|---------|--------|-------------|
| Agentic 科学研究 Terminal-Bench-Science 0.1 | 52.6% | 24.7% | 29.0% | 22.4% |
| Agentic 编程 Terminal-Bench 4.0 | 55.8% (Mythos 60.9%) | 42.0% | 52.3% | 37.3% |
| 知识工作 GDPval-AA v2 | 1853 | 1723 | 1824 | 1711 |
| Computer use OSWorld 2.0 (partial) | 77.9% | 72.9% | 75.4% | — |
| Computer use OSWorld 2.0 (strict) | 41.7% | 36.1% | 39.6% | — |
| 多学科推理 HLE (no tools) | 60.9% | 57.8% | 56.6% | — |
| 多学科推理 HLE (with tools) | 65.0% | 63.8% | 63.6% | — |
| 业务流程 AutomationBench | 31.4% | 17.1% | 26.9% | 19.6% |
| Agentic 编程 CursorBench 3.2.0 | 73.4% | 70.5% | 70.0% | 67.2% |

**Terminal-Bench-Science 0.1 提升幅度最大：52.6% vs 24.7%（Fable 5），接近翻倍。** ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md]

## 科学研究能力

### 分子设计
Mythos 5.1 能设计高亲和力的蛋白质结合剂。在 3 个靶点上，其结合亲和力比 Adaptyv Bio 蛋白质设计竞赛中最佳设计高 10 倍。在 12 个靶点上的命中率接近 50%（当前行业典型为 10-15%）。 ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md]

### 计算分析与建模
Fable 5.1 训练了一个神经网络，基于 NASA Magellan 任务 30 多年前的雷达图像，创建了金星三分之一区域的高分辨率高程图。新地图分辨率达 2-3 公里（此前 10-20 公里），高度精度提高 25%。以 Creative Commons 许可证发布。 ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md]

## 安全与合规

- **网络安全**：Fable 5.1 可用于发现软件漏洞，但不能开发利用代码
- **生物学**：与美国政府合作建立访问计划，支持 Mythos 5.1 的高级生物学能力
- **EFS**：数据存储在客户完全控制的云基础设施中，Anthropic 无法访问
- **False positive 减少 60%**（网络安全领域）

## 合作伙伴反馈

- **Jane Street Capital**: "Fable 5.1 解决了更多编码问题，达到交易直觉的 SOTA"
- **Cognition**: "在发布日将 Devin 的 Opus 5 流量迁移到 Fable 5.1"
- **Millennium**: "Fable 5.1 发现了一个 4-5 年无人能解释的罕见崩溃原因"
- **MongoDB**: "Fable 5.1 在约 3 天内构建了一个复杂原型，无人值守运行数小时"
- **Every**: "Fable 级智能，Opus 级价格，Sonnet 级速度"

## 深度分析

### 接口分层：同一模型、两套安全包络

Fable 5.1 与 Mythos 5.1 是同一权重、不同安全包络的分层发布：通用版 Fable 5.1 面向所有用户，Mythos 5.1 仅通过 trusted access programs 提供，专门支持网络安全与生命科学研究。这种"权重相同、访问分层"的策略把安全分级从训练阶段前移到访问控制阶段——模型能力本身不变，变化的是谁能使用它、在什么基础设施里使用。 ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md:13]

### 降价的杠杆在缓存定价而非推理效率

25-45% 的降价并非来自推理架构突变，而是 cache read 定价下调：典型负载约降 25%，高 agentic 负载（大量重复读取长上下文）最高降约 45%。这说明 agent 工作负载的成本模型已经从"每次调用多少钱"演变为"上下文重读多少钱"——缓存定价正在成为 agentic 产品竞争力的核心变量。 ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md:17]

### 科学产出物成为差异化宣传点

Terminal-Bench-Science 0.1 从 24.7% 跃升至 52.6%（接近翻倍），是全部基准中最大跳变；分子设计（3 个靶点亲和力超 Adaptyv Bio 竞赛最佳设计 10 倍、12 靶点命中率接近 50%，行业典型仅 10-15%）与金星高程图（分辨率 10-20km → 2-3km，Creative Commons 发布）是两个可独立验证的实物产出。相比之下编程与知识工作类基准的进步是 10-30% 的渐进式提升——Anthropic 正在用"科学产出物"而非跑分来构建差异化叙事。 ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md:46]

### EFS 把数据主权落到基础设施层

Enterprise Frontier Safeguards 不是"承诺不保留数据"的合同条款，而是把存储位置移到客户完全控制的云基础设施——等效零数据保留，但信任对象从 Anthropic 的承诺换成了客户自己的基础设施。合作伙伴名单（Jane Street、Millennium、IMC 等金融机构）暗示这正是受监管行业采用前沿模型的合规解法；EFS 分阶段推出前的过渡期用零数据保留选项衔接。 ^[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026.md:19]

## 实践启示

1. 模型选型时对高 agentic 负载优先比较 cache read 价格而非裸 token 单价——同一模型在长上下文重读场景的成本差可达 45%。
2. 金融/医疗等受监管团队可把 EFS 纳入合规路径评估；在分阶段可用之前先用零数据保留选项过渡。
3. 安全误报减少 60% 意味着此前因误报率放弃模型辅助漏洞发现的团队值得重新评估——注意边界：可用于发现漏洞，不能用于开发利用代码。
4. 评估科研类 agent 任务应参考 Terminal-Bench-Science 等专用基准，通用编程基准无法反映这代模型在科学场景的跳变（两者进步幅度差一倍以上）。
5. 如需 Mythos 5.1 的网络安全/生命科学能力，提前规划 trusted access program 的申请流程，它不对一般渠道开放。

## 与 Fable 5 的核心差异

| 维度 | Fable 5 | Fable 5.1 |
|------|---------|-----------|
| 价格 | 基准 | 降 25-45% |
| 科学研究 | 24.7% | 52.6% (↑113%) |
| 编程 | 42.0% | 55.8% (↑33%) |
| 安全误报 | 基准 | 减 60% |
| 数据保留 | 标准 | EFS 可选零保留 |
→ [[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026|原文存档]]
