---

title: "Anthropic Claude Fable 5.1 / Mythos 5.1 发布"
created: 2026-09-03
updated: 2026-09-03
type: entity
tags: [anthropic, claude, fable, mythos, model-release, benchmark, safety, protein-design, cybersecurity]
sources: [raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026]
confidence: 0.9
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

## 与 Fable 5 的核心差异

| 维度 | Fable 5 | Fable 5.1 |
|------|---------|-----------|
| 价格 | 基准 | 降 25-45% |
| 科学研究 | 24.7% | 52.6% (↑113%) |
| 编程 | 42.0% | 55.8% (↑33%) |
| 安全误报 | 基准 | 减 60% |
| 数据保留 | 标准 | EFS 可选零保留 |
→ [[raw/articles/anthropic-claude-fable-5-1-mythos-5-1-2026|原文存档]]
