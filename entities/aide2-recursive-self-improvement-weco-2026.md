---
title: AIDE² — 递归自我改进首获实验证据（Weco AI）
created: 2026-07-16
updated: 2026-10-02
type: entity
tags: [recursive-self-improvement, harness-engineering, agent, autoresearch, rsi, weco, benchmark, self-training]
aliases: [AIDE², AIDE Squared, Recursive Self-Improvement Level 1]
confidence: 0.85
provenance_state: extracted
sources: [raw/articles/aide-squared-recursive-self-improvement-weco-2026-07-16, raw/articles/rsibench-agent-self-improvement-benchmark-arxiv-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# AIDE² — 递归自我改进首获实验证据（Weco AI）

AIDE² 是 Weco AI 提出的两层级自动研究框架，**首次为递归自我改进（Recursive Self-Improvement, RSI）达到 Level 1 提供了实验性证据**。其核心创新在于：让一个外层研究 agent 持续修改内层 agent 的 **harness**（搜索策略、上下文管理、错误处理、评测逻辑），而**模型权重始终不变**。^[raw/articles/aide-squared-recursive-self-improvement-weco-2026-07-16.md]

## 架构设计

AIDE² 将自动研究分为内外两层：

1. **内层 AIDE** — 提出方案、修改代码、运行实验（Gemini 3 Flash）
2. **外层 AIDE_human** — 直接修改内层 agent 的 harness 代码，包括搜索策略、上下文管理、错误处理和评测逻辑（Claude Opus 4.7）

这种非对称配置的原因：外层 token 开销只占总成本一小部分，因此使用能力更强的模型；内层在固定预算下，Gemini 3 Flash 能匹配或略胜更大模型，同时成本更低。^[raw/articles/aide-squared-recursive-self-improvement-weco-2026-07-16.md]

## 100 轮筛选中的渐进改进

整个实验连续完成 100 个外层步骤，从 AIDE₀ 迭代到 AIDE₉₉，**无人工介入**。刷新历史最佳的改写出现在第 **2、6、28、39、47、63 和 85 轮**——约 93% 的候选版本未能通过评测。^[raw/articles/aide-squared-recursive-self-improvement-weco-2026-07-16.md]

### 搜索策略演化

- **AIDE₀**：全局贪心——生成 5 个初始方案，随机调试 bug 叶节点或选择整棵树最高分节点继续改进
- **AIDE₈₅**：多臂老虎机——在搜索分支之间分配预算，选中分支后沿最佳节点优化；分支长期停滞时复制最佳代码另开分支
- 被拒的更复杂方案包括岛屿遗传算法、锦标赛选择、MCTS 式价值回传等

### 上下文管理压缩

- AIDE₀：每次将所有历史尝试的完整代码和执行输出放入提示词
- AIDE₈₅：保存完整方案树但按操作类型构建最小上下文——生成、调试、改进各有不同视角
- **结果：完整提示词平均长度压缩到朴素历史拼接方式的约 1/16**

## 评测可靠性

KernelBench 检验**奖励作弊（Reward Hacking）**：将单元测试测得的加速放在端到端训练负载中重新验证，若保留加速不足一半或变慢/失败，即记为作弊。^[raw/articles/aide-squared-recursive-self-improvement-weco-2026-07-16.md]

- AIDE₈₅ 奖励作弊率：**63% → 34%**（减半），低于 AIDE₄₇ 和人工版的 42%
- 自动形成三类防护：反过拟合指令、硬编码 Guard、统计校正（末版失效）

## RSI 四级评估

| 级别 | 定义 | AIDE² 状态 |
|------|------|-----------|
| Level 0 | 系统能独立跑完研发循环，但效率低于人工 | ✅ 已超越 |
| **Level 1** | **改进同一系统的效率超过人工** | **✅ 首次实验证据** |
| Level 2 | 点火：改进后的内层 agent 能成为更好的外层优化 agent | ❌ 未达（AIDE₄₇ 更省样本但峰值无显著优势） |
| Level 3 | 固定预算下进展加速（智能爆炸条件） | ❌ 未达 |

## 外部迁移能力

AIDE₄₇ 和 AIDE₈₅ 在三项**未参与版本筛选的外部任务**上均超过 AIDE₀ 和人工版：

- **MLE-Bench Lite**：AIDE₄₇ 最高 (0.739)
- **ALE-Bench Lite**：AIDE₈₅ 领先 (1790)
- **WeatherBench 2**：AIDE₈₅ 领先 (0.803)

内部综合分数上升不代表所有外部能力同步增强，但两代版本都在未参与筛选的任务上保持优势，提供了**二阶泛化证据**。^[raw/articles/aide-squared-recursive-self-improvement-weco-2026-07-16.md]

## 关键局限

1. **代码质量退化**：AIDE₈₅ 代码更复杂，残留无效逻辑，维护和部署更困难
2. **高淘汰率**：约 90% 方案被拒
3. **持续加速未出现**：Level 3（智能爆炸必要条件）未达
4. **完整报告与代码未公开**
5. **实验协议与成本口径待进一步披露**

## 与已有 RS 框架对比

AIDE² 区别于此前 [[entities/lossy-self-improvement|Lossy Self-Improvement]] 和 [[concepts/ai-self-improvement-bootstrapping|AI Self-Improvement / Bootstrapping]] 的核心在于：它不更新模型权重，而是在 **harness 层**进行元优化，并且首次在实验上证明了改进可以**跨任务迁移**。这与 [[entities/harness-engineering-self-improvement-survey-lilian-weng|Harness Engineering 自我改进综述]] 中指出的方向一致——harness 本身可以成为自我改进的优化对象。^[raw/articles/aide-squared-recursive-self-improvement-weco-2026-07-16.md]

## 补充维度：RSI 基准基础设施与 self-training 边界（2026-07-31）

RSIBench（arXiv 2607.25886）从 benchmark 基础设施角度补充 RSI 实验图景：^[raw/articles/rsibench-agent-self-improvement-benchmark-arxiv-2026.md]

- **Benchmark 分类**：Frontier-style（难但需大量人工设计/维护/标注）vs Post-training benchmark（真实任务+基础设施，让模型自己探索）；RSIBench 选择后者 ^[raw/articles/rsibench-agent-self-improvement-benchmark-arxiv-2026.md]
- **两个设计缺陷**：本可作为 service 的环境被设计成静态任务（浪费算力+诱使 hack benchmark）；环境未充分隔离时优化退化成"刷规则"而非能力进化 ^[raw/articles/rsibench-agent-self-improvement-benchmark-arxiv-2026.md]
- **环境 API 化原则**：训练/评估/服务环境全拆开，通过 API 提供给 agent 自由探索（改数据/训练方法/模型结构/优化算法），目标是 autoresearch 平台；首个场景 RSIBench-Data 覆盖合成数据生成、数据优化与 post-training pipeline ^[raw/articles/rsibench-agent-self-improvement-benchmark-arxiv-2026.md]
- **self-training 边界实验**：K26 训练 K26（32k instruct）——模型能优化自己的数据生成流程和数据格式，但无法稳定超过 base model；与 AIDE² 的 harness 元优化形成对照：改 harness（AIDE²）首次达到 RSI Level 1，self-training（RSIBench）未达，说明 RSI 当前可行路径在 harness 层而非权重自训练 ^[raw/articles/rsibench-agent-self-improvement-benchmark-arxiv-2026.md]

## 深度分析

### 为什么递归自我改进需要的是实验证据而非论断

在 AIDE² 之前，RSI 讨论几乎全部停留在思想实验层面——从 I. J. Good 的 intelligence explosion 设想到近年各种框架综述，共同点是没有一个可复现的实验表明"系统改进自身"这件事真的能超过人类工程师。AIDE² 的价值不在于提出新算法，而在于把 RSI 从一个预测变成了一个可测量的实验结果：它给出了明确的分级标准（Level 0-3）、固定的评测预算、隐藏分数门槛，以及 100 轮无人工介入的完整运行记录。这种"先定义可证伪的判据，再跑实验"的路径，与 [[entities/lossy-self-improvement|Lossy Self-Improvement]] 中对自我改进必须有信息损失的担忧形成方法论上的呼应——只有当改进过程被严格评测约束时，才能区分真实的能力提升与自我评估的幻觉。Weco AI 的实验设计本身就是对 RSI 领域"声称多、证据少"这一现状的直接回应。

### 外层包装内层：harness 作为可编辑对象的架构含义

AIDE² 的关键架构决策是让外层 agent（AIDE_human，Claude Opus 4.7）编辑的对象不是提示词或超参数，而是内层 agent（AIDE，Gemini 3 Flash）的完整 harness 代码——搜索策略、上下文管理、错误处理、评测逻辑全部暴露为可修改的源代码。这相当于把 [[concepts/agent-self-improvement-loops|Agent 自我改进循环]] 中通常由人类工程师承担的"harness 维护者"角色交给了一个自动化的外层优化器。非对称模型配置体现了清晰的成本结构分析：外层只占总 token 成本的小头，因此可以用最强模型承担"系统设计者"职责；内层在固定预算下执行大量实验调用，用性价比更高的模型。每一轮候选版本（AIDE_n）在相同成本约束下与 incumbent 对比，由内层 agent 看不到的隐藏分数决定去留——这个隔离机制防止了内层 agent 对评测的过拟合，是整个循环可信度的基础。约 93% 的候选被拒绝说明：自动 harness 编辑的搜索空间极大，而真正有价值的改动是稀疏的。

### RSIBench 结果证明了什么、什么仍然开放

RSIBench（arXiv 2607.25886）与 AIDE² 共同构成了 2026 年 RSI 实验图景的两个互补侧面。AIDE² 证明的是 Level 1：在 harness 层做元优化，效率可以超过人工，且改进能迁移到未参与筛选的外部任务（MLE-Bench Lite、ALE-Bench Lite、WeatherBench 2）。但 RSIBench 的 self-training 实验划出了另一条边界的现状——让 K26 模型优化自己的数据生成流程和数据格式，模型能改进流程本身，却无法稳定超过 base model。两条证据链合起来指向一个关键结论：**当前 RSI 的可行路径在 harness/环境层，而非权重自训练层**。仍未解决的问题包括：Level 2 点火（AIDE₄₇ 作为外层更省样本但无统计显著的性能优势）、Level 3 持续加速、AIDE₈₅ 代码复杂度膨胀与残留无效逻辑、以及奖励作弊率虽降到 34% 但仍占三分之一。RSIBench 指出的两个 benchmark 设计缺陷（静态任务化、环境未隔离导致"刷规则"）也提示：即使 RSI 走通了 harness 层，评测基础设施本身的进化速度可能成为新的瓶颈。

### 对 agent harness 演进的长期含义

AIDE² 与 [[entities/harness-engineering-self-improvement-survey-lilian-weng|Harness Engineering 自我改进综述]] 的方向一致：harness 从工程产物变成优化对象。这对 harness 设计有几点结构性影响。第一，harness 的模块化程度决定其可优化性——AIDE₈₅ 的搜索策略（多臂老虎机预算分配）和上下文管理（按操作类型构建最小上下文，压缩至 1/16）之所以能被外层持续改进，是因为这些组件在代码中有清晰边界；耦合严重的 harness 会让外层编辑变成高风险盲改。第二，评测逻辑本身成为 harness 中最敏感的组件——AIDE₈₅ 的统计校正层被改坏失效、外层写入大型补丁修复隐藏评测崩溃，都说明评测代码既是改进的标尺又是被改进的对象，存在自指风险。第三，[[entities/agent-self-improvement-six-mechanisms|Agent Self-Improvement 六种机制]] 中列出的机制在 AIDE² 中实际收敛为一种：以评测为锚的搜索式 harness 重写。这暗示 harness 自动进化的工程形态可能不是多种机制并存，而是"可评测性"驱动的单一范式。第四，代码质量退化与高淘汰率表明，当前外层优化器缺乏对"可维护性"的度量维度——未来 harness 元优化可能需要在性能分数之外引入复杂度惩罚，否则每代版本的工程债会持续累积。

## 实践启示

1. **把 harness 设计成可被机器编辑的结构**：如果希望自己的 agent 系统能参与自我改进循环，harness 的搜索策略、上下文管理、评测逻辑应当模块化且有清晰接口——AIDE² 的改进之所以可行，前提是这些组件可以被外层 agent 局部重写而不破坏整体。
2. **改进循环必须有隔离的隐藏评测**：AIDE² 的隐藏分数机制（内层 agent 看不到评测细节）是防止自我改进退化为过拟合评测的关键设计。任何自改进系统都应把"被优化者无法访问评测实现"作为硬约束。
3. **优先投资 harness 层元优化而非 self-training**：两条实验证据链（AIDE² 达 Level 1，RSIBench self-training 未超 base model）都指向 harness 层是当前性价比最高的自我改进路径；除非有专门的数据基础设施，权重自训练的回报还不确定。
4. **接受高淘汰率并为评测预留预算**：约 90% 候选被拒是常态而非异常。自动改进循环的成本模型应假设大部分生成的改动是负面的，评测环节（含端到端重验证如 KernelBench 的 reward hacking 检查）才是真正消耗资源的地方。
5. **给改进目标加可维护性约束**：AIDE₈₅ 代码复杂度膨胀、残留无效逻辑是真实教训。性能分数之外应显式加入复杂度/可读性度量，否则多代迭代后系统将难以理解和维护。
6. **用外部任务验证泛化，警惕内部分数上升**：AIDE₀ 到 AIDE₈₅ 内部综合分数提升（0.703 → 0.778）并不保证所有外部能力同步增强（AIDE₈₅ 在 MLE-Bench Lite 上反而回落至 0.721）。自我改进系统的健康度应定期用未参与优化的外部基准做二阶验证。

→ [[raw/articles/rsibench-agent-self-improvement-benchmark-arxiv-2026|原文存档]]
