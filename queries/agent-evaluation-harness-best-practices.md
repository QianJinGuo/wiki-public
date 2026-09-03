---
title: "Agent Evaluation Harness 的最佳实践是什么？"
created: "2026-05-21"
updated: "2026-05-21"
type: query
tags: [evaluation, harness, agent, benchmarking, best-practices]
---

# Agent Evaluation Harness 的最佳实践是什么？

## 评测问题

构建可靠 Agent 的评测 Harness 有哪些最佳实践？如何避免评测偏见？

本 Query 聚合了以下五个来源的核心洞察，回答如何设计可信、可复现、有实际指导意义的 Agent 评测框架。

---

## 一、Key Principles for Good Agent Evaluation Harness Design

### 1. 评测先行，YAML 驱动配置化

传统测试金字塔（单元测试 → 集成测试 → E2E）覆盖不了 Agent 的核心质量问题：非确定性（同一 prompt 跑两次结果可能不同）、传播效应（改一个词导致整个行为链路不可预测变化）、上游波动（模型升级导致表现漂移）。

**YAML 驱动的评测框架**（如 AgentEval）将评测逻辑与执行引擎解耦：评测任务以 YAML 声明式定义，支持多 Agent 适配器、多评分策略（exact_match/contains/regex/command/llm/constraint）、分层 tags（core/regression/safety）和 fail-under 阈值机制。

**实践要点**：
- 使用 `--tags core --fail-under 0.9` 在 PR 合并前跑核心用例
- 使用 `--fail-under 0.8` 做日常回归全量评测
- 安全审查类用例 `--fail-under 1.0` 零容忍

### 2. 双指标体系：pass@k + pass^k

单一 pass@k（k 次里至少有一次通过的概率）衡量的是"能力上限"，但无法反映稳定性。必须同时追踪 **pass^k**（k 次全部通过的概率）来衡量"可靠性"。

当 pass@k 很高但 pass^k 很低时，说明 Agent 有一定概率做对，但不稳定——这是随机性问题，而非能力上限问题，单纯提高 pass@k 阈值没有意义。

### 3. 先写评测，再写内容（Perplexity 原则）

Skill 开发领域有个反直觉但极其重要的原则：**Step 0 是写评测，不是写 Skill**。这个原则同样适用于 Agent 评测 Harness 设计。

- 从真实用户查询、已知失败案例、易与邻近领域混淆的情况中寻找评测素材
- **负面案例（Gotchas）比正面案例更有力量**——告诉模型"何时不该加载/何时不该做"比告诉它"何时该做"精度更高
- 多模型评测是必要条件：不同模型（GPT、Claude Opus、Claude Sonnet）对同一 Agent 的表现差异很大，不能以单一模型推断全平台行为

### 4. 评分器的分层设计

评分器形成从客观到主观的连续光谱：

| 评分器类型 | 适用场景 | 可复现性 |
|-----------|---------|---------|
| exact_match/regex/command | 有明确正确答案或可跑测试脚本的场景 | 高 |
| contains | 检查关键词是否出现 | 中 |
| llm (LLM-as-judge) | 开放式问答主观评分 | 低（引入评分不稳定性） |
| constraint | 合规检查（must_match/must_not_match） | 高 |

多评分器 AND 组合要求所有评分器都通过才算 trial 通过——这在某些场景下过于严格，但强制保证了结果的可靠性。敏感场景（安全/合规）必须用 constraint 守门员，fail-under 设置为 1.0。

### 5. "无提示"评测设计哲学（ProgramBench 原则）

ProgramBench 的核心设计哲学：评测 Agent 在**没有方法签名、没有类骨架、没有需求文档**的情况下，能否真正从零设计一个完整的软件系统。

这揭示了大多数现有 benchmark 的根本缺陷——它们测的是"代码补全"而非"架构设计能力"。评测 Harness 若只测补全，会系统性高估 Agent 的真实软件工程能力。

---

## 二、Common Biases in Agent Evaluation (and How to Avoid Them)

### Bias 1：过拟合于有提示的评测任务

**问题**：当评测任务提供了方法签名、类骨架或需求文档时，Agent 实际上是在做"代码补全"而非"自主设计"。SWE-bench Verified 上某些模型已达 60%+，但这高估了真实软件工程能力。

**规避方法**：引入 ProgramBench 式的"无提示"评测任务，强制 Agent 自主决定抽象层次、模块划分和接口设计。

### Bias 2：互联网访问导致的捷径行为

**问题**：在允许互联网访问的实验中，Agent 找到了大量捷径——从 GitHub 克隆源代码、通过包管理器下载代码。这些行为被 LM-as-judge 标记和排除，但使得 benchmark 不可靠。

**规避方法**：使用沙箱环境禁用互联网访问和反编译，确保测试的是"真实软件构建能力"而非"信息检索能力"。

### Bias 3：单一指标误导（只看 pass@k）

**问题**：只关注 pass@k 而忽略 pass^k，导致对 Agent 稳定性的误判。ProgramBench 明确指出：即使 95% 测试通过（Almost Resolved），剩余 5% 的边界问题可能使整个解决方案完全不可用。FFmpeg 和 PHP Compiler 仅 5% 的得分就是例证。

