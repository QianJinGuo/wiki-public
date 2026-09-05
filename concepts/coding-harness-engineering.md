---
title: Coding Harness 工程本质
created: 2026-05-07
updated: 2026-09-05
type: concept
tags: [harness, agent, coding, engineering, pi, openclaw]
related:
  - [[entities/langchain-anatomy-agent-harness|LangChain Anatomy of Agent Harness]]
  - [[raw/articles/pi-openclaw-coding-harness|原文存档]]
  - [[entities/agent-principle-architecture-engineering-practice|你不知道的 Agent 原理架构与工程实践]]
sources: ['raw/articles/pi-openclaw-coding-harness']
confidence: high
---
# Coding Harness 工程本质
从 Pi（minimal terminal coding harness）到 OpenClaw，系统阐述 coding agent 的工程分层、五个可复用工程模式、以及 runtime kernel 与 control plane 的分工。
## Pi 定位
**Pi = minimal terminal coding harness**。默认工具集：read、write、edit、bash。再往外才是 session、context files、compaction、skills、extensions、TUI、RPC、SDK。
核心认知：**能力不会从概念里自动长出来，它靠一层层工程边界托住。**
## Pi 分层架构
```
Provider API -> agent loop -> coding tools
-> session / context / compaction
-> terminal UI / RPC / SDK
```
packages 结构：
- `packages/ai`：多 provider LLM API 适配层
- `packages/agent`：通用 agent runtime（tool calling、state、event streaming）
- `packages/coding-agent`：终端 coding harness
- `packages/tui` / `packages/web-ui`：界面层
## 五个可复用工程模式
### 1. Context 是投影，不是有界容器
三类状态分开：
- **给模型看的**：经过治理的 projection
- **给 UI 看的**：用户界面消息
- **给审计/恢复看的**：完整 event log
### 2. Transcript 是账本，working context 是视图
```
durable history: 完整行动轨迹（JSONL，不删除）
working context: 当前模型可见材料
summary: 两者之间的压缩视图
```
只留历史 → 模型被拖垮；只留摘要 → 摘要一错就没证据。**两层都要保留。**
### 3. 权限要进运行时管线
- `beforeToolCall`：参数/路径/权限/风险检查（在命令生成后、执行前拦截）
- `afterToolCall`：审计/截断/错误标记
比 prompt 里写"不要做危险操作"可靠得多。
### 4. Runtime kernel 小，control plane 厚
| | Pi | OpenClaw |
|--|-----|---------|
| 定位 | Runtime kernel | Product control plane |
| session | JSONL transcript | JSONL + sessions.json |
| 工具策略 | 内置静态 | 动态（按 channel/sender 规则计算） |
| 通道 | TUI 本地 | IM/移动/Canvas/cron/webhook |
### 5. 失败路径和证据链一起设计
- read：截断 + 提示续读 offset
- bash：完整输出路径 + timeout + abort
- edit：oldText 不唯一或重叠 → 拒绝执行
## Pi → OpenClaw 的演进关键点
- **session 两层状态**：transcript 记录发生了什么，sessions.json 记录该进入哪条轨道
- **动态工具策略**：按 agent/provider/channel/sender/sandbox 规则计算 effective tool policy
- **context builder 进入运行时**：系统提示词构建纳入 AGENTS.md/SOUL.md/TOOLS.md/IDENTITY.md/USER.md
- **安全边界被通道放大**：本地 CLI → 网络消息入口，权限不再是"加弹窗"
## 搭 Agent 稳定路线
1. 只读 Agent（read、grep、find、ls）→ 2. 精确修改（edit + diff）→ 3. 命令执行（bash + timeout）→ 4. event log → 5. context builder + compaction → 6. skills/extensions/MCP/memory
## 结论
> **弱模型需要 harness 帮它做事，强模型更需要 harness 确保它做事不越界、可复盘、能交付。**
prompt scaffolding 会变薄，runtime harness 会变硬。
## 关联页面
- [[concepts/openclaw-architecture|OpenClaw 架构解析]] — 800行代码实现：Tool抽象/MessageBus/SubagentManager/REPL主循环
- [[entities/langchain-anatomy-agent-harness|LangChain Anatomy of Agent Harness]] — Agent=Model+Harness框架 + 六大组件 + Context Rot 三策略
- [[concepts/harness-engineering-framework|Harness Engineering 框架]] — Prompt/Context/Harness三层 + 七环节控制回路 + Generator/Evaluator
- Pi Agent Runtime — Pi agent runtime 核心源码解析
## 关联实体

