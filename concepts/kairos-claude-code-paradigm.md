---
title: KAIROS — Claude Code 常驻协作范式
created: 2026-05-07
updated: 2026-08-29
type: concept
tags: [claude-code, agent, paradigm-shift, claude, persistent-agent, memory, kairos]
related:
  - [[entities/claude-code-architecture|Claude Code 架构解析]]
  - [[entities/agent-memory-architecture|Agent Memory 架构]]
  - [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]]
sources: ['raw/articles/claude-code-kairos-paradigm-2026']
review_value: 8
review_confidence: 9
review_recommendation: strong
review_stars: 5
confidence: high
---
# KAIROS — Claude Code 常驻协作范式
## 概述
KAIROS 是 Claude Code 的下一个 AI 形态：把运行范式从「终端里的同步问答器」转变为「长期在线、异步协作、跨渠道接入、能自己维持工作节奏的常驻代理」。代码中 KAIROS 出现 154 次（变量前缀合计 365 次）。
名称来源：KAIROS (καιρός)，古希腊语，意为「正确的、关键的或合宜的时刻」——定性的、超越时序的「时机」。
## 核心范式转变
| 维度 | 普通 CLI | KAIROS |
|------|----------|--------|
| 存在方式 | 用户在终端前才工作 | 持续在线，异步协作 |
| 会话 | 每轮独立，进程结束价值终止 | 进程重启后接回原会话 |
| 外部事件 | 接不住 | 可通过 channel 推送触发 |
| 记忆 | 主题文件 + 索引 | Daily Log + 事件流优先 |
| 输出 | 适配终端阅读 | 适配异步消费（Brief 压缩） |
| 动作 | 读/写/执行/输出 | 等待/监听/回传/唤醒/跨渠道 |
## 已实现的子系统
### KAIROS_BRIEF
异步场景输出压缩层。在后台任务长时间运行、外部事件触发、移动端通知等场景下，用最低认知成本传递状态。不是文案优化，是工程问题。
### KAIROS_CHANNELS
外部消息通过 channel notification 进入会话，包装成结构化消息处理。Claude Code 不再只属于本地终端，用户可在终端外继续与同一工作流互动。
### KAIROS_PUSH_NOTIFICATION / KAIROS_GITHUB_WEBHOOKS
外部事件订阅能力。订阅 PR 状态、代码变更等，反向驱动内部执行。
### KAIROS_DREAM
记忆蒸馏路径。整合 daily logs 和 session transcript 提取长期记忆。
### 工具层
- **SleepTool**：时间维度资源管理，等待/唤醒/周期/空闲/值班
- **SendUserFileTool**：文件回传
- **PushNotificationTool**：异步通知
- **SubscribePRTool**：PR 订阅
## Bridge — 会话连续性的基石
Bridge 数据流：远端入口收到消息 → 通过 bridge 拉取工作 → 创建或恢复 REPL → 执行 → 结果回传。
关键：`useReplBridge`，assistant 模式下启用 **perpetual bridge session**，让远端看到的是同一条持续会话，而不是每次 CLI 启动都开一条新的 session。
没有 perpetual session：用户面对的是多段相似但断裂的对话，每次恢复都像重新认识一次项目。
有了 perpetual session：用户开始把它当成「同一个一直在线的协作者」。
## Daily Log 记忆系统
普通模式（主题文件 + 索引）：
- 新信息立即整理成 topic files
- MEMORY.md 维护索引
- 适合短周期会话
KAIROS 模式（Daily Log + 事件流优先）：
- 白天 append-only 记录到当日日志
- 不急着重组和提炼
- 后续蒸馏成长期 memory
**KAIROS 还想把 session transcript 也纳入记忆蒸馏**：长期记忆来源从「当前轮总结」扩展到「完整工作轨迹」——谁在什么时候发来过什么消息、某项任务等待了多久、哪个验证步骤失败过、某个方向为什么被放弃。
## 尚未闭环的 stub（成熟度瓶颈）
| 组件 | 状态 | 影响 |
|------|------|------|
| isAssistantMode() | 返回 false | 主程序无法按 assistant 模式切换 |
| isKairosEnabled() | 返回 false | gate 未接，产品级放行缺失 |
| session discovery | 返回空数组 | 常驻会话接续链路断裂 |
| proactive 状态机 | stub | autonomous work 行为协议无执行层 |
| assistant 专属 prompt | 空 | 上下文初始化不完整 |
## 五大产品价值
1. **留存提升**：常驻会话 + 跨重启续接 + 长期记忆 → 用户把它当「长期协作体」
2. **任务完成度提升**：后台执行 + sleep 唤醒 + 外部事件驱动 → 补上高价值长任务缺口
3. **渠道扩大**：channels + push + bridge + webhook → 从 Developer Tool 往 Agent Platform 滑
4. **粘性增强**：上下文越深，替换成本越高
5. **产品天花板抬高**：竞争对象从 coding assistant CLI → 开发团队操作层代理
## 四大代价
1. **系统复杂度暴涨**：长生命周期会话、幂等外部事件接入、唤醒节奏控制、跨时间跨状态排障
2. **成本模型变差**：tick + sleep 本质上是用更多 API 调用换持续在线行为
3. **安全门槛更高**：trusted directory + KAIROS gate，企业采购必须正面回答「它什么时候能自己执行」
4. **产品承诺与实现未对齐**：主入口闭环未打穿，外围越多系统越像「展台很大，地基不稳」
## 当前成熟度评估
| 维度 | 评估 |
|------|------|
| 产品意图 | ✅ 非常清楚，方向一致，不是拼凑 |
| 框架布线 | ✅ 工具层/提示词层/bridge层/memory prompt分叉/_channel notification 均已铺 |
| 关键外围实现 | ✅ Bridge perpetual session、频道接入、Brief 规则、daily-log memory prompt 有骨架 |
| 主入口闭环 | ❌ assistant 主模块、gate、session discovery、proactive 状态仍是 stub |
定位：**接近产品化的战略子系统**，不是完整封版功能包。
## 关键洞察
> KAIROS 真正改变的，是 Claude Code 的「存在方式」。普通 Claude Code 的存在方式是：用户打开终端，它出现；终端关闭，这轮交互的主价值结束。KAIROS 的存在方式是：用户不在场时，它仍然能持有状态、接收事件、维持节奏、积累记忆、回传结果。
> 这才是 KAIROS 值得持续投入的原因——它决定的不是某个 feature，而是产品类别：从 Developer Tool 推向责任承接者。
## KAIROS 与 OpenClaw / AgentCore 的架构对比