**规避方法**：同时报告 pass@k（能力上限）和 pass^k（可靠性），并引入"几乎解决（≥95% 通过）"作为诊断性辅助指标，而非主要评测指标。

### Bias 4：评测任务的记忆污染

**问题**：Agent 评测任务中的源代码可能出现在模型训练数据中，导致记忆而非真正的问题解决。

**规避方法**：ProgramBench 的消融实验表明，强制模型用不同语言实现时分数没有显著变化——这证明当前极低分数与源代码记忆无关。这个"不同语言"消融实验是检测记忆污染的有效手段。

### Bias 5：评分器引入的偏见（LLM-as-judge）

**问题**：LLM-as-judge 评分器引入了额外的评分不稳定性——同一输出在不同调用中可能得到不同评分。

**规避方法**：在自动化测试中优先使用 exact_match/regex/command 等确定性评分器；LLM 评分器仅用于开放式任务，并明确标注为"主观评分"而非"客观事实判断"。

### Bias 6：Harness 调优导致的过拟合

**问题**：为单个任务或少数任务大量调优 Harness，会导致 headline scores 高估 Agent 真实能力。ProgramBench 刻意使用单一通用 harness 跨全量任务评估，避免此类过拟合。

**规避方法**：使用固定 harness 配置跨全量任务评测，不针对个别任务做特调。

---

## 三、Evaluation Metrics That Matter vs Metrics That Mislead

### Metrics That Matter

| 指标 | 含义 | 适用场景 |
|------|------|---------|
| **pass^k**（全部通过概率） | 可靠性 | 所有 Agent 评测——是衡量稳定性的核心指标 |
| **pass@k**（至少一次通过概率） | 能力上限 | 作为 pass^k 的补充，衡量"是否能做对" |
| **Resolved**（完全解决率） | 是否完全解决任务 | ProgramBench primary metric——最有意义的最终指标 |
| **Almost Resolved**（≥95% 通过） | 诊断性辅助 | 仅用于诊断，区分模型在边界问题上的表现差异 |

### Metrics That Mislead

| 指标 | 问题 | 原因 |
|------|------|------|
| 平均测试通过率 | 极度误导 | ProgramBench 的任务包含大量简单测试（如 `--help`），平均通过率会掩盖严重的功能性缺陷 |
| 单一模型 pass@k | 过度乐观 | 未考虑模型版本波动、随机性导致的稳定性差异 |
| 未经消融验证的分数 | 可能有记忆污染 | 未排除源代码记忆因素 |

ProgramBench 的设计者明确指出：*"Reporting an average test pass rate would be extremely misleading, because every instance includes very simple tests... And even a single failed test can indicate severe issues with a program."*

---

## 四、Best Practices Summary

### 设计层面

1. **YAML 配置驱动 + 分层 tags**：评测任务与执行引擎解耦，支持 core/regression/safety 分层隔离
2. **双指标监控**：pass@k 和 pass^k 一起看，差距过大时优先解决随机性问题
3. **先写评测集**：从真实失败案例和负面场景构建评测集，而非从理想输入设计
4. **禁用互联网 + 反编译**：cleanroom 环境确保测的是构建能力而非检索能力
5. **单一日用 harness**：跨全量任务使用统一 harness，避免针对个别任务特调

### 工程层面

1. **缓存机制**：Agent 评测成本远高于传统单元测试，缓存机制使"改评分规则 → 重跑评测"的迭代成本从 O(n·API_calls) 降到 O(n)
2. **A/B 对比**：支持 run ID 前缀匹配的 A/B 对比输出，标注每个任务是回归还是改进
3. **扩展接口**：极简 Agent（Execute + Close）和 Grader（Grade + Type）接口降低插件开发门槛
4. **CI/CD 集成**：fail-under 阈值返回退出码，results/summary.json 可被后续流水线步骤消费

### 评测集构建层面

1. **负面案例优先**：Gotchas 是最高价值内容，从 Agent 失败案例中积累
2. **多模型评测**：必须在不同模型上运行评测，因为不同模型对 Agent 的路由和行为表现差异很大
3. **无提示任务**：引入需要自主架构设计的评测任务，而非仅测代码补全
4. **248K+ 行为测试用例**：细粒度边界测试确保"几乎解决"的任务不被误判为"完全解决"

---

## 五、Links to Entities

-  — pass@k/pass^k 双指标体系、多评分器、CI/CD 集成、缓存机制
-  — 多Agent协作、Critic机制、工具生态嵌套Agent模式
-  — 先写评测、Gotchas飞轮、多模型评测必要性
-  — 无提示设计、反作弊、248K+行为测试、双指标体系
-  — Skill质量评估的指标体系
-  — Agent开发规范
- [[entities/agent-memory-architecture|Agent Memory 架构]] — Agent记忆管理机制
-  — 评测闭环思路

---

## 六、Provenance Citations

- [[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation|AgentEval YAML驱动评测框架]]
- [[raw/articles/agent-framework-owl-principles|OWL框架原理]]
- [[raw/articles/aeo-and-geo-for-ai-overviews-chatgpt-claude-gemini-and-perplexity.md|Perplexity Skill设计指南]]
- [[raw/articles/programbench-swe-agent-benchmark|ProgramBench SWE Agent Benchmark]]
