---
title: "Claude Code到底有多费token？对比实验来了：三大框架最多差30倍"
created: 2026-07-31
updated: 2026-11-13
type: entity
tags: ['harness', 'token-efficiency', 'claude-code', 'hermes', 'kimi-code', 'composio', 'benchmark', 'agent-cost', 'harness-tax']
sources: [raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026.md|原文存档]]

机器之心转述 Composio 团队对比实验：用同一个模型（Kimi K3）放入三个不同的 agent 框架（harness）——Claude Code、Hermes、Kimi Code——跑 28 个任务。核心发现：**成功率差不多，token 消耗最多差 30 倍**。结论：想降低 agent 成本，先看 harness 而不是急着换模型——harness 本身能把成本拉开 9 倍，而模型能力表现差不多。 ^[raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026.md]

## 深度分析

### 输入 token：分差的主战场

三个 harness 跑同样的 Kimi K3，成功率几乎打平（20/28、21/28、22/28），token 中位数却拉开约 6 倍（约 6.1 万、6.7 万、34 万），最省与最费的极端个案相差 30 倍 ^[raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026.md]。Raschka 用 Qwen3.6 做的独立观察印证了方向：成功率相近时 Claude Code 的 token 用量常是其他 harness 的 2-3 倍，且差异出在输入而非输出——模型并没有多写一倍的代码 ^[raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026.md]。分差的根源在 harness 的上下文组装策略：每次工具调用后如何拼接系统提示、对话历史与工具结果；失败后的重试是把完整上下文原样重灌还是做增量裁剪；工具输出（大文件读取、长命令输出）是全文进上下文还是按预算截断。34 万 token 的中位数意味着几乎每一步都在重付一笔与任务难度无关的"背景成本"，纯粹由 harness 的组装习惯决定。换言之，同一个模型在不同 harness 里"看到"的世界完全不同：一个每步只携带最小工作集，另一个则拖着全量历史前进，账单差距在任务开始前就已注定。

### 成功率为何掩盖成本维度："harness 税"与基准设计

单看成功率，三个 harness 看起来等价（最多差 2 题）；把 token 和成本放进来，排名才显形——单位成本差 6 倍以上。社区由此提出 **harness 税** 指标：一旦工具调用与重试进入循环，这笔税不是线性增长，而是随循环次数复利放大 ^[raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026.md]。对基准测试的含义是：只报 pass/fail 会系统性偏向上下文最"慷慨"的 harness，因为更多输入往往换来对难任务更高的边际成功率，但代价被隐藏。像 [[entities/arena-ai-harness-tax-coding-agents|harness 税]] 与 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准]] 所主张的，评测应报告 tokens-per-success 与成本/任务，让"做成"与"花多少做成"同时可见。

### 95% 输入占比：agent 成本的结构性构成

agent 工作流中输入 token 约占 95%，说明成本大头不是"生成"而是"喂给" ^[raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026.md]。输入由三块构成：系统提示（每次请求重复计费）、工具 schema 定义（几十个工具的描述常驻上下文）、文件与命令输出的原文 dump。这解释了两件事：其一，为什么最贵的 harness 恰是上下文最不节制的那个；其二，为什么 [[concepts/context-management-agent-systems|上下文管理]]、prompt caching（参见 [[entities/anthropic-prompt-caching-claude-code|Anthropic Prompt Caching]]）与按需注入工具描述是 harness 优化的第一杠杆——它们直接砍掉占 95% 的那部分。顺带一提，这也是推理服务商持续压低输入单价的商业动因——当输入是成本的绝对主体，输入侧的每一分折扣都会被 harness 的循环结构成倍放大。

### Writer 控制变量实验：harness 是独立于模型的优化层

Writer 的实验补上了另一半证据：固定 harness 层，22 个企业任务 × 6 个基础模型（Claude Sonnet 4.6、Gemini 3.1、Gemini Flash 3.5、Qwen 3.6、GLM 5.1、Palmyra X6），只优化 harness——成本 -41%、中位延迟 -44%、token -38%，完成质量 0.78→0.81 基本持平 ^[raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026.md]。所有模型同向受益，说明收益来自 harness 层本身，而非某个模型的特殊配合：harness 与模型是两个可分离的优化变量。与 Composio 的实验合看构成完整矩阵——Composio 固定模型换 harness，量出分差有多大（9 倍成本）；Writer 固定 harness 换模型并优化层，量出收益有多少（-41% 成本）。两个方向都指向同一结论，即 [[concepts/harness-as-product-surface|Harness 即产品]]：模型决定能力上限，harness 决定成本下限，这也是 [[entities/agent-harness-12-components-7-decisions|harness 组件设计]] 的核心命题。

## 实践启示

1. **排查成本先测 harness 税**：同一模型同一任务在不同 harness 各跑一遍再谈模型选型——token 中位数可差 6 倍、极端差 30 倍，成本问题的第一嫌疑人是 harness 而不是模型。
2. **评测必须报 tokens-per-success 与成本/任务**：只报成功率会漏掉主要分差；20-22/28 的能力差距远小于 6 倍的成本差距，选型应按"每做成一个任务花多少钱"排序。
3. **把输入 token 当第一优化对象**：压缩系统提示、裁剪工具 schema、按需加载工具描述、给大文件读取设截断预算——95% 的输入占比决定了杠杆位置；再叠加 prompt caching 把重复输入打折。
4. **给重试循环设上下文策略**：失败重试前对历史做裁剪或摘要，只重灌与失败相关的最小上下文，否则 harness 税随 loop 非线性放大——每一次失败的探索都可能变成后续所有步骤的固定背景成本。
5. **最快与最省不是同一个 harness**：Hermes 中位耗时最短（179 秒）但 token 不是最少（Kimi Code 约 6.1 万）——延迟敏感与成本敏感是两个选型维度，按业务目标分别选。
6. **优化顺序：先 harness 后模型**：Writer 数据显示仅 harness 层即可拿到 -41% 成本且质量持平；模型升级的正交收益应叠加在这个更便宜的基础之上，而不是替代它。

## 来源

- 原文: [[raw/articles/claude-code-token-cost-harness-comparison-30x-jiqizhixin-2026.md|Claude Code到底有多费token？对比实验来了：三大框架最多差30倍]]
- 原始链接: : "https://mp.weixin.qq.com/s/G29lkLZbQo59_s6wHGhAfQ
