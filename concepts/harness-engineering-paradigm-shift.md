---
title: Harness Engineering 三次范式跃迁与四根支柱
created: 2026-05-07
updated: 2026-08-29
type: concept
tags: [harness-engineering, context-engineering, prompt-engineering, agentic-coding, anthropic, openai, four-pillars, failure-modes, enterprise]
description: 从 Prompt Engineering → Context Engineering → Harness Engineering 的三次范式跃迁，四根支柱（上下文架构/专业化/持久化记忆/结构化执行），四类失败模式，企业级 AI 代码率从 24.86%→90.54% 的实战方法论。
---
# Harness Engineering 三次范式跃迁与四根支柱
## 核心定位
Harness Engineering 是 2026 年 AI Coding 的核心方法论。从 Prompt Engineering → Context Engineering → Harness Engineering 经历三次范式跃迁，核心从"优化单次交互"演进到"设计跨会话/跨Agent/跨阶段的完整系统架构"。 ^[raw/articles/harness-engineering-90-percent-pillars.md]
## 三次范式跃迁
| 阶段 | 核心关注 | 隐喻 | 代表性工作 |
|------|----------|------|------------|
| **Prompt Engineering**（2022-2024）| 单次交互优化 | 写好一封邮件 | Few-shot, CoT, 角色设定 |
| **Context Engineering**（2025）| 给 Agent 看什么 | 给邮件附上正确附件 | RAG, 动态上下文构建 |
| **Harness Engineering**（2026）| 跨会话/Agent/阶段的系统架构 | 造一辆好车 | 约束/反馈/编排/改进闭环 |
**核心引用**：
- Ryan Lopopolo（OpenAI）：*"Agents aren't hard; the Harness is hard."*
- Mitchell Hashimoto：*"Every time you discover an agent has made a mistake, you take the time to engineer a solution so that it can never make that mistake again."*

**三层不是替代关系，而是叠加关系**——好的 Harness 依赖好的 Context，好的 Context 依赖好的 Prompt。 ^[raw/articles/prompt-context-harness-three-evolutions.md]

**Harness = 工具 + 验证 + 反馈 + 约束**。本质是为 AI 系统构建工程基础设施：验证机制、反馈回路、错误恢复。当 Agent 可以调用工具、访问外部世界、执行长时间任务时，模型的推理能力不再是瓶颈——瓶颈变成了整个系统的可信度。 ^[raw/articles/prompt-context-harness-three-evolutions.md]
## 四根支柱
### 支柱一：上下文架构（Context Architecture）
**原则**：Agent 应恰好获得当前任务所需的上下文——不多不少。 ^[raw/articles/harness-engineering-90-percent-pillars.md]
## 关联实体

**上游依赖**:
- [[entities/一文带你弄懂-ai-圈爆火的新概念harness-engineering-v2]] — 提供基础理论/方法
-  — 提供基础理论/方法
- [[entities/长周期-agent-详解-从-ralph-loop-到可接管-harness]] — 提供基础理论/方法

**下游应用**:
-  — 具体应用场景
- [[entities/claudes_next_enterprise_battle_is_not_mo]] — 具体应用场景
- [[entities/harness-engineering-three-evolutions]] — 具体应用场景

**平行协作**:
- [[entities/ai-native-rd-org-design-xiaobin]] — 替代/补充方案
- [[entities/agent-orchestration]] — 替代/补充方案
- [[entities/context-engineering-three-memory-paradigms]] — 替代/补充方案


→ [[entities/一文带你弄懂-ai-圈爆火的新概念harness-engineering-v2|一文带你弄懂 AI 圈爆火的新概念：Harness Engineering]]
**反面教训**：OpenAI 团队将 AGENTS.md 写成百科全书 → "所有内容都重要 = 没有内容重要"
**正确做法**：AGENTS.md ~100行，作为**索引和地图**，指向深层文档。 ^[raw/articles/prompt-context-harness-three-evolutions.md]

上下文治理的核心是**信息层级设计**：将 context 分为三层——索引层（百行级别，模型始终读取）、子文档层（按需加载，模型读取频率中等）、原始数据层（模型几乎不直接读取，仅在必要时查询）。这与人类工作记忆的"组块"（chunking）原理一致——不是记忆的内容越多越好，而是组织得越好越有用。 ^[raw/articles/prompt-context-harness-three-evolutions.md]
### 支柱二：Agent 专业化（Agent Specialization）
受限工具集的专业 Agent 优于拥有全部权限的通用 Agent。 ^[raw/articles/harness-engineering-90-percent-pillars.md]
→ 
**Anthropic 三角色分离**：
- **Planner** — 规划
- **Generator** — 实现
- **Evaluator** — 验证
> "将做事的 Agent 和评判的 Agent 分开，是一个强有力的杠杆。" ^[raw/articles/prompt-context-harness-three-evolutions.md]

