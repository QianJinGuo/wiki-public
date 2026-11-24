---

title: "Anthropic 大规模代码迁移方法论 — Claude Code 多 Agent Loop 的工程实践"
created: 2026-07-18
updated: 2026-11-13
type: entity
tags: [agent, claude-code, code-migration, loop-engineering, anthropic, agent-loop, engineering-practice]
sources: [raw/articles/anthropic-claude-code-large-scale-code-migration-2026]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Anthropic 大规模代码迁移方法论 — Claude Code 多 Agent Loop 的工程实践

> **Background**：本文档基于 Anthropic 官方博客 2026-07-16 发布的文章建立，总结了 Anthropic 内部使用 Claude Code 进行大规模代码迁移（Bun Zig→Rust、Mike Krieger Python→TypeScript）的六步方法论。参考了 Anthropic 官方博客多个来源。

## 核心洞见

"You don't fix the code. You fix the process (loop) that produced the code." ^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md]

## 两个关键案例

### Bun Zig→Rust（Jarred Sumner）
- 100 万+ 行代码，< 2 周完成
- 100% 测试套件通过后 merge
- 19 个回归 post-merge，全部修复
- Token 消耗：59 亿 input + 6.9 亿 output（~$165K API pricing）
- 成果：memory leaks 从 6,745MB → 609MB，binary 缩小 19%，性能提升 2-5%

### Python→TypeScript（Mike Krieger）
- 165,000 行 TypeScript 在一个周末完成
- 数百个 agent、8 个 phase gate、3 轮 adversarial review
- 最终 parity check diff 每个命令的输出 vs Python 原版
- 编译时间从 30min → 2s，binary 启动快 6x，退休独立部署 pipeline

## 六步方法论

### 前置：构建强 Judge
Judge 必须能对等地评估原始代码和目标代码。步骤：分类测试 → 重写为可移植断言 → 验证 judge（对好代码通过，对故意写坏的代码失败）。 ^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md]

### Step 1: 创建 Rulebook + Dependency Map + Gap Inventory
- **Rulebook**: 结构保持型翻译用查表规则，重新设计型用设计文档
- **Dependency Map**: 文件依赖分析，决定并行工作流的分片
- **Gap Inventory**: 语言特有的差距（Zig→Rust 是内存管理，Python→TS 是 interface contract）

### Step 2: 压力测试规则
小规模迁移作为"shakedown cruise"：一个 agent 用 rulebook 翻译 3 个文件、一个 agent 像资深工程师一样翻译 3 个文件、一个 agent 用 diff 创建新规则。在大规模部署前捕获关键问题。 ^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md]

### Step 3: 并行翻译所有文件
Multi-agent loop 架构：implement → review → fix。使用小模型做 implementer、大模型做 reviewer。Adversarial review（2 个 reviewer，分歧 → 第三个裁决）。Rulebook 持续增长，代码从不手动修补。当 reviewer 反复发现相同错误时，修改 rulebook 而非逐文件修补，然后重新生成受影响的 batch。 ^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md]

### Steps 4-6: 编译 → 运行 → 匹配行为
同一 loop 架构，逐步减少 human judgment。Fixer agent 处理 compiler errors → smoke test crashes → test suite failures。Build daemon 序列化重建（最昂贵的操作只允许一个进程执行）。 ^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md]

## 最佳实践
- 关注 pattern，而非 individual failures（fixer agent 处理个体问题）
- Adversarial review + mechanical verification
- 不要在所有环节用最大模型（implementer 用小模型，reviewer 用大模型）
- 前置 human hours（rulebook + stress test 最耗时）
- 让工作队列 mechanical & resumable（"完成"=输出文件在磁盘上存在）
- 审查 loop results，而非 individual code

## 深度分析

### 为什么"修 loop 而非修代码"在迁移场景成立

代码迁移的特殊性在于正确性标准是外生且可机械判定的——原实现的行为本身就是规格。这使它成为多 agent loop 的理想靶场：与开放式功能开发不同，迁移的 ground truth 已经存在，judge 可以用测试套件把"对/错"压缩成二值信号。Anthropic 的核心公式——把人力从"改代码"转移到"改产生代码的流程"——本质上是把修复动作从 O(代码行数) 降维到 O(rulebook 规则数)：当 reviewer 反复发现同类错误时，改一条规则再重新生成受影响的 batch，远比逐文件修补便宜。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:26] 这与 [[concepts/loop-engineering-methodology|Loop Engineering 方法论]] 中"错误分类 → 系统级修复"的思路一致。

### Judge 是整个方法的杠杆点