KAIROS 的产品定位介于 OpenClaw 的工具框架和 AgentCore 的托管平台之间，理解这种中间态有助于看清 KAIROS 的战略价值与风险。

[[concepts/openclaw-architecture|OpenClaw]] 解决的是"如何在单一 Agent 框架内高效编排工具和子 Agent"——它的核心抽象是 Tool 系统、MessageBus 和 SubagentManager，所有设计都围绕"让主 REPL 循环能精准控制子 Agent 的执行"这一目标。苏雄在代码解析中指出的"薄抽象 + 显式控制流 + 贴近模型 API = 工程确定性"，OpenClaw 是一个工具开发框架，不是平台。^[raw/articles/openclaw-architecture-800lines.md]

 解决的是"如何托管多个 Agent 的运行时环境"——它的核心抽象是 Harness（模型之外的一切：编排逻辑、执行环境、工具连接、状态管理、身份认证、可观测性），AWS 通过 AgentCore 把 Harness 工程变成了可托管的云服务。AgentCore 假设 Agent 已经存在，它提供的是**执行舞台**。

KAIROS 处于两者之间：它既不是工具开发框架，也不是托管执行平台，它要解决的是**Claude Code 如何从开发者工具变成团队协作成员**。KAIROS 的核心问题不是"Agent 怎么执行"，而是"Claude Code 的存在方式如何改变"——从"终端里的会话工具"到"跨渠道、跨重启、持续在线的协作者"。这个定位让 KAIROS 与 OpenClaw 和 AgentCore 形成了一个分工：OpenClaw 定义工具如何被 Agent 调用，AgentCore 定义 Agent 如何被平台托管，KAIROS 定义 Agent 如何与人类长期协作。

三者的关键设计差异：

| 维度 | OpenClaw | AgentCore | KAIROS |
|------|----------|-----------|--------|
| 核心抽象 | Tool + Subagent | Harness | Bridge + Channel |
| 设计目标 | 工具调用确定性 | 运行时托管 | 协作连续性 |
| 会话模型 | 单次 REPL 循环 | Session 概念淡 | Perpetual Session |
| 记忆 | 无持久化（REPL 重启即失） | 外部存储挂载 | Daily Log + 蒸馏 |
| 外部接入 | 无 | MCP Server + REST | Channel + Webhook |
| 安全假设 | 开发者本地 | 云端受控环境 | 需要 Trusted Directory |

---

## KAIROS 的工程化挑战与未决问题

KAIROS 的成熟度瓶颈不仅体现在代码层面（5 个 stub），更反映了一个深层矛盾：**产品承诺的协作体验需要工程基础设施的支撑，而这些基础设施本身还未成型**。

**Perpetual Bridge Session 的工程复杂度**。KAIROS 的 Bridge 机制要实现"远端消息接续同一会话"，需要在 Bridge 侧维护 session 状态。这带来了一个分布式系统经典问题：Claude Code 进程重启后，Bridge 如何知道上一个 session 的状态在哪里？当前代码中 `session discovery` 返回空数组，说明这链路的持久化方案尚未打通。在生产环境中，这会直接导致"每次 Claude Code 重启，用户感觉像换了一个新的协作者"——这与 KAIROS 的核心价值承诺完全矛盾。

