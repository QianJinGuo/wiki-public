---
title: "Autoresearch: AI Agent-Driven Algorithmic Development (File Compression Experiment)"
created: 2026-07-04
updated: 2026-09-13
type: entity
tags: [agent, coding-agent, claude-code, ai-research, autonomous-agent, autosearch]
sources: [raw/articles/autoresearch-claude-constrained-optimization-elliotcsmith-2026]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Autoresearch: AI Agent-Driven Algorithmic Development (File Compression Experiment)

> **Background**: This entity documents Elliot Smith's Autoresearch experiment, where Claude Code was used autonomously to develop a file compression algorithm over 10 iterations. The work is in the tradition of Karpathy's "Autoresearch" concept — using AI agents to autonomously explore and solve constrained engineering problems. ^[raw/articles/autoresearch-claude-constrained-optimization-elliotcsmith-2026.md]

## 实验设计

Smith 选择了文件压缩作为测试问题，因为其目标函数简单（文件体积越小越好）且约束明确（位完美往返 + 压缩/解压不超过 300 秒）。使用 Claude Code (Sonnet 4.6) 默认设置，Rust 语言（类型系统自动强制执行签名约束）。^[raw/articles/autoresearch-claude-constrained-optimization-elliotcsmith-2026.md]

实验基础设施：
- 压缩/解压函数桩（零压缩 baseline）
- 单元测试（字符串 + 简单文件的往返验证）
- Benchmark 脚本：从公共领域收集视频/音频/文本/随机数据样本
- 300 秒超时保护（防无限循环）
- 每次迭代前清除 Claude context → 自动生成 plan → 接受后让 agent 自主运行^[raw/articles/autoresearch-claude-constrained-optimization-elliotcsmith-2026.md]

## 结果

10 次迭代跨 ~2 周运行（主要受 Claude Code 额度限制）。最终结果：^[raw/articles/autoresearch-claude-constrained-optimization-elliotcsmith-2026.md]

- 相比基线压缩率提升 **34%**
- 最佳迭代组合了 LZ4 风格字典编码 + bit-packing 层
- 对比生产工具：达到 zstd 压缩率的 68%，但解压速度快 2x
- 所有代码开源：github.com/smitec/agent-compression

## 方法论意义

这个实验的核心价值在于展示了 AI agent 可以在几乎无人工干预的情况下，自主开发一个有实际意义的算法解决方案。与传统的基于梯度的 ML 优化不同，agent 通过代码修改和实验测试来搜索设计空间。^[raw/articles/autoresearch-claude-constrained-optimization-elliotcsmith-2026.md]

Smith 的基本方法是：提供一个带约束的工程问题框架（类型签名 + 测试 + 基准），让 agent 自主迭代改进。这与 `loop-engineering` 中的"开发循环"范式一致——但这里的 agent 是编码者而非辅助工具。

## 与同类工作的关系

- 与 [[entities/headroom-context-compression-cache-stabilization|Headroom]] 不同：Headroom 关注 LLM 上下文压缩，这里关注通用数据压缩
- 与 `loop-engineering` 模式相关：agent 自主迭代改进的自动化循环
- 与 Karpathy 的 Autoresearch 概念一脉相承：让 Agent 自主进行端到端研究

→ [[raw/articles/autoresearch-claude-constrained-optimization-elliotcsmith-2026|原文存档]]

---
## 深度分析

### 约束下的自主搜索究竟展示了什么

Smith 的实验最容易被误读为"AI 写出了一个压缩算法"，但真正的信号不在这份算法本身，而在于搜索过程的形态：agent 在固定目标函数（文件越小越好）与硬约束（位完美往返、300 秒时限）下，通过反复修改代码、运行基准、读取失败反馈来迭代，而不是靠梯度计算或预先设计的启发式。这更接近一种"经验性设计空间搜索"，其单位不是参数更新，而是一次可编译、可测量的代码提交。因此 34% 的改进并非某个单点灵感，而是十轮淘汰后留下的最优点。理解这一点，才能把"agent 自主研究"从营销话术还原为一个有明确输入、明确度量、明确停止条件的工程循环。

### 可验证性是自改进得以闭环的前提

