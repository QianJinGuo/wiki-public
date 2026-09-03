---
title: "Orchestrator-Worker 多智能体架构"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, orchestrator, worker, multi-agent, architecture, openclaw]
sources: [entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]
---

## 定义

Orchestrator-Worker 架构是多智能体的主流形态：一个 orchestrator agent 接收用户任务、拆解、分派给若干 worker agent、收集结果、聚合输出。Worker 之间彼此不通信，所有协调通过 orchestrator——简化了状态管理但 orchestrator 成为瓶颈。

## 核心范式

- **集中调度**：orchestrator 是唯一 state holder，worker 无状态
- **任务拆解**：orchestrator 必须能产出可独立完成的子任务列表
- **结果聚合**：worker 返回结构化摘要，orchestrator 做合并 / 冲突解决
- **反模式：worker 互相通信**：会迅速失控，必须强制走 orchestrator

### 背景与提出

Orchestrator-Worker 架构的雏形来自 1990 年代的「管理者-工作者」模式（Manager-Worker Pattern），最初用于分布式计算的任务调度。MPI（Message Passing Interface）在 1990 年代就引入了这个模式：用一個 master 进程管理任务分发，多个 worker 进程并行处理，master 负责收集结果。这个模式在 HPC 领域沿用了三十年，因为它天然适配「可并行、独立性强」的计算任务。

LLM agent 领域引入这个架构，是从 2023 年 AutoGen 框架开始的。AutoGen 的核心抽象是 `AssistantAgent`（可以视为 orchestrator）和 `UserProxyAgent`（可以视为 worker）的组合，前者负责任务规划和结果聚合，后者负责执行具体工具。这个组合后来被证明是 LLM agent 从「单 agent toy demo」走向「production system」的关键一步 ^[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]。

这个架构出现的背景是单 agent 的能力瓶颈：当 agent 需要同时做「规划」「执行」「验证」时，把所有能力塞进一个 prompt 里会导致行为不稳定。拆成 orchestrator + worker 后，orchestrator 专注于「判断下一步做什么」，worker 专注于「执行具体操作」，职责单一化让行为更可预测。代价是增加了系统复杂度和 token 开销，但 production 环境里，可预测性的价值通常超过开销。

### 范式细节

**集中调度**意味着 orchestrator 是唯一的 state holder。Worker 无状态——它不记得自己上一步做了什么，不记得之前返回过什么结果，每次调用都是从零开始（但会收到 orchestrator 传递的上下文）。这个设计极大简化了系统状态管理：不需要在多个 agent 之间同步状态，只需要维护 orchestrator 的状态，然后通过 context 传递给 worker。「全局状态」变成了「通过 context 传递的局部状态」，避免了分布式系统中状态同步的复杂性。

**任务拆解**是 orchestrator 最核心的能力，也最难做好。一个好的任务拆解需要：识别任务中的并行机会（哪些子任务可以同时执行？）、识别任务间的依赖关系（哪些必须串行？）、评估每个子任务的工作量（太细碎则 overhead 高，太粗糙则并行度低）。实践中，orchestrator 的任务拆解能力取决于它的 prompt 工程质量——需要明确告诉它「拆解的原则是什么」「如何判断子任务的边界」。AutoGen 框架提供了默认的任务拆解策略，但在复杂任务上经常需要自定义。

**结果聚合**面临的挑战比想象中大。Worker 返回的可能是不同格式的内容（一个输出了代码，一个输出了文档，一个输出了错误日志），orchestrator 需要把它们合并成一个 coherent 的最终输出。常见的聚合策略包括：串接（concatenation，按执行顺序拼接各 worker 输出）、摘要（每个 worker 输出都做一次摘要，再拼接摘要）、选择（只选某个 worker 的输出作为最终答案）。选择哪种策略取决于任务类型——代码生成任务通常选 coder 的输出，代码审查任务通常综合 reviewer 的意见。

**反模式：worker 互相通信**是一个被反复证明的危险设计。当 worker A 可以直接给 worker B 发消息时，系统会出现「幽灵消息」问题——orchestrator 不知道 A 给 B 发过什么消息，导致它对任务进度的判断完全错误。更严重的是，当 worker 可以互相通信时，orchestrator 就不再是唯一的决策中心，它对任务的控制权被分散到多个节点，系统行为变得不可预测。工程实践中的硬性规则是：worker 之间绝对不允许直接通信，所有信息交换必须经过 orchestrator ^[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]。

### 局限与反对声音

orchestrator 成为单点瓶颈是最明显的系统性风险。当 orchestrator 需要同时管理 10+ 个 worker 时，它的 LLM 调用频率会非常高——每次 worker 返回结果都需要 orchestrator 判断「任务完成了吗？还需要继续分派吗？」如果 orchestrator 的决策速度跟不上，整个系统的吞吐量就被 orchestrator 限制。用更快的 LLM（如 GPT-4o）替代 GPT-4 可以部分缓解这个问题，但无法从根本上解决集中调度的串行化效应。

第二个批评是「orchestrator 的任务拆解能力本身就是瓶颈」。拆解任务需要 deep understanding of the task——对于一个它从未见过的任务类型，orchestrator 很可能无法做出好的拆解。这个问题和 [[concepts/agent-self-improvement-loops|agent 自改进循环]] 相关：orchestrator 需要从历史任务中学习什么类型的拆解效果好，什么类型的导致了返工。这个学习过程本身需要记录和评估机制，增加了系统复杂度。

第三个问题是「context 传递的信息失真」。Worker 的输出经过 orchestrator 摘要后再传给下一个 worker，信息会有损失。如果 coder 输出了一份详细的技术方案，reviewer 只看到了 orchestrator 总结的「方案可行，建议通过」，reviewer 就失去了「方案里有几个地方值得细看」的机会。这个信息失真在长链任务（5+ 步 pipeline）里会累积，导致最终 worker 做出与实际情况不符的决策。

### 现实案例

[[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw]] 的 orchestrator 实现是最经典的参考。它的 orchestrator 有一个固定的「思考模板」：每次接收到 worker 结果后，按顺序问自己三个问题：「任务完成了吗？」「下一个应该调哪个 worker？」「传递给下一个 worker 的 context 应该包含哪些信息？」这个结构化的决策流程比「让 LLM 自由发挥」要稳定得多。在 OpenClaw 的最佳实践里，orchestrator 的 system prompt 通常包含具体的决策规则（如「如果 reviewer 返回『需要重写』，则立即把 coder 的新版本发给 reviewer 重审，不要等到其他 worker 都完成」），这些规则是团队通过多次迭代总结出来的 ^[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]。

multi-agent orchestration 的另一个案例是 ChatDev 的实现：它的 orchestrator 是一个「虚拟 CTO」，负责理解产品需求、拆解开发任务、协调 coder 和 reviewer 的工作。ChatDev 在 GitHub 上开源后的数据分析显示，虚拟 CTO 最常见的失败模式是「任务拆解粒度不当」——把一个需要 50 步的复杂任务拆成 3 个大步骤，导致每个 worker 的 context 过于饱和，处理质量下降。团队后来的改进是增加了「自动拆解审核」机制：拆解方案出来后，再让同一个 LLM 评估「这个拆解方案是否有遗漏？」如果有就打回重解。

## 在 wiki 中的关联

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw orchestrator 实战]]
- multi-agent orchestration
- [[concepts/multi-agent-team-coordination|多智能体团队协作]]
- [[concepts/subagent-spawning-pattern|subagent spawning]]
- [[concepts/agent-orchestration-patterns|agent orchestration 模式]]

## 进一步阅读

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