**上游依赖**:
- [[entities/langchain-anatomy-agent-harness]] — 提供基础理论/方法
- [[entities/agent-principle-architecture-engineering-practice]] — 提供基础理论/方法
- [[entities/langchain-anatomy-agent-harness]] — 提供基础理论/方法

**下游应用**:
- [[entities/openclaw-comprehensive-guide-32k-chars]] — 具体应用场景
- [[entities/agent-memory-architecture-ruofei]] — 具体应用场景
- [[entities/martin-fowler-ai-rd-harness-nondeterminism]] — 具体应用场景

**平行协作**:
- [[entities/harness-engineering-long-term-agent-tasks]] — 替代/补充方案
- [[entities/loongsuite-genai-semconv]] — 替代/补充方案
- [[entities/hermes-agent-goal-runtime-architecture]] — 替代/补充方案


→ [[raw/articles/pi-openclaw-coding-harness|原文存档]]
## 深度分析
### 1. 从"模型能力"到"运行时秩序"的范式转移
原文最核心的洞察是：**能力不会从概念里自动长出来，它靠一层层工程边界托住。** 这与 Harness Engineering 框架的"Agent = Model + Harness"公式形成深度呼应——Model 决定 AI 有多聪明，Harness 决定 AI 有多可靠。但 Pi → OpenClaw 这条线索进一步揭示：即便是"可靠"本身也分两层——runtime kernel（负责语义）vs control plane（负责产品世界）。当模型能力趋同时，真正产生差异的是外层那套运行时秩序 ^[raw/articles/pi-openclaw-coding-harness.md:22-90]。
### 2. 五工程模式揭示的深层问题：Context 不是容器，是投影
"Context 是投影，不是有界容器"这个表述，直接点出了传统 RAG 和 Agent 记忆系统的共同误区：把 context 当成存放知识的仓库，以为塞得越多 AI 就越"知道得多"。Pi 的做法是：working context 是经过治理的投影，transcript 是不可篡改的账本，两者分离且必须同时存在 ^[raw/articles/pi-openclaw-coding-harness.md:56-73]。这个原则对工程实践的直接影响是：compaction（压缩）不能只是摘要，必须同时保留完整 transcript 作为审计和回滚的证据。
### 3. 权限管线的前后拦截：比 Prompt 约束更可靠的方案
beforeToolCall / afterToolCall 的拦截设计，解决的是一个被广泛忽视的问题：Prompt 约束对 AI 的约束力是软性的（模型可以选择忽略），而运行时拦截是硬性的（在工具真正执行前就可以拒绝或修改）。OpenClaw 在此基础上增加了动态工具策略：按 owner/provider/channel/sender 规则计算 effective tool policy ^[raw/articles/pi-openclaw-coding-harness.md:77-82]。这种"规则计算"比"枚举允许列表"更灵活也更安全，因为它把权限判断从静态配置变成了运行时决策。
### 4. 稳定路线揭示的工程真相：Agent 无法一次建成
六步稳定路线（只读 Agent → 精确修改 → 命令执行 → event log → context builder/compaction → skills/extensions/MCP/memory）的真正含义是：每一步都在修复前一步暴露的 Harness 漏洞 ^[raw/articles/pi-openclaw-coding-harness.md]。只读 Agent 的核心挑战是"工具 schema、上下文组装、结果截断"——这些看似简单的问题在生产环境中会暴露大量边缘 case。跳过步骤的直接后果是：在不稳定的基层上叠加复杂功能，底层问题会以更高频率和更大破坏力在上层爆发。
### 5. "Prompt scaffolding 会变薄，runtime harness 会变硬"的深远含义
这个判断的深层含义是：随着模型能力提升，AI 能够自主完成更多认知任务，但真实环境中的执行约束（文件系统边界、用户身份、权限层级、session 恢复）不会消失——这些约束只能被转移到 Harness 层 ^[raw/articles/pi-openclaw-coding-harness.md]。这意味着 Harness Engineering 不是过渡方案，而是长期系统：模型越强，Harness 越需要硬化，因为模型出错的破坏半径变大了。
## 实践启示
1. **用 Pi 六步路线评估团队当前阶段**：在引入 skills、MCP、memory 等高级功能前，先确认只读 Agent（read、grep、find、ls）是否已稳定。特别关注工具 schema 定义和结果截断——这两个问题会在规模上导致严重的上下文污染 ^[raw/articles/pi-openclaw-coding-harness.md]。
2. **Transcript + Working Context 的双层保留原则**：Compaction 时必须同时保留完整 JSONL transcript 和摘要 entry。验证标准：摘要丢失时能否从 transcript 完整重建当前 session 状态？能，则设计正确；不能，则在 Compaction 前加防丢机制 ^[raw/articles/pi-openclaw-coding-harness.md:63-73]。
3. **beforeToolCall / afterToolCall 是权限设计的最小必要集**：对于高风险工具（bash、write、edit），必须在这两个拦截点实现参数校验、路径限制和审计日志。OpenClaw 的动态工具策略（按 channel/sender 计算 effective policy）是一个可借鉴的权限抽象模式 ^[raw/articles/pi-openclaw-coding-harness.md:77-82]。
4. **Runtime kernel 与 control plane 的分离是长期多通道 Agent 的技术基础**：如果系统需要支持多 channel（本地 CLI、IM、移动端、webhook），必须将 runtime kernel（模型+loop+工具调用）与 control plane（路由+会话管理+通道适配）严格分离。Pi → OpenClaw 的演进路线是这个分离的最佳参考 ^[raw/articles/pi-openclaw-coding-harness.md:83-90]。
5. **失败路径设计先于功能开发**：在实现任何新工具之前，先定义 read 截断策略、bash timeout 和 abort 机制、edit 的 oldText 不唯一处理。证据链和失败路径必须一起设计，否则 Agent 出错时既无法回滚也无法复盘 ^[raw/articles/pi-openclaw-coding-harness.md:93-99]。
## 相关实体
- [[entities/agent-principle-architecture-engineering-practice|你不知道的 Agent 原理架构与工程实践]]
- [[entities/harness-engineering-让-coding-agent-可靠完成长程任务-v2|Harness Engineering: 让 Coding Agent 可靠完成长程任务]]
- [[entities/aiaigc-summit-guest-lineup|AIAIGC峰会嘉宾阵容]]
- [[entities/openclaw-comprehensive-guide-32k-chars|OpenClaw 完全指南：这可能是全网最新最全的系统化教程了！（3.2W字，建议收藏）]]
- [[entities/agent-memory-architecture-ruofei|Agent Memory 架构解析]]
- [[entities/martin-fowler-ai-rd-harness-nondeterminism|Martin Fowler AI 研发 Harness：非确定性承重层]]
- [[entities/hermes-agent-vs-openclaw-comparison|Hermes Agent vs OpenClaw 对比分析]]
- [[entities/autoclaw-使用体验自带-66-个-skill可接入聊天工具安全性高|AutoClaw 使用体验：自带 66 个 Skill、可接入聊天工具、安全性高]]
- [[entities/agent-reliability-context-drift-tool-hallucination|Agent Reliability: Context Drift & Tool Calling Hallucination]]
- [[entities/harness-engineering-long-term-agent-tasks|Harness Engineering：让 Coding Agent 可靠完成长程任务]]
- [[entities/loongsuite-genai-semconv|LoongSuite GenAI 可观测语义规范]]

- [[entities/hermes-agent-goal-runtime-architecture|Hermes Agent /goal 长任务运行时架构]]
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei|长周期 Agent 详解：从 Ralph Loop 到可接管 Harness]]
- [[entities/lowcode-framework-custom-agent-decision-framework-hello-agents|低代码 Agent、框架 Agent、自研 Agent 决策框架]]
- [[queries/harness-peer-review-framework|Harness Design Peer Review Framework]]
- [[entities/three-tools-in-one-gstack-superpowers-openspec-engineering-ai-coding|三器合一：gstack + Superpowers + OpenSpec 工程化 AI 编程实战]]

- [[entities/ai-agent-engineer-capability-map|AI Agent 工程师能力地图]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
