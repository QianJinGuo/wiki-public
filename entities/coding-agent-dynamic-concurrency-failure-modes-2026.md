---
title: "Coding Agent 动态并发实证：354 任务 2124 次执行的收益、28 种失败模式与三类有效协作机制"
created: 2026-10-24
updated: 2026-11-15
type: entity
tags: [agent, coding-agent, multi-agent, subagent, harness, agent-orchestration, benchmark, evaluation, agent-failure-modes]
sources: [raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026]
confidence: 0.85
provenance_state: extracted
---

# Coding Agent 动态并发实证：354 任务 2124 次执行的收益、28 种失败模式与三类有效协作机制

> 南京大学与 UIUC 的配对实验研究（arXiv 2610.10263，数据与轨迹 artifact 见 https://github.com/schwerli/Concurrency-Failures-Trajectory-Artifact）。对 Codex、Claude Code、Kimi Code 三种前沿 Coding Agent 在「开/关动态并发」两种模式下做 354 个软件工程任务、2124 次执行的对照，并人工标注 1062 条并发轨迹，给出 4 大类 / 13 子类 / 28 种失败模式，以及并发真正产生价值的三类协作机制。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

## 问题设定：从静态工作流到动态并发

ChatDev、MetaGPT 一类系统采用**静态并发**——预先规定谁规划、谁实现、谁测试，信息传递与同步方式在任务开始前固定；而 Codex、Claude Code、Kimi Code 正在转向**动态并发**（Dynamic Concurrency）：主 Agent 在长周期执行过程中自主决定是否创建子智能体、何时创建、如何协调与整合结果。三种机制的代表形态分别是 Codex Multiagent V2（主 Agent 继续执行的同时动态创建子智能体，消息交换协调）、Claude Code Dynamic Workflow（动态生成工作流脚本，组织有依赖关系的执行阶段）、Kimi Code AgentSwarm（批量委派一批不同目标的子智能体并行执行）。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

评测设计上，研究覆盖四个长周期 Benchmark：RepoZero（跨语言重建完整代码仓库）、ProgramBench（据参考程序与文档重新实现完整程序）、NL2Repo（自然语言描述生成代码仓库）、LoopsBench（需持续推进、维护任务依赖的软件工程过程），并引入 SWE-bench Verified 作为相对短周期、目标明确的软件修复对照。每个任务由三种 Agent 分别在并发与顺序模式下执行，合计 2124 次执行、约 112 亿 Token、按标准 API 价格约 20,780 美元成本。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

## RQ1：并发收益高度依赖任务形态，成本却普遍上升

- **长周期任务出现收益**：LoopsBench 上 Claude Code 完成率 7.1% → 21.4%（+14.3pp），Codex 21.4% → 25.0%，Kimi Code 维持 25.0% 但平均测试通过率 62.5% → 66.6%；NL2Repo 上 Codex 34.6% → 57.7%。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]
- **同一 Benchmark 上不同机制结果相反**：NL2Repo 上 Kimi Code 反而从 57.7% 降到 46.2%。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]
- **短周期任务普遍受损**：SWE-bench Verified 上 Codex 前后均为 71.0%，Claude Code 83.0% → 59.0%，Kimi Code 77.0% → 59.0%；目标明确、改动范围小时，拆分与协调成本吃掉并行收益。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]
- **总计与成本**：354 个任务整体，Codex 并发模式多完成 8 个（未达统计显著），Claude Code 与 Kimi Code 分别少完成 42 和 19 个（显著下降）；Token 消耗为顺序模式的 1.41–3.31 倍，15 组 Agent×Benchmark 组合中 14 组平均运行时间反而增加（如 Claude Code 在 SWE-bench Verified 上 17.5min → 27.8min）。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]
- **但并发提供了顺序模式拿不到的能力**：LoopsBench 上 Claude Code 有 4 个任务只在并发模式下成功；366 组两模式均成功的配对中 102 组并发更快，其中 46 组至少缩短 20%。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

## RQ2：Agent 数量不等于并发效率

研究区分「累计创建的子智能体数」与「真正同时在执行、时间重叠的子智能体数」，前者可以很高而后者接近 0。1062 条并发轨迹显示：Codex 倾向维持小规模子智能体群组，Claude Code 通过多个工作流阶段持续创建，Kimi Code 一次性启动较大批次。高难度 LoopsBench 任务上 Claude Code 平均创建约 134 个子智能体、Kimi Code 约 67 个，但并发峰值 Kimi 更高。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