三角色分工的深层逻辑是**模拟人类工程团队的利益博弈结构**：Planner 代表产品利益（功能是否完整）、Generator 代表实现利益（功能是否高效实现）、Evaluator 代表质量利益（实现是否正确）。三者的利益天然不完全一致，这种不一致正是高质量输出的来源。AI 倾向给自己的 Bug 打高分（"自恋问题"）——这是因为 AI 被训练为"让对话继续"，在代码审查场景下，这意味着"让代码被接受"而非"让代码正确"。 ^[raw/articles/prompt-context-harness-three-evolutions.md]
### 支柱三：持久化记忆（Persistent Memory）
进度持久化在文件系统上，而非上下文窗口中。 ^[raw/articles/harness-engineering-90-percent-pillars.md]
→ [[entities/长周期-agent-详解-从-ralph-loop-到可接管-harness|长周期 Agent 详解：从 Ralph Loop 到可接管 Harness]]
**标准化启动序列**：
```
检查工作目录 → 读取 Git Log + progress.md → 定位最高优先级未完成任务 → 开始工作
```
### 支柱四：结构化执行（Structured Execution）
永远不让 Agent 在未经审查和批准书面计划之前写代码。 ^[raw/articles/harness-engineering-90-percent-pillars.md]
→ [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道-v2|深入理解 Claude Code 源码中的 Agent Harness 构建之道]]
**四阶段执行流**：理解 → 规划 → 执行 → 验证（每阶段有质量门禁）
> "Waiting is expensive, fixing is cheap" ^[raw/articles/harness-engineering-90-percent-pillars.md]
## Anthropic 四类失败模式
| 模式 | 描述 | 解法 |
|------|------|------|
| **One-shot Syndrome** | 上下文窗口超过 40% 填充率后质量快速衰退 | Sweet Spot < 40% |
| **Premature Victory Declaration** | 部分完成就宣布结束，核心未验证 | Browser Automation 端到端验证 |
| **Premature Feature Completion** | 功能未做端到端测试，部署后关键路径不通 | Puppeteer MCP 截图验证 |
| **Cold Start Problem** | 新会话需大量 Token 重新理解项目 | progress.md + 持久化记忆 |
**共同根源**：Agent 缺乏外部结构化约束和反馈机制。
**核心缺陷**：*"Agents are incapable of accurately evaluating their own work"*
## 企业级挑战
| 挑战 | 说明 |
|------|------|
| **认知负担** | Agent 知识边界 = 代码库文件边界；隐性知识（高频变更区/配置引用/字段约束）难以捕获 |
| **质量控制缺失** | Agent 产出速度远超人工审查速度；语法正确但语义错误 |
| **熵的累积** | Suboptimal Pattern 累积导致 Code Rot；解法：Golden Principles 编码化 + Entropy Garbage Collection |
**关键转变**：文档从"给人看"变成"给 Agent 看"。发现 Bug = 修 Harness，从根源消除同类问题。 ^[raw/articles/harness-engineering-90-percent-pillars.md]
## 开发者角色范式转移
| 传统模式 | Agent-First 模式 |
|----------|-----------------|
| 写代码 | 设计 Agent 工作环境 |
| 调 Bug | 编写规范文档 |
| Code Review | 任务拆分与验收管理 |
## 新兴趋势：协议层崛起与 Control Plane 之争
2026 年 Agent 领域最值得关注的结构性变化，是**控制平面的崛起与协议标准化浪潮**。过去两年，Agent 开发的核心战场在"框架"——LangChain、CrewAI、AutoGen 等框架争夺开发者心智。但进入 2026 年，战场正在转移： ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

**趋势一：A2A + MCP 双协议合流**
Google 的 A2A（Agent-to-Agent）与 Anthropic 的 MCP（Model Context Protocol）正在从竞争走向互补。A2A 定义 Agent 之间的协作协议（任务分发、状态同步、能力发现），MCP 定义 Agent 与工具/数据源的交互协议。两者组合覆盖了 Agent 系统中最核心的两类连接：Agent↔Agent 和 Agent↔资源。这与 2025 年 Anthropic 推出 MCP、2026 年 A2A 捐给 Linux Foundation 的时间线高度吻合。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

**趋势二：从框架之争到 Runtime 之争**
OpenAI Responses API + SDK + Tracing 的发布，AWS Bedrock AgentCore 的 GA，Microsoft Foundry Agent Service 的 GA，标志着主流云厂商都在推出自己的 Agent Runtime。框架是开发者工具，Runtime 是运维基础设施——谁控制了 Runtime，谁就控制了 Agent 系统的"操作系统"。开发者开始意识到：选框架是临时的，选 Runtime 才是战略决策。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

**趋势三：Observability 基础设施的快速成熟**
GitHub Agentic Workflows 进入 CI/PR/Issue 主流程，Microsoft Foundry Agent Service 提供企业级 eval + 全链路 tracing，说明 Agent observability（可观测性）已从"nice to have"变为"必须具备"。这呼应了 Harness Engineering 的核心洞察：质量是需要成本的，而成本的可视化是质量控制的前提。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

