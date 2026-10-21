---
title: "MemCurrent: Databricks 生产级 Agent 持久记忆架构"
created: 2026-10-21
updated: 2026-10-21
type: entity
tags: [agent, memory, harness, databricks, evaluation, production]
sources: [raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026]
confidence: 0.8
provenance_state: extracted
---

# MemCurrent: Databricks 生产级 Agent 持久记忆架构

Databricks AI Research 在 2026-10 发布的 MemCurrent，把 agent memory 从"检索增强的花活"拉回到工程权衡的主线上：生产 agent 真正难的不是能不能写出 demo，而是同时守住 **quality（长尾任务答对）** 和 **efficiency（规模化下的成本与延迟）**。作者的前提判断是，多数 memory 技术在两个维度间做零和交换，而记忆之所以值得投入，正因为它是少数能同时改善两侧的手段——但"历史越多越好"是错的，把全部交互塞进上下文既贵又会让 agent 迷失。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

## 三类记忆接入方式与它们的代价

文章先把"怎么接记忆"拆成三种模式，这个分类本身比具体实现更有迁移价值。**plugin** 模式在会话的固定点自动读写，更新即时、行为可预测，但运行时代价与延迟高，且在不需要记忆时也照样烧 token；**tool** 模式把读写决策交给 agent，灵活但质量受 agent 自身能力约束；**offline pipeline** 模式在后台按计划或触发器更新记忆，延迟到达但运行时成本最低（异步批量）。三者的取舍表实际上是在回答一个设计问题：**你要把记忆的"决策成本"放在运行时还是放在离线批处理里**。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

MemCurrent 的选择是不选边，而是分层：**episodic memory** 用 plugin 方式 append-only 地记录每个 step（原始历史可回溯、可审计），**derived memory** 则是结构化的"沉淀物"（用户偏好、指标定义、纠正过的错误、成功工作流），由两条通道产出——在线 tool 通道处理显式请求（例如"别在 /tmp 下写脚本"），离线 dreaming 通道从每段新会话里蒸馏隐式经验并清理陈旧条目。两类记忆都落在生产级数据库里，支撑多用户与低延迟读取。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

## 评测：自建 benchmark 与消融

作者没有只跑公开 benchmark，而是指出 Mem0 一类的开源评测多为"模拟人格的开放式闲聊"，与真实工作负载脱节；他们的内部 benchmark 来自真实数据工作区的 12K+ 会话，先挖掘出 3K 条含记忆价值信号的候选，再为每条草拟带评分标准的后续问题。这个构造过程值得记下来：**记忆评测的瓶颈在"构造出必须依赖历史才能答对的问题"，而不在打分模型**。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

结果分两层。在三个开源 benchmark 上，MemCurrent 以约 12 倍更少的输入 token（8.4K vs 103K）达到与 full-context 相当的水平，在历史最长的两个上甚至超过 full context（把全部历史塞进 prompt 反而让 agent 混乱）；内部 benchmark 上则把 pass rate 提升到 3 倍，同时少用 31% token、快 26%，在 pass-rate/token 平面上位于 Pareto 前沿，压过 Mem0、PropMem 与 OpenClaw memory。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

消融给出了最有信息量的数字：无记忆 22.3%，仅 episodic 35.8%，仅在线 tool 通道 53.9%，仅离线 dreaming 63.5%，两者结合最高。也就是说 **derived memory 承担了主要收益，episodic 的价值在于补回蒸馏时被丢掉的细节**；而"离线整合"比"在线即时写"更强，说明记忆质量的瓶颈是 consolidation 的时机与方法，而非读写的即时性。这一结论与 [[concepts/memory-consolidation-decay|记忆整合与衰减]] 的既有判断一致，也直接反驳了"记忆越实时越好"的直觉。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

## 与既有记忆框架的关系

在 wiki 的记忆聚类里，MemCurrent 的位置是"生产化落地层"：[[concepts/agent-memory-architecture|Agent 记忆架构]] 给出分层蓝图，[[concepts/episodic-vs-semantic-memory-agent|episodic vs semantic]] 区分记忆类型，[[concepts/memory-derivation-observability|记忆派生与可观测]] 讨论蒸馏链路，MemCurrent 则用一组可对照的数字把这些抽象层落到"哪一层贡献了多少"的实证上。它同样呼应 [[concepts/working-set-vs-long-term-memory|工作集与长期记忆]] 的核心命题——上下文窗口不是记忆，记忆是窗口之外那部分被验证过、可复用、可清理的知识。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

对做 harness 的人，最可直接借用的两条是：(1) 把记忆拆成"append-only 原始日志 + 可重建的派生层"，让记忆可审计、可回滚（这与 [[entities/agent-memory-architecture|Agent Memory Architecture]] 中的日志化主张同构）；(2) 把 consolidation 放到离线 dreaming 流水线，用批量换成本，同时把"显式指令"留给在线 tool 通道以免延迟。值得注意的是评测口径：作者用 Claude Opus 4.6 作 agent 模型、GPT 5.6 sol 作 memory 模型，并在 LangGraph 与 OpenAI Agents SDK 两套 harness 上取均值，这意味着数字衡量的是"记忆层"而非某个 harness 的偶然表现。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

→ [[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026|原文存档]]
