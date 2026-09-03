---
title: AI Agent 探索之路：从 Task-Driven 到 Goal-Driven
created: "2026-05-10"
updated: 2026-08-29
type: concept
tags: [agent, task-driven, goal-driven, sdd, observability, control-plane, mcp, a2a]
sources: [raw/articles/ai-agent-exploration-path-legacy-tech]
---
# AI Agent 探索之路：从 Task-Driven 到 Goal-Driven
## 核心观点
> 人是瓶颈。但解决瓶颈的方式不是让 AI 替代人，而是让系统不再依赖人的实时在场。
> 作者是一个有十年经验的老开发，从手动管理 4-6 个 AI 终端的困境出发，逐步构建出可以 24 小时无人值守运行的 AI Agent 系统。整个探索过程经历了多个阶段：意识到瓶颈 → 80% 需求用 Bash 解决 → Vibe Coding 翻车 → SDD + 调度层 → 自举能力 → Goal-Driven 演进。
## 关键原则
### 决策层级：目标 → 代码 → CLI → Prompt → Agent
每往上一层，不确定性增加一个量级，成本也增加一个量级。**能在下层解决的，绝不上推**。
| 层级 | 适用场景 | 不确定性 |
|------|----------|----------|
| 目标层 | 想清楚到底要解决什么 | 最低 |
| 代码层 | 确定性逻辑 | 低 |
| CLI 层 | 组合现有工具 | 中低 |
| Prompt 层 | 需要语义理解和判断 | 中高 |
| Agent 层 | 多步推理、动态决策、循环执行 | 最高 |
**能用 10 行 Bash 解决的，别折腾 AI。** 这不是反 AI，是尊重工程。
### Vibe Coding 是先易后难，SDD 是先难后易
- Day 1-3：产出速度惊人，成就感爆棚
- Day 7：代码开始乱，陷入"打地鼠"
- Day 14：被迫打开每个文件浏览，大量过度设计
- Day 15：整整一天"设计与实现对齐"
> 捷径的尽头是弯路，大道的尽头是自由。
### 脚手架 > 模型
投入回报对比（个人经验估算）：
- 模型升级：成本 +300%，效果 +20%
- 脚手架升级：成本 +50%，效果 +200%
**优先投资脚手架，而不是追最新最贵的模型。**
## 24h 打工人系统架构
### 核心：文件 + 轮询
调度层做四件事：
1. **接收任务**：用户反馈进来，写入文件队列
2. **分发执行**：轮询队列，调用 CLI 执行
3. **状态管理**：记录每一步的输入输出，持久化到文件
4. **失败切换**：某个 CLI 配额用完，自动换下一个
### SDD（Spec-Driven Development）
每个需求处理完留下一组文档：
- `spec.md`：目标、验收标准
- `plan.md`：技术方案、涉及文件、实现步骤
- `tasks.md`：任务清单，每个任务有描述和状态
> **留痕不是为了 debug，而是为了进化。**
### 智能并发策略
| 策略 | 做法 | 理由 |
|------|------|------|
| 组间并发 | 前端任务和后端任务同时跑 | 代码在不同目录，不会冲突 |
| 组内串行 | 同一个前端项目的任务排队执行 | 可能修改同一文件，避免冲突 |
| 失败隔离 | 单个任务失败不影响其他组 | 故障不扩散 |
### 工具失败自动切换
配合 Tool Prober 定时探测工具可用性：
- 正常：task → codex（可用）→ 执行成功
- 失败切换：task → codex（配额耗尽）→ gemini-cli → 执行成功
- 全部不可用：task → 等待 → 5 分钟后自动探活
## Agent 自举
Agent 自己修了自己的 bug 的前提：
1. **清晰的设计文档** —— AI 知道每个模块该做什么、不该做什么
2. **SDD 流程** —— spec → plan → tasks 的标准路径
3. **constitution.md** —— 架构约束文件，定义代码组织规范、命名规则、模块边界
> 自举的前提是 constitution.md（架构约束文件）。不需要写得多长，但至少要覆盖三件事：目录结构约定、模块边界、命名规则。
## 从 Task-Driven 到 Goal-Driven
| 维度 | Task-Driven | Goal-Driven |
|------|-------------|-------------|
| 人的角色 | 项目经理 + 执行监督 | 目标设定者 / 审核者 |
| Agent 的角色 | 执行器 | 自主推进者 |
| 决策中心 | 在人脑子里 | 在目标 + 边界 + 系统状态里 |
| 主要成本 | 人持续编排 | 前期建模和约束设计 |
| 适用场景 | 简单、一次性任务 | 长期、复杂、持续推进任务 |
**Task-Driven 解决执行问题，Goal-Driven 解决迭代问题。**
### Goal-Driven 的 5 个前提
1. **目标必须清晰** —— 不是模糊愿望，而是可推进、可判断的目标表达
2. **边界必须清晰** —— 哪些能做，哪些不能做，资源上限是什么
3. **状态必须可见** —— 当前做到哪一步，卡在哪，为什么卡
4. **过程必须留痕** —— 否则无法知道成功或失败的原因
5. **权限必须可控** —— 它到底能调用哪些工具，能写到哪里
> Goal-Driven 不是更放权，而是更强约束下的有限自治。
### STATE.yaml 共享状态
用共享状态来协调多 Agent 协作，避免主 Agent 上下文越来越重的瓶颈。
## 生产级 Agent 系统地基（5 层）
1. **目标表达**：到底想完成什么
2. **能力单元**：有哪些 Skill/工具/工作流
3. **运行时状态**：当前正在做什么
4. **治理边界**：允许做什么，不允许做什么
5. **评估反馈**：哪些行为值得固化，哪些必须修正
少任何一层，系统都可能看起来能跑，但跑不稳。
## 协议层趋势
| 时间 | 事件 | 意义 |
|------|------|------|
| 2025-03-11 | OpenAI 发布 Agents 新基座 | Responses API + Tools + SDK + Tracing，runtime 开始收敛 |
| 2025-04-09 | Google 发布 A2A | Agent 间协作有了协议化趋势 |
| 2025-06-23 | A2A 捐给 Linux Foundation | 从企业项目变行业标准 |
| 2026-02-13 | GitHub 发布 Agentic Workflows | Agent 进入 CI/PR/Issue 主流程 |
| 2026-03-16 | Microsoft Foundry Agent Service GA | 企业级 eval + 全链路 tracing |
> **Agent 开发正在从"框架之争"，转向"协议 + runtime + control plane 之争"。**
## 落地路径（6 步）
| 步骤 | 做什么 | 核心产出 |
|------|--------|----------|
| 第一步 | 写清楚 spec | 要做什么、不做什么、怎么算完成 |
| 第二步 | 执行过程留痕 | Prompt/状态/输出/错误全记录 |
| 第三步 | 补 observability 和 eval | 知道为什么成功、为什么失败 |
| 第四步 | 高频动作沉淀为 Skill | 模板 + 规则 + 代码 |
| 第五步 | 引入调度和并发 | 调度层 + 轮询 + 失败切换 |
| 第六步 | 最后才尝试 Goal-Driven | 目标表达 + 治理边界 + 共享状态 |
**先让一次执行可复盘，再让它可重复，再让它可规模化，最后让它可有限自主。**
## Agent 记忆与持久化范式
探索之路走到 24h 无人值守阶段时，核心问题从"Agent 能否完成任务"变为"Agent 如何记住和延续"。这实际上是 AI Agent 持久化记忆（Persistent Memory）范式的核心命题——如何让 Agent 的状态不随会话结束而消失。