六步流程中唯一的前置环节不是写规则，而是构建能"对等地评估原始代码和目标代码"的 judge：分类现有测试 → 改写为对两种代码库都可运行的断言 → 用故意写坏的代码验证 judge 会失败。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:30-34] 用对抗性方式校准 judge 本身（对好代码必须通过、对坏代码必须失败）是最容易被忽略的一步：judge 太弱，loop 会把错误代码放进 merge gate；judge 太严，修复成本失控。Bun 案例中"100% 测试套件通过才 merge"的硬门之所以可信，正是因为 judge 先经过了校准。

### 并行度来自依赖图，模型分级来自错误成本不对称

Step 1 的 dependency map 不只是文档——它直接决定 fan-out 的分片边界，是"百万行、两周"吞吐的来源。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:36-39] 模型分级（implementer 用小模型、reviewer 用大模型）背后是错误成本的不对称：implementer 写错的代价只是 loop 内一次 review-revise 往返，reviewer 漏检的代价是错误流入下游 batch 甚至 merge gate，因此参数量应花在检出侧而非生成侧。Krieger 案例的 8 个 phase gate 加 3 轮 adversarial review（2 个 reviewer 分歧时由第 3 个 agent 裁决）把这个不对称制度化了。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:24]

### 机械可恢复的任务队列是长跑的前提

"完成 = 输出文件在磁盘上存在"这一定义把工作队列变成了纯文件系统状态机：任何 agent 崩溃后重跑只会补做缺失的输出，不会重复已完成的工作。对于 59 亿 input token（~$165K API pricing）量级的作业，这不是优化而是必需品——任何非幂等调度在小时粒度的失败面前都会放大成本。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:23] Build daemon 把最昂贵的重建操作收敛为单进程串行执行，与 [[concepts/harness-loop-architecture|Harness Loop 架构]] 中"昂贵操作由受控 harness 排队"的做法同构。

### 结果的信号意义

Bun Rust 移植后 unsafe 块仅占 4%，2,000 次构建的内存泄漏从 6,745MB 降至 609MB，binary 在 Linux/Windows 缩小 19%，HTTP 服务等真实负载提速 2-5%。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:59] 这些不是"AI 写的代码勉强能用"的信号，而是"agent 生产 + 机械验证"的产物在内存安全等硬指标上达到人工迁移常见水位之上的证据。19 个 post-merge 回归全部修复则说明同一套 loop 在 merge 之后仍可复用于维护期。

## 实践启示

1. **先建 judge，再写规则。** 迁移项目的前期投入应优先给 judge：盘点现有测试、改写为可移植断言、并用故意写坏的实现验证 judge 会 fail。judge 不可靠时，后续所有 phase gate 都只是仪式。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:30-34]
2. **用 3 文件小迁移当 shakedown cruise。** 大规模 fan-out 前做三臂对照：rulebook 翻译 3 个文件、资深工程师式翻译 3 个文件、用 diff 生成新规则，把 rulebook 的系统性缺陷在个位数文件的成本下暴露掉。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:41-42]
3. **同类错误出现两次以上，修 rulebook 而非修文件。** reviewer 反复命中同一类错误是规则缺失的信号；改规则后重新生成受影响 batch，永远不要手动修补单个文件——手工补丁会让代码库偏离 loop 的可再生产性。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:44-45]
4. **按错误成本分配模型预算。** implementer 用小模型压成本，reviewer 用大模型保检出率；两个 reviewer 分歧时升级第三个 agent 裁决，而不是让人类逐 diff 仲裁。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:44-45]
5. **把"完成"定义为磁盘上的输出文件。** 任务队列用文件存在性做幂等标记，任何 agent 崩溃后可直接重跑；在数天、数十亿 token 的作业里，这决定了失败成本是常数而非累积。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:55]
6. **迁移结束不等于 loop 退役。** Bun 的 19 个 post-merge 回归全部由同一套 fixer loop 修复；把 loop 保留为维护期工具，回归处理也从"人读代码"变成"人审 loop 结果"。^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md:23]

## 与其他实体的关系

本实体提供了 **多 agent loop 用于生产级代码迁移** 的具体方法论，与以下实体互补：

- [[entities/claude-code-dynamic-workflows-thariq-blog-gaia.md|Claude Code Dynamic Workflows]]（动态工作流基础）
- [[entities/claude-code-loop-types-official-taxonomy-four-modes.md|Claude Code Loop Types]]（loop 类型官方分类）
- [[entities/agentic-loop-engineering-handbook-empirical-framework.md|Agentic Loop Engineering]]（loop 工程框架）
- [[entities/claude-code-engineering-truth-1.6-98.4.md|Claude Code Engineering Truth]]（工程实践）
- [[entities/claude-code-architecture.md|Claude Code 架构]]

→ [[raw/articles/anthropic-claude-code-large-scale-code-migration-2026|原文存档]] ^[raw/articles/anthropic-claude-code-large-scale-code-migration-2026.md]