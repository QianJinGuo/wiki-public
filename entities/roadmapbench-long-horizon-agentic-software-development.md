---
title: "RoadmapBench: Long-Horizon Agentic Software Development 基准评估"
created: 2026-06-30
updated: 2026-08-29
type: entity
tags: [benchmark, agent, coding-agent, evaluation, software-engineering, long-horizon]
sources: [raw/articles/arxiv-2605-15846-roadmapbench]
confidence: 0.85
review_value: 8
review_confidence: 8
---

# RoadmapBench: Long-Horizon Agentic Software Development 基准评估

> RoadmapBench 是一个面向长期、多目标软件开发的编码 Agent 评估基准，包含 115 个基于真实开源版本升级的长期任务，覆盖 17 个仓库和 5 种编程语言。最强模型 Claude-Opus-4.7 仅解决 39.1% 的任务，揭示长期软件开发仍是未解决难题。^[raw/articles/arxiv-2605-15846-roadmapbench.md:1-35]

## 摘要

现有编码 Agent 基准（如 SWE-bench）主要聚焦单 issue 的 Python 仓库 bug 修复，以粗粒度的 pass/fail 作为评估结果，无法捕捉真实工程规模下的长期、多目标开发能力。RoadmapBench 填补了这一空白：每个任务将 Agent 置于源版本代码快照上，提供多目标的 roadmap 指令，要求实现目标版本引入的功能，中位修改量 3700 行跨 51 个文件。对 13 个前沿模型的系统评估显示，即使最强模型 Claude-Opus-4.7 仅解决 39.1% 的任务，而最弱模型仅 5.2%，表明长期软件开发仍是 largely unsolved problem。^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

## 研究背景

### 现有基准的局限

当前编码 Agent 基准（如 SWE-bench、HumanEval）存在以下问题：

1. **单 issue 聚焦**：主要关注单一 bug 修复或功能添加^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
2. **语言单一**：主要集中在 Python 仓库^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
3. **评估粒度粗**：简单的 pass/fail 二元结果^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
4. **规模有限**：无法反映真实工程中跨版本、多文件、长期迭代的复杂性^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

### 真实软件开发的特点

实际生产环境中的软件迭代：
- **时间跨度长**：单个版本迭代需要数月协调工作^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- **范围广泛**：涉及多个文件的大规模修改^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- **目标多元**：同时处理多个相关功能和改进^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- **语言多样**：涉及多种编程语言和技术栈^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

## RoadmapBench 设计

### 数据集构成

| 指标 | 数值 |
|------|------|
| 任务数量 | 115 个长期编码任务 |
| 数据来源 | 真实开源版本升级 |
| 仓库数量 | 17 个开源仓库 |
| 编程语言 | 5 种（Python、Rust、Go、TypeScript、Java） |
| 中位修改量 | 3,700 行代码 |
| 中位文件数 | 51 个文件 |

^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

### 任务定义

每个任务包含三个核心元素：

1. **源版本（Source Version）**
   - 升级前的代码快照^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
   - Agent 从此状态开始工作^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

2. **目标版本（Target Version）**
   - 升级后的代码快照^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
   - 作为评估参考标准^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

3. **Roadmap 指令**
   - 多步骤、多目标的开发指令^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
   - 描述需要从源版本实现到目标版本的功能^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

### 评估指标

RoadmapBench 超越简单的 pass/fail，提供多维度质量评估：

- **功能正确性**：实现的功能是否符合目标版本行为^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- **代码质量**：代码风格、可读性、可维护性^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- **架构一致性**：与现有代码库的架构模式和设计原则的一致性^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- **修改效率**：是否最小化不必要的更改^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

## 评估结果

### 模型表现

对 13 个前沿模型的系统评估结果：

| 模型 | 解决率 |
|------|--------|
| Claude-Opus-4.7 | 39.1% |
| GPT-4.5 | ~35%（估计） |
| Gemini-2.5-Pro | ~32%（估计） |
| ... | ... |
| 最弱模型 | 5.2% |