时序上，首次委派发生位置的中位数：Codex 约在执行过程 4.0% 处，Claude Code 22.1%，Kimi Code 26.0%（Codex 可在主 Agent 继续执行时直接启动子智能体，后两者需先组织工作流或任务批次）。但「至少两个子智能体同时存在」的时间占比中位数为 Codex 23.9%、Claude Code 52.7%、Kimi Code 15.6%——启动更早不等于重叠更充分。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

## RQ3：失败发生在协作层，而非模型编程能力

对 1062 条并发轨迹人工标注（含独立复核），形成 4 大类、13 子类、28 种失败模式，出现在 650 条轨迹、共 804 次实例中；出现失败模式不等于任务失败，部分成功任务仍带重复工作、文件冲突与额外集成成本。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

| 类别 | 占比（实例） | 子类别要点 | 典型证据 |
|------|------|------|------|
| 任务编排问题 | 28.5% | Missing Owner（任务缺负责人）、Pseudo-Concurrency（伪并发）、Load Imbalance | Kimi Code 在 ProgramBench 的 tailspin 重建中创建 17 个子智能体只做「调查」，实现仍走串行，最终 0/785 测试通过 |
| 执行治理问题 | 29.0% | 启动/汇合时机错误、失败后工作丢失、主 Agent 缺少主动监控 | Kimi Code 在 seqtk 重建中把 22 项功能分给 20 个子智能体（11s 内全启动），却设置了超过任务总预算（40min）的两小时等待，最终 92/440 测试 |
| 全局上下文管理问题 | 9.3% | 任务要求缺失、接口约定不清、交接负担过重、缺全局验证、重试不继承进展 | Claude Code 在 RepoZero 的 base58 迁移中交接约 8 万字符说明，子智能体仍重做环境检查与源码分析 |
| 共享状态与代码集成问题 | 33.2% | Concurrent Writes（并发写入冲突）、Shared Environment Contamination（共享环境污染） | Codex 在 Clipper 迁移中主 Agent 与子智能体互相覆盖源码与项目配置；Claude Code 两个子智能体用相同临时目录路径导致测试数据被改写 |

这四类问题与传统并发程序的数据竞争同构，但冲突主体变成「自主编写和修改代码的 Agent」，因此文件所有权、环境隔离与结果集成成为可靠性要素。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

## RQ4：三类真正产生价值的协作机制

从 131 组存在并发收益的配对轨迹中归纳出三类机制：**互补工作**（任务可拆为相对独立且能正确集成的部分，如 justext 任务由主 Agent 实现核心算法、8 个子智能体分头交付文件与检查，执行时间 734s → 468s）、**并行探索**（多个 Agent 同时尝试不同方案再由主 Agent 比较整合，如 Halite 重建中并发版通过 292/391 测试而顺序版构建失败）、**独立验证**（子智能体独立检查并补上主 Agent 缺失的关键证据，如密码学子智能体指出 ECDSA nonce reuse 场景下 P-256 曲线阶数要求，使恢复算法得以修正）。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

三类机制的共同前提是：主 Agent 必须能把多个执行过程的结果**整合进最终产物**；否则并发只是把 Token 换成并行调查。

## 深度分析

### 并发收益取决于任务可分解性，而非"并发开关"本身

同一 Benchmark 上三种机制可以给出方向相反的结果：NL2Repo 上 Codex 从 34.6% 升至 57.7%，Kimi Code 却从 57.7% 跌至 46.2%。这说明并发的价值不是打开某个开关就能兑现的，而是任务形态与协作机制是否匹配的结果——可拆分为相对独立、能正确集成的子问题时并发才有机会；短周期、改动集中的修复型任务里，拆分与通信开销稳定地吃掉并行收益（SWE-bench Verified 上两家 Agent 下降 18-24pp，一家纹丝不动）。另一个容易被忽略的事实是：整体成功率没涨，并发仍可能带来能力增量——Claude Code 在 LoopsBench 上有 4 个任务只在并发模式下成功，这意味着并发改变了"能解什么"，而不只是"解多快"。

### 调度行为的三种原型：累计创建数与时间重叠数是正交指标