这个案例之所以能近乎无人值守地推进，关键不在模型多强，而在于任务自带一个廉价、确定、无法被话术欺骗的裁判：解压结果必须与原文逐字节相等，一旦不等就是硬失败；压缩比可以量化下降，超时可以直接终止。正因为存在这样的 oracle，agent 才能在没有人类逐轮审查的情况下判断自己是否真的变好，也才不会被"看起来更优"的输出误导。反过来说，凡是没有低成本自动判定成败手段的领域，自主循环就会退化为自我表扬——agent 优化的是措辞而非结果。可验证性因此不是工程细节，而是自主研究能否成立的前置条件。

### Harness 的功劳常常大于模型

实验中真正决定上限的，是 Smith 在 agent 入场之前搭好的脚手架：Rust 的类型系统把"不得改动函数签名"变成编译期强制约束，成对单元测试封死了"能压不能解"的退化路径，benchmark 脚本统一了数据与口径，300 秒护栏杜绝了以时间换空间的作弊。这些都不属于模型能力，而是 [[concepts/harness-engineering-framework|Harness Engineering]] 的产物。它提示我们：同一模型在不同脚手架下的表现可以天差地别，因此评测"agent 能不能做研究"时不先交代 harness，结论几乎没有可比性。提升自主研究的能力，很多时候是改进环境而非改进模型。

### 为什么算法任务是有利的试验场

文件压缩恰好落在"目标清晰、反馈即时、结果可重现"的窄带上，这让它成为理想试验场，却也天然偏向乐观：正确性由字节比对裁决，性能由计时器裁决，失败模式立刻暴露，且全程不依赖主观品味。相比之下，产品设计、系统架构、战略判断等任务没有如此干净的 oracle，成功标准往往滞后数月甚至相互冲突。因此本实验有力地证明了"在可完全验证的窄任务上自主循环可行"，但它并没有、也不应被解读为"自主循环可推广到任何工程问题"。[[concepts/evaluation-harness-design|评估 Harness 设计]] 的质量，直接决定了这类结论的迁移半径。

### 从窄任务到开放式工程的边界

真正的鸿沟在于验证成本与问题定义的稳定性。算法任务的目标函数在循环开始前就已冻结，而开放式工程中"做什么才算对"往往要在做的过程中才逐步浮现，agent 每推进一步都可能重新定义成功。此时自主循环面临两类风险：一是 [[concepts/agent-self-improvement-loops|自改进循环]] 在弱反馈下追逐局部指标而偏离真实价值；二是搜索空间的开放程度超出任何自动基准的覆盖，agent 的"改进"无法被现有测试证伪。可行的迁移方向不是直接照搬，而是先为开放领域人工构造代理指标与分阶段验收，把不可验证的大问题切成一串可验证的小问题，再逐步交还自主权。

## 实践启示

1. **先冻结目标与约束，再放 agent 入场。** 启动任何自主优化循环之前，把成功度量、硬性约束与停止条件写成可执行的检查，而不是留在人的脑子里；目标函数越早确定，循环越不容易漂移。

2. **投资一个廉价且不可欺骗的验证器。** 位完美往返这样的 oracle 是整个循环的支点。若领域没有现成验证器，先花力气构造测试、基准或形式化校验，再谈自动化——没有裁判的自主循环只是自说自话。

3. **把约束下沉到基础设施层。** 能用类型系统、接口签名、CI 门禁强制执行的规则，就不要依赖 prompt 叮嘱；机制比叮嘱可靠，也省去逐轮人工核对（见 [[concepts/harness-engineering-framework|Harness Engineering]]）。

4. **为退化路径设护栏。** 显式加入超时、资源上限与"不得牺牲正确性换性能"的硬检查，防止 agent 用时间换空间、或以压缩换丢失数据的作弊式优化。

5. **每轮清空上下文并重写计划。** 参考 Smith 在每次迭代前重置上下文、生成新 plan 的做法，避免历史偏见累积，让每轮都在干净前提下重新评估方向（见 [[concepts/agent-self-improvement-loops|自改进循环]]）。

6. **如实划定泛化边界。** 在窄且可验证的任务上成功后，不要默认结论可以平移到开放式工程；先量化验证成本与目标稳定性，再决定哪些环节可以交还自主权（见 [[concepts/autonomous-agent-systems|自主 Agent 系统]]）。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

