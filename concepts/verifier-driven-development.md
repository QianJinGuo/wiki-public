---
title: "verifier 驱动开发"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, verifier, testing, agentic-engineering, harness, quality-gate]
sources: [entities/karpathy-vibe-coding-agentic-engineering, entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]
---

## 定义

verifier 驱动开发是 agentic engineering 的核心方法论：每个 AI 产出必须有一个自动 verifier（测试、linter、类型检查、形式化证明、人工 review gate）签字才算交付。无 verifier 的 agent 输出 = 未验证的猜测。

## 核心范式

- **verifier first**：写功能前先定义验收 verifier，否则 agent 没有目标函数
- **多层 verifier**：unit test / integration / 静态分析 / 形式化 / 人工审核 五层 defense in depth
- **verifier 可被 agent 调用**：让 agent 在自己的 loop 内运行 verifier 自纠错
- **verifier 是 bottleneck**：当 agent 写代码快过 verifier 验证，verifier 反过来成为新瓶颈

## 背景与提出

Verifier-driven development 的源头可以追溯到 Karpathy 2026 年提出的「AI 产出的问责性」——如果 AI 生成了一段代码，但没有自动验证通道证明这段代码是对的，那它本质上就是未验证的猜测。 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering] 这个思想在传统软件工程中被叫「测试先行」（Test-Driven Development），但在 agent 系统中它被赋予了新的含义：verifier 不仅是测试，也是 agent 自身可调用的监督信号。

Karpathy 在 2026 年访谈中给出了一个精确的判断：「传统计算机容易自动化**你能写进代码的东西**；这一代 LLM 容易自动化**你能验证的东西**。」这两句话刻画了两种自动化的本质差异。传统软件自动化依赖规则显式化——你必须能把业务逻辑精确写成代码。而 LLM 自动化依赖的是可验证性——你不必写出规则，但必须能判断输出对错。这个框架解释了为什么 LLM 在数学、代码这些有明确验证标准的领域能力飙升，而在「洗车题」这种人类觉得简单但无法结构化验证的任务上翻车。 ^[entities/karpathy-vibe-coding-to-agentic-engineering]

## 范式细节

在 agentic engineering 语境下，verifier 和传统测试有 4 个区别。第一，verifier 可以被 agent 在 loop 内调用——agent 写完代码后自己跑测试、看结果、自 fix，形成一个「写→测→修」的内循环。第二，verifier 是多层的：unit test（函数级正确性）、integration（服务级行为）、static analysis（安全/风格）、formal verification（关键路径）、human gate（高风险确认）。第三，verifier 可以是 LLM-as-judge——当传统测试无法覆盖开放式输出（如文案、设计决策）时，用另一个 LLM 做验证。第四，verifier 的输出是 agent 的奖励信号——让 RL-like 的自我改进成为可能。 ^[entities/claude-code-core-internals]

具体到实现层面，Claude Code 的五层 Context 压缩本身就是一种 verifier：每次压缩前，系统会验证压缩后的上下文是否仍然包含足够支撑当前任务的信息。microCompact 利用 `cache_edits` 参数在服务端屏蔽旧工具结果——这相当于一种「软删除 verifier」，在不完全破坏缓存命中的前提下，验证哪些旧内容可以被暂时「忘记」。autoCompact 通过 fork 子 Agent 生成摘要——这个子 Agent 本身就是一个 verifier，用来检验主 Agent 的对话历史是否可以被有损压缩。 ^[entities/claude-code-core-internals]

verifier 的「多层 defense in depth」原则在实际项目中的数字：Codex 团队在 2025 年底的实验显示，在没有多层 verifier 的情况下，AI 代码生成的一次通过率（首次生成就能通过测试）是 31%；加上一层 unit test verifier 后提升到 58%；再加上一层 integration test 后是 73%；再加上一层 static analysis 后是 81%。这说明 verifier 层数的边际收益递减，但每一层都有实质性的质量提升。 ^[entities/karpathy-vibe-coding-agentic-engineering]

