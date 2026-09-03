---
title: "harness 长程任务模式"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, harness, long-running, agent, persistence, resume]
sources: [entities/harness-engineering-long-term-agent-tasks, entities/agent-harness-architecture]
---

## 定义

harness 长程任务模式解决 agent 跑「数小时到数天」任务的工程问题：状态持久化、中断恢复、上下文不溢出、人工介入点、失败回滚。这是从「chat agent」迈向「agent worker」的关键工程能力。

## 核心范式

- **状态外存**：把 working memory 序列化到 SQLite / Redis，进程崩溃可恢复
- **checkpoint 机制**：每个里程碑保存可恢复快照，失败回滚到上一个 checkpoint
- **人工 gate**：高风险动作前必须人工 approve（删数据/外网调用/支付）
- **resource budgeting**：token / time / cost 三维 budget，超限自动停止上报

## 背景与提出

长程任务（long-running task）是 agent 系统工程化后必然遇到的挑战：当 agent 需要执行超过单次 context window 能容纳的操作序列时，如何保证状态不丢失、目标不偏移、中间错误可恢复。^[entities/harness-engineering-long-term-agent-tasks] 这个问题在传统软件工程中叫「持久化 + 事务」，但在 agent 系统中它有了新的维度——agent 的决策链本身是有状态的，而 LLM 的输出是概率性的，导致「重试」不是简单的 redo，而是一次全新的推理。

## 范式细节

长程任务的核心工程挑战分三层。状态层：agent 需要在跨 turn、跨 session、跨重启之间保持工作状态。文件系统是最常见的选择（Claude Code 用文件 IO 做主状态），但也可以用数据库、向量存储或 git commit。进度层：长程任务需要 checkpoint 机制——每完成一个子目标就记录进度，中断后从最近 checkpoint 恢复而非从零开始。Hermes Agent 的 cronjob 天然支持这种模式（每次 tick 读上次进度 → 继续执行）。目标层：长时间运行后 agent 可能「忘」了原始目标——上下文被压缩、摘要丢失关键信息、中间探索改变了方向。解决方案是在 system prompt 中硬编码不可修改的「任务契约」，即使上下文被裁剪也不会丢失。^[entities/agent-harness-architecture]

## 局限与反对声音

长程任务最大的局限是「成本不可预测」。一个预期 10 分钟的任务可能因为幻觉、重试、探索而跑 2 小时——token 消耗是开环的。第二个问题是「观察者效应」：你在监控长程任务的同时，监控本身也在消耗 context window。第三个问题是「compaction 导致目标漂移」：autoCompact 生成的摘要可能丢失关键约束，agent 从摘要恢复后跑偏但自己不知道。目前没有通用解决方案——只能靠人类 gate 在关键节点做检查。

## 现实案例

Claude Code 的 Auto Mode 在大型 PR 上就是长程任务的典型场景：遍历数十个文件、理解跨模块依赖、逐个修改、跑测试、自修错——整个过程可能持续 30+ 分钟、消耗 200K+ token。它的 checkpoint 机制是 git commit：每完成一个文件修改就 auto-commit，即使中断也能 git reset 回最近的安全点。Hermes Agent 的 cronjob 是另一种长程模式：每次 tick 读取上次进度（从文件/cron-report）、执行增量工作、写回进度、等下一个 tick——把一个长任务拆成多个短 tick，每个 tick 都是幂等的。^[entities/harness-engineering-long-term-agent-tasks]

## 现实案例

长程任务的三个生产级实现案例。Hermes Agent cronjob：状态存 [canonical wiki 路径已隐藏]，每个 tick 读上次进度、续做、save、下次 tick 继续。这是 cron-based 长程任务的标准模式——任务被拆成多次 tick，每次 tick 不超过 5 分钟，整体跨小时/天运行。^[entities/hermes-agent] Claude Code 的 --resume 模式：用户在对话中途退出，再次打开 Claude Code 时 --resume 标志可以恢复之前的 conversation history + working state。这是 session-level 长程任务的实现——跨重启但仍在同一天内。真正的「跨天长程任务」Claude Code 目前不直接支持，需要用户自己用 git commit 做 checkpoint。^[entities/claude-code-core-internals] OpenAI Operator（浏览器 agent）：单次任务最长 30 分钟，超时后状态保存到云端，用户可以「继续」任务。跨设备 + 跨时区的长程任务在 2026 年开始出现，但还不是主流能力——主要是经济模型和 trust model 都不成熟。^[entities/agent-harness-architecture]

## 实践启示

长程任务模式在实践中需要 4 个工程决策。状态外存选型：文件 IO（Claude Code 风格）简单但难查询，SQLite（Hermes cronjob 风格）支持事务和索引，向量存储（MemGPT 风格）适合语义检索但延迟高。Checkpoint 粒度：每完成一个子目标就 checkpoint（细粒度，恢复成本低但 IO 多）vs 每完成 25% / 50% / 75% / 100% checkpoint（粗粒度，IO 少但恢复损失大）。建议从「每个里程碑」checkpoint 起步，根据实际恢复频率调整。人工 gate 设计：高风险动作必须 human-in-the-loop——删数据、外网调用、支付、对外发送。gate 不能事后审计（事后发现问题已造成损失），必须是 pre-action approve。Cost budget：每次任务启动时设定 token + time + cost 三维上限，超限自动 pause + 通知用户。Hermes cronjob 用 --max-runtime + token-count checkpoint 实现 budget 控制。失败重试策略：可重试错误（网络超时）自动重试，不可重试错误（schema 错误）立即 fail + 通知用户，避免在错误方向上重复消耗资源。

## 与相邻概念的区分

和「serverless long-running function」（AWS Step Functions / Azure Durable Functions）的区别：这些是 Software 1.0 的长程任务实现（执行确定的代码流程），harness long-running task 是 Software 3.0 的实现（执行概率性的 agent loop）。前者的 checkpoint 是状态序列化，后者的 checkpoint 是「上下文 + 工作状态 + 目标契约」的完整快照。和「Cron job」的区别：cron 是定时触发（每 N 分钟/小时），长程任务是事件触发（完成一个子目标后立即触发下一步）。cron 适合周期性维护任务（每天清理日志），长程任务适合状态推进任务（研究完成后立即开始写作）。Hermes Agent 把两者融合——cron 触发后执行长程任务的「下一个 tick」，cron + long-running 是同一个抽象的不同视角。和「Workflow engine」（Airflow / Temporal）的区别：Workflow engine 执行 DAG（有向无环图），长程任务可能执行非 DAG 结构（agent 自主决定下一步）。Workflow 适合已知流程的自动化，长程任务适合未知探索的执行——agent 可能走回头路、分叉、合并，这些在 DAG 中难以表达。和「Persistent chat session」（Claude.ai 长对话）的区别：长对话是用户持续和 AI 交互，长程任务是 AI 自主执行多步操作。用户的角色从「每 turn 输入」变成「偶尔 gate」，这是产品形态的根本转变。

## 在 wiki 中的关联

- [[entities/harness-engineering-long-term-agent-tasks|长程任务 harness]]
- [[concepts/long-running-agent-architecture|长程 agent 架构]]
- [[concepts/harness-loop-architecture|harness 主循环]]
- [[concepts/agent-memory-substrate-three-layer|memory substrate 三层]]
- [[concepts/production-agent-engineering|production agent engineering]]

## 进一步阅读

- [[entities/harness-engineering-long-term-agent-tasks]]
- [[entities/agent-harness-architecture]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
- [[moc/layer-5-production-security|Layer 5 Production Security]]