当前业界存在三种主要记忆方案，各有不同的适用场景：

**RAG（显式记忆）**：基于向量检索的外部记忆系统，将信息编码为向量存储在 FAISS 等向量库中，按需检索。在细粒度推理任务（小说 QA）上表现最强，延迟约 2249ms，容量理论上无限。缺点是检索质量依赖 Embedding 质量，跨领域泛化能力有限。

**MSA（隐式记忆）**：基于 KV Cache 分级缓存的方案，将历史上下文压缩进模型权重层。HotpotQA 多跳检索场景表现优于 RAG（4.172 vs 3.815），但细粒度小说 QA 弱于 RAG（1.574 vs 2.152）。核心问题是 mean pooling 压缩会稀释细粒度词序和转折信息，论文消融实验显示禁用原始文本注入后性能下降 37.1%。

**Doc-to-lora（参数记忆）**：将记忆编码为 LoRA 权重，容量约 8K tokens。实验结果显示该方案全面失败——gold 答案字符串出现率仅 32%（RAG 为 76%），且信息越多表现越差，本质上是压缩有损与幻觉的组合代价超过其收益。

对于 24h 打工人系统而言，最实用的方案是**文件系统持久化 + 按需加载**的混合策略： ^[raw/articles/prompt-context-harness-three-evolutions.md]
- **工作目录即记忆**：Git Log + progress.md + STATE.yaml 构成 Agent 的外部海马体
- **启动序列即回忆**：每次启动时自动执行"检查工作目录 → 读取 Git Log + progress.md → 定位最高优先级未完成任务 → 开始工作"的标准流程
- **索引层保持精简**：巨型上下文压缩至百行索引目录，动态加载子文档，避免上下文污染 ^[raw/articles/prompt-context-harness-three-evolutions.md]

这与 SDD 的"留痕"原则高度一致——留痕不仅是 debug 手段，更是在为 Agent 构建外部记忆系统。当 Agent 因配额耗尽或会话中断而重启时，文件系统中的进度文档就是它的"记忆恢复"机制。
## 企业 Agent 规模化：从个人工具到 24h 运营体系
个人开发者用 AI 编程，与企业级 AI Agent 系统之间隔着一道深沟。前者关注单次任务完成率，后者关注的是**系统的持续可用性、治理合规性和成本可控性**。 ^[raw/articles/agent-从能用到管好中间差了什么.md]

企业 Agent 规模化面临四个结构性挑战： ^[raw/articles/agent-从能用到管好中间差了什么.md]