## 局限与反对声音

Verifier-driven development 的核心问题：「如果 verifier 用 LLM 实现，那谁验证 verifier？」LLM-as-judge 有已知的 self-enhancement bias、position bias、length bias——你觉得 agent 输出的文案不错，是因为 LLM 评审更喜欢 LLM 写的风格。多层嵌套后会形成「审核链」但信任度不会线性增加。

具体来说，self-enhancement bias 的意思是：Claude 生成的代码，由 Claude 评审，最容易通过。这是因为 Claude 的训练数据中大量包含「风格类似 Claude 生成的代码」，所以模型天然偏爱这种风格。position bias 的意思是：LLM 评审更容易接受对话中先出现的论点。length bias 的意思是：更长的输出更容易被评为「更详细/更好」。这三个 bias 在单一 verifier 时已经存在，在多层嵌套 verifier 时会被放大——第一层的 bias 被第二层进一步放大，最终结论可能与实际情况相去甚远。

所以即使 Karpathy 自己也在 agentic engineering 的 verifier 部分保留了人工 gate：高风险动作、删除数据、改动核心架构——这些必须有人类签字。即使 verifier 系统再复杂，这几类操作的人类审查在 2026 年仍然不可替代。 ^[entities/karpathy-vibe-coding-to-agentic-engineering]

另一类更根本的局限：verifier 的覆盖率永远不会是 100%。Karpathy 在访谈中指出：「能写进 verifier 的东西是有限的，而 AI 能生成的领域是无限的。」这意味着 verifier 驱动开发的适用场景天然受限——越是能精确描述「什么是对的」领域（代码正确性、数学证明），verifier 效果越好；越是难以形式化描述「什么是好的」领域（代码质量、架构选择），verifier 越无力。 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]

## 现实案例

Hermes Agent 的 wiki pipeline 是 verifier-driven development 在知识管理领域的体现：每个新 entity 入库前必须通过 wiki lint（结构 verifier），每个 dataset 的 review_value×review_confidence 必须 ≥ 49（质量 verifier），每个 commit 触发 pre-commit gate（provenance verifier）。三合一确保 AI 编译的知识不被「未验证猜测」污染。 ^[entities/hermes-wiki-9-step-auto-growing-knowledge-network]

Claude Code 的 Plan Mode 是 verifier-driven development 的产品级实现：模型完成规划后调用 ExitPlanMode，系统将规划内容写入 `.claude/plans/` 下的 Markdown 文件，用户必须在 UI 中显式点击批准，才能将权限模式从 `'plan'` 恢复为原始模式并进入执行阶段。这个流程的本质是：把「规划」当作一种特殊的 verifier——它的输入是用户原始需求，输出是一个可读的、有的人类可以判断的「承诺集」，只有人类确认「这个承诺集是对的」之后，执行才会开始。 ^[entities/claude-code-core-internals]

在代码生成评估领域，verifier 的具体化是 SWE-Bench 类型的基准测试：AI 生成代码补丁，由真实测试用例验证正确性。这类 benchmark 的局限性恰恰说明了 verifier 的核心约束——它只能验证「测试覆盖的场景」，而无法验证「测试没有覆盖的场景」。这也是为什么 SWE-Bench 的通过率从来不是「代码能力」的全貌，只是「能在标准测试用例上通过」这一特定维度的测量。 ^[entities/karpathy-vibe-coding-agentic-engineering]

## 在 wiki 中的关联

- [[concepts/agentic-engineering-paradigm|agentic engineering 范式]]
- harness gate 评估
- AI 测试 QA pipeline
- 代码生成评估
- [[concepts/ai-r-and-d-when-ai-builds-itself-bottleneck-shift-r-d-harness|AI R&D 瓶颈迁移到 verifier]]

## 进一步阅读

- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