RQ2 最大的方法论贡献是把"并发程度"操作化：累计创建的子智能体数可以很高而真实重叠接近于零。三种 Agent 恰好构成三种调度原型——Codex 极早启动（首次委派中位数 4.0%）但重叠占比仅 23.9%；Claude Code 晚启动（22.1%）却维持 52.7% 的重叠；Kimi Code 批量启动、峰值最高但重叠只有 15.6%。LoopsBench 高难任务上 Claude Code 平均创建约 134 个子智能体，若这些子智能体只做串行调查的放大版，数量本身就是开销。评价一个 Harness 的并发能力，应当直接测量"至少两个子智能体同时存在的时间占比"与"产出是否被整合"，而不是数创建了多少个。

### 失败分类学揭示：协作层取代模型能力成为可靠性前沿

28 种失败模式中占比最高的两类——共享状态与代码集成（33.2%）和执行治理（29.0%）——都不涉及模型的编程能力，而涉及系统设计。这与传统并发程序的数据竞争同构，但冲突主体从线程变成了自主写代码的 Agent，因此文件所有权、临时环境隔离与结果汇合时机从工程细节升级为可靠性要素。两个极端案例值得单独记住：tailspin 重建中 17 个子智能体只做调查、实现仍走串行，最终 0/785——伪并发的典型形态；seqtk 重建中 20 个子智能体 11 秒内全部启动、分工合理，但主 Agent 设了两小时等待而任务预算只有 40 分钟，最终 92/440——回收时机是与分配时机同等重要的决策。失败标注还有一个反直觉之处：出现失败模式不等于任务失败，成功任务的轨迹里也普遍存在重复工作与额外集成成本，即并发在"成功"背后的隐性税。

### 上下文交接悖论：传递更多信息不等于更高效的利用

RepoZero base58 迁移案例中，工作流把调查结果汇总成约 8 万字符的说明交给子智能体，后者仍重新执行环境检查与源码分析，任务在产出前被中断。原因不是信息不足而是信息不可用：交接缺少"已完成什么、下一步做什么、接口如何约定"的任务状态摘要，子智能体只能靠重新探索恢复全局。这提示多智能体系统的交接协议应当以状态增量（进展、决策、约束）为单位，而不是原始调查的堆叠——与 [[concepts/context-management-agent-systems]] 的结论一致。

### 三类有效机制的共同充分条件：主 Agent 的整合能力

互补工作、并行探索、独立验证三种机制表面不同，共享同一个结构：多样化生成 + 中央整合。justext 任务里 8 个子智能体分头交付、主 Agent 实现核心算法，734s → 468s 的加速来自分工可正确集成；Halite 重建里真正带来收益的不是重复生成两份代码，而是主 Agent 比较、选择并把两套实现的有效部分合并（292/391 vs 顺序版构建失败）；密码学子智能体的价值则在于补上主 Agent 缺失的关键证据（ECDSA nonce reuse 下 P-256 的阶数约束），使算法得以修正。这也解释了为什么前述失败大多表现为"整合失败"：没有整合能力的并发，只是把 Token 换成并行调查。这种多路探索 + 主节点裁决的结构与 [[concepts/orchestrator-worker-architecture]] 描述的模式吻合。

## 核心判断与工程含义

论文把问题从「How many agents can work in parallel?」改写为「How can agents work effectively together?」——并发能力不是创建子智能体的 API，而是任务组织、执行治理、上下文交接与共享状态管理的系统能力。对 Harness 设计的直接含义有三条：(1) 短周期、改动集中型任务默认串行更优，动态并发应作为长周期任务的显式策略而非全局默认；(2) 子智能体必须有明确的 owner 与产出契约，否则退化为伪并发；(3) 共享工作区需要文件所有权与临时环境隔离机制，这是当前失败占比最高（33.2%）的一类。^[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026.md]

## 关联

- 并发协作的机制与边界：[[concepts/multi-agent-systems]]、[[concepts/multi-agent-context-isolation]]、[[concepts/subagent-spawning-pattern]]
- Harness 视角：[[concepts/harness-engineering]]、[[concepts/coding-agent-architecture]]、[[concepts/agent-orchestration-patterns]]
- 具体机制实例：[[entities/claude-code-dynamic-workflows-multi-agent-orchestration]]、[[entities/agent-orchestration]]
- 评测设计：[[concepts/agent-evaluation-benchmark-frameworks]]、[[concepts/evaluation-harness-design]]

→ [[raw/articles/coding-agent-dynamic-concurrency-failure-modes-2026|原文存档]]