^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

### 关键发现

1. **性能差距显著**
   - 最强与最弱模型之间差距达 7.5 倍^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
   - 即使最强模型也远未达到实用水平^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

2. **与现有基准的对比**
   - 现有 bug-fix 基准与长期开发能力之间存在巨大差距^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
   - 在 SWE-bench 上表现优秀的模型，在 RoadmapBench 上可能表现平平^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

3. **核心瓶颈**
   - **长期规划**：理解和执行跨多步骤的复杂路线图^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
   - **跨文件协调**：在数十个文件间保持一致性和正确性^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
   - **架构一致性**：理解并遵循现有代码库的架构模式^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

## 深度分析

### 为什么长期开发如此困难？

**1. 上下文窗口限制**
- 长期任务需要维护大量上下文信息^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 现有模型的上下文窗口虽在扩展，但有效利用仍是挑战^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**2. 规划与执行分离**
- 需要高层规划能力和低层执行能力的协调^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 规划错误往往在执行后期才显现，导致级联失败^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**3. 增量验证困难**
- 长期任务的中间状态往往不可运行或无法验证^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 缺乏细粒度的反馈信号指导 Agent 调整方向^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**4. 架构理解深度**
- 需要理解代码库的整体架构、设计模式和隐含约束^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 表面模仿容易，深层理解困难^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

### 多语言支持的挑战

覆盖 5 种编程语言带来额外复杂性：

- **语言特性差异**：不同语言的抽象机制、类型系统、并发模型^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- **生态差异**：包管理、构建系统、测试框架的差异^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- **惯用法差异**：每种语言的惯用模式和最佳实践^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

### 与 SWE-bench 的对比

| 维度 | SWE-bench | RoadmapBench |
|------|-----------|--------------|
| 任务类型 | 单 issue bug 修复 | 多目标版本升级 |
| 代码规模 | 通常 <500 行修改 | 中位 3,700 行修改 |
| 文件数量 | 通常 <10 个文件 | 中位 51 个文件 |
| 语言 | 主要为 Python | 5 种语言 |
| 评估粒度 | pass/fail | 多维度质量评估 |
| 最强模型表现 | >50% | 39.1% |

^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

## 实践启示

### 对 Agent 架构设计的启示

**1. 分层规划架构**
- 需要显式的高层规划层，将 roadmap 分解为可管理的子任务^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 子任务之间需要依赖管理和协调机制^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**2. 增强上下文管理**
- 需要更智能的上下文选择机制，在有限窗口内保留最相关信息^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 考虑外部记忆机制，如向量数据库或符号知识库^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**3. 增量验证机制**
- 设计可在开发过程中频繁运行的测试和验证检查点^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 尽早发现偏离，避免后期大规模回滚^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**4. 架构感知能力**
- Agent 需要显式建模代码库的架构结构^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 利用静态分析、依赖图、代码嵌入等技术增强架构理解^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

### 对评估体系的启示

**1. 超越 pass/fail**
- 需要多维度质量评估，捕捉解决方案的细微差别^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 考虑人工评估与自动评估的结合^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**2. 真实场景覆盖**
- 评估数据集应反映真实工程实践的多样性和复杂性^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 定期更新以跟上技术演进^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**3. 可解释性评估**
- 不仅评估结果，还评估 Agent 的决策过程和推理链^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 帮助识别失败模式和改进方向^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

### 对产业应用的启示

**1. 当前能力边界**
- 长期、多文件软件开发任务仍需要人类主导^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- Agent 更适合作为辅助工具，而非独立开发者^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**2. 渐进式自动化**
- 从子任务级别开始自动化，逐步扩展到完整任务^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 建立有效的人机协作流程^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

**3. 领域特定优化**
- 针对特定语言、框架或应用类型优化 Agent 能力^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]
- 利用领域知识弥补通用能力的不足^[raw/articles/arxiv-2605-15846-roadmapbench.md:14-19]

→ [[raw/articles/arxiv-2605-15846-roadmapbench|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