**抽象层级错位**：传统 IAM（RAM）控制云产品 API 权限，但企业需要的是业务语义层的权限管理。例如"市场部可以使用营销文案生成 Agent"，而非"市场部可以调用 token 数为 X 的 API"。Agent 平台需要三层多租户体系（UserGroup → User → UserSpace）将底层云资源管理封装为符合人类直觉的业务接口。 ^[raw/articles/agent-从能用到管好中间差了什么.md]

**隔离粒度粗糙**：共用实例导致数据泄露与资源争抢。组级资源配额、用户级权限叠加、空间级数据完全隔离，从根本上消除"裸奔"状态——这与个人开发者的并发策略（组间并发，组内串行）逻辑相似，但规模和严格程度完全不同。 ^[raw/articles/agent-从能用到管好中间差了什么.md]

**协作链路断裂**：业务开发者依赖基础设施团队提需求、等排期。平台提供开发者控制台自助服务，同时通过资源审批流平衡敏捷性与规范性——这对应着从 Task-Driven 到 Goal-Driven 的角色转变：基础设施团队从"执行者"变为"平台维护者"，业务开发者从"等待者"变为"目标设定者"。 ^[raw/articles/agent-从能用到管好中间差了什么.md]

**成本黑盒与审计缺失**：缺乏实时配额监控和业务上下文关联的日志。精细化配额管理（六档规格）+ OpenTelemetry 可观测大盘实现事前预警、事后审计。量化来看，合理配额限制和资源回收机制可降低 30%-50% 非必要 Token 消耗。 ^[raw/articles/agent-从能用到管好中间差了什么.md]

对于从个人 24h 系统演进到企业平台，最关键的不是技术选型，而是**认知转变**：从"让 AI 完成任务"到"让 Agent 系统持续可靠运行"。质量是需要成本的——OpenAI 3-7 人团队 5 个月近百万行代码的实践表明，6 小时深度 Harness（F-Harness） vs 20 分钟快速执行，20 倍时间差异带来的是质的飞跃的可靠性。 ^[raw/articles/prompt-context-harness-three-evolutions.md]

Karpathy 在 2026 年的访谈中提出一个关键论断：**可验证性决定自动化上限**。没有验证体系托底，Agentic Engineering 顶多算更高级的 Vibe Coding。个人 24h 系统的下一步演进方向，正是从"能跑"到"可验证"——这需要引入 Evaluator Agent（独立于 Generator 的验证者）来模拟人类工程团队的利益博弈结构。 ^[raw/articles/karpathy-vibe-coding-to-agentic-engineering.md]
## 相关资源
- [[raw/articles/ai-agent-exploration-path-legacy-tech|原文存档]]
- [[concepts/sdd-specification-driven-development-harness|SDD（规格驱动开发）]]
---
> **真正的跃迁，不是让 AI 多做几个步骤，而是让人退出微观调度。增强自我，而非取代自我。**
## 相关实体
- [[entities/anthropic-官方生产级-agent-最佳实践12-个可复用的-mcp-设计模式-v2|Anthropic 官方生产级 Agent 最佳实践：12 个可复用的 MCP 设计模式]]
- [[entities/yumanju-ai-full-flow-efficiency|柚漫剧 AI 全流程提效拆解]]
- [[entities/claude-code-skills-mcp-rules-source-analysis|Claude Code 源码解析：Skills/MCP/Rules 底层机制对比]]
- [[entities/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2|AI tool poisoning exposes a major flaw in enterprise agent security]]
- [[entities/claude-code-mcp-server|Claude Code MCP Server]]
- [[entities/aws-devops-agent-实战云网络故障自主调查与修复建议|AWS DevOps Agent 实战：云网络故障自主调查与修复建议]]
- [[entities/十年老技术开发的-ai-agent-探索之路-v2|十年老技术开发的 AI Agent 探索之路 v2]]
- [[entities/harness-engineering-three-evolutions|Harness Engineering：AI工程的三次进化]]
- [[entities/context-engineering-three-memory-paradigms|上下文工程：三种 Agent Memory 方案对比实验]]
- [[entities/karpathy-vibe-coding-to-agentic-engineering|Karpathy 最新访谈：从 Vibe Coding 到 Agentic Engineering]]
- [[entities/agent-从能用到管好中间差了什么|Agent 从能用到管好中间差了什么]]

## 关联实体

**上游依赖**:
- [[entities/anthropic-官方生产级-agent-最佳实践12-个可复用的-mcp-设计模式-v2]] — 提供基础理论/方法
- [[entities/yumanju-ai-full-flow-efficiency]] — 提供基础理论/方法
- [[entities/claude-code-skills-mcp-rules-source-analysis]] — 提供基础理论/方法

**下游应用**:
- [[entities/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2]] — 具体应用场景
- [[entities/claude-code-mcp-server]] — 具体应用场景
- [[entities/aws-devops-agent-实战云网络故障自主调查与修复建议]] — 具体应用场景

**平行协作**:
- [[entities/harness-engineering-three-evolutions]] — 替代/补充方案
- [[entities/context-engineering-three-memory-paradigms]] — 替代/补充方案
- [[entities/karpathy-vibe-coding-to-agentic-engineering]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-0-foundation|Layer 0 Foundation]]