**趋势四：Harness 的"衰变感知"设计**
Harness Engineering 不是静态的——它需要随模型能力变化而演化。Claude 3.0→3.5 升级后，许多硬编码检查规则自然变得不必要。这意味着好的 Harness 组件应该设计为**可插拔的**：当模型能力提升导致某个组件不再必要时，可以低成本移除，而非重构整个系统。 ^[raw/articles/prompt-context-harness-three-evolutions.md]
## 开放问题：尚未解决的核心挑战
尽管 Harness Engineering 在 2026 年取得了显著进展，但仍有几类问题尚未被系统性地解决：

**问题一：Agent 评估（Eval）的根本困境**
当前 Agent eval 主要依赖两种方式：人工标注（成本高、速度慢）和 LLM as Judge（偏见、容易被"讨好"）。真正的挑战在于：**Agent 的行为空间远大于语言模型的输出空间**——一个 Agent 可能花 30 分钟探索代码库后才找到一个关键 bug，这种长时序行为难以用静态 benchmark 评估。F-Harness 的 6 小时深度验证 vs 20 分钟快速检查揭示了一个残酷事实：高质量 eval 的成本是低质量 eval 的 20 倍以上，但两者在基准测试上可能显示相近的分数。 ^[raw/articles/prompt-context-harness-three-evolutions.md]

**问题二：多 Agent 协作的状态一致性**
当多个专业化 Agent（Planner/Generator/Evaluator）并行工作时，它们对"任务完成"的判断可能不一致。Generator 认为代码写完了，Evaluator 认为测试没通过，Planner 认为需求理解有偏差——这种状态不一致如果不能被及时仲裁，整个系统的协作效率会急剧下降。STATE.yaml 和共享状态文件是当前的临时解法，但缺乏像 Git 一样的版本控制和冲突合并机制。 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

**问题三：工具 poisoning 与安全边界**
企业 Agent 系统面临一个尚未被充分重视的威胁：工具 poisoning。Agent 需要调用外部工具（MCP Server、API）来执行任务，但如果工具的输出被恶意篡改或注入，Agent 会将错误信息当作正确答案继续执行。与传统安全边界不同，Agent 的安全边界是动态的——它取决于 Agent 对工具输出的信任程度，而这个信任程度又取决于 Agent 的推理能力（会随模型升级而变化）。 ^[raw/articles/ai-tool-poisoning-exposes-a-major-flaw-in-enterprise-agent-security-v2.md]

**问题四：Harness 的"技术债垃圾回收"**
AI Agent 系统天然产生技术债——因为 AI 的输出不如人类工程师稳定，每次"差不多就行"的实现都会累积为未来的维护负担。OpenAI 的实践是定期后台扫描修复（像垃圾回收一样），但关键问题是：**谁来定义什么是"债"？** 人类的判断标准（代码可读性、架构合理性）与 AI 的判断标准（是否通过测试、是否满足 spec）并不总是一致的。 ^[raw/articles/prompt-context-harness-three-evolutions.md]

**问题五：从"能用"到"管好"的组织能力断层**
企业 Agent 规模化需要的不仅是技术，更是组织能力的重建。传统开发团队的考核指标是"代码产出量"，而 Agent-First 团队需要的考核指标是"系统可靠性"和"自动闭环率"。这种转变要求招聘标准、晋升通道、团队结构都做出调整，但大多数企业尚未建立对应的 Agent 运营流程。 ^[raw/articles/agent-从能用到管好中间差了什么.md]
## 关联
- [[concepts/harness-engineering-framework]] — Harness Engineering 框架完整解析
- [[concepts/coding-harness-engineering]] — Coding Harness 工程本质
- [[concepts/managed-agents-architecture]] — Anthropic Managed Agents 架构
- [[concepts/claude-code-source-leak-lifecycle]] — Claude Code 源码级生命周期解析
- [[concepts/ai-team-knowledge-harness|AI Team 知识沉淀体系]]
- [[entities/claudes_next_enterprise_battle_is_not_mo|Claude's next enterprise battle is not models: it's the agent control plane]]
- [[entities/harness-engineering-three-evolutions|Harness Engineering：AI工程的三次进化]]
- [[entities/harness-engineering-90-percent-pillars|Harness Engineering 四根支柱与四要素架构]]
- [[entities/ai-native-rd-org-design-xiaobin|AI Native 研发组织设计]] — 许晓斌：AI Native 时代组织从 Org Chart 向 Execution Graph 的范式转移
- [[entities/agent-orchestration|Agent orchestration]]
- [[entities/context-engineering-three-memory-paradigms|上下文工程：三种 Agent Memory 方案对比实验]]
- [[entities/民生银行基于规格驱动开发sdd的-codeagent-私域研发探索与实践|民生银行基于规格驱动开发 SDD 的 CodeAgent 私域研发探索与实践]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