**Proactive 状态机的执行层缺失**。KAIROS 描述了一个 autonomous work 行为协议：Agent 应该能主动发现问题、主动发起行动、主动汇报进度。但这套协议需要有执行层来承载——不是 prompt 里写"你应该主动"，而是需要有状态机、任务队列和调度器来确保"主动"真的发生。当前 `proactive 状态机` 是 stub，意味着 KAIROS 在很长一段时间内仍会是"响应式工具"而非"主动式协作者"。

**信任边界与安全模型的未对齐**。KAIROS 的常驻特性意味着 Claude Code 在用户不在场时持有状态、持有上下文、甚至可能持有执行权限。 的设计明确区分了"工具调用"（低风险）和"自主执行"（高风险），并通过 Trusted Directory 和 KAIROS gate 来控制边界。但 KAIROS 代码中的 `isKairosEnabled()` 返回 false 说明这些安全边界尚未建立。当一个常驻 Agent 同时持有状态和执行能力时，"它什么时候可以自己行动"是一个必须正面回答的产品问题，而不是可以留到后续版本的安全隐患。

**上下文积累与上下文污染的临界点**。KAIROS 的记忆系统承诺"谁在什么时候发来过什么消息、某项任务等待了多久、哪个验证步骤失败过"——这些是真正的长期记忆。但随着 session 时间拉长，Claude Code 的上下文窗口会面临与[[concepts/openclaw-architecture|OpenClaw]]工具注册表相同的问题：信息越多，检索越慢，注意力越分散。当 Daily Log 积累到一定规模后，KAIROS 需要回答"哪些记忆应该被激活，哪些应该被归档"——这个问题目前没有工程实现，只有概念框架。

---

## 相关概念
- [[concepts/ahe-agentic-harness-engineering|AHE — Agentic Harness Engineering]] — KAIROS 的后台执行、记忆系统与 AHE 的 Harness 可观测性有相通之处
- [[entities/claude-code-architecture|Claude Code 架构解析]] — Claude Code 的底层架构基础
- [[entities/agent-memory-architecture|Agent Memory 架构]] — Daily Log 记忆方案与 Agent 记忆架构的关联
- [[concepts/openclaw-architecture|OpenClaw 架构]] — 工作流中枢定位与 OpenClaw 的记忆/协作设计可对照
- [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2|刚刚Opus 4.7发布，相比4.6核心变化，与Claude Code搭配最佳实践]]
- [[entities/boris-cherny-ide-to-agent-console|Boris Cherny — 从 IDE 到 Agent 控制台]]
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 提示词体系源码解析]]
- [[entities/读完-claude-code-和-openclaw-的-memory-源码我对agent记忆需要向量数据库这件事产生了怀疑|Claude Code vs OpenClaw 记忆系统 — 向量数据库必要性反思]]
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]]
- [[entities/claude-code-kairos-paradigm-2026|Claude Code KAIROS 范式 2026]] — KAIROS 仓库代码的深度解析，分析其记忆系统重构、Brief 机制、 Bridge 基础设施等
- [[concepts/karpathy-llm-wiki-v2|Karpathy LLM Wiki V2]]
- [[entities/llm-wiki-obsidian-wiki-gbrain-self-organization-self-evolution|深度解析LLM Wiki / Obsidian-Wiki / GBrain：Agent时代知识的"自组织"与"自进化"]]
- [[entities/hermes-agent-self-evolving-source-analysis|hermes-agent-self-evolving-source-analysis]]

## Related Concepts
- [[entities/agent-engineering-principles-architecture-practice|Agent 原理、架构与工程实践]]

- [[entities/ai-agent-engineer-capability-map|AI Agent 工程师能力地图]]

## 新增关联实体
- [[entities/claude-code-engineering-truth-1.6-98.4]]
- [[entities/garry-tan-complexity-ratchet-90percent-testing-20260513]]
- [[entities/gstack-garry-tan-600k-lines-60-days]]
- [[entities/karpathy-llm-wiki-second-brain-awkthole]]
- [[entities/markdown-ai-era-ifanr-20260513]]

## 关联实体

**上游依赖**:
- [[entities/claude-code-architecture]] — 提供基础理论/方法
- [[entities/agent-memory-architecture]] — 提供基础理论/方法
- [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2]] — 提供基础理论/方法

**下游应用**:
- [[entities/刚刚opus-47发布相比46核心变化与claude-code搭配最佳实践-v2]] — 具体应用场景
- [[entities/boris-cherny-ide-to-agent-console]] — 具体应用场景
- [[entities/claude-code-prompt-source-analysis]] — 具体应用场景

**平行协作**:
- [[entities/hermes-agent-self-evolving-source-analysis]] — 替代/补充方案
- [[entities/agent-engineering-principles-architecture-practice]] — 替代/补充方案
- [[entities/ai-agent-engineer-capability-map]] — 替代/补充方案

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
