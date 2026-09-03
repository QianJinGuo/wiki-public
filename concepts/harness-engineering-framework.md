---
title: Harness Engineering 框架
created: 2026-04-27
updated: 2026-08-29
type: concept
tags: [concept, agent, architecture, harness, prompt-engineering, context-engineering]
related:
  - [[entities/thin-harness-fat-skills|Thin Harness Fat Skills]]
  - [[entities/openclaw-prompt-context-harness|OpenClaw Prompt/Context/Harness]]
  - [[entities/claude-code-prompt-context-harness|Claude Code Prompt/Context/Harness]]
  - [[entities/alibaba-skill-up-agent-skill-evaluation|Alibaba Skill-Up CLI]]
sources:
  - raw/articles/harness-engineering-framework
  - raw/articles/peking-university-harness-engineering-taming-agent-2026
confidence: high
---
# Harness Engineering 框架
## 概述
AI 工程领域三次重心的演进：
| 阶段 | 核心问题 | 关键词 |
|------|------|------|
| Prompt Engineering | 怎么说 | 角色/风格/Few-shot/分步/格式约束 |
| Context Engineering | 给什么 | RAG / Skills渐进式披露 / 信息选择与裁剪 |
| Harness Engineering | 别跑偏 | 持续监督/纠偏/验收/约束/失败恢复 |
三层包含关系：**Prompt ⊂ Context ⊂ Harness**。
## 核心方程
> Agent = Model + Harness
> Harness = Agent - Model
**Harness = 除模型外的所有东西**。它决定了模型看到什么、能做什么、按什么规则做、做错了怎么纠偏，以及最后如何把能力稳定地交付出来。
## 六层结构
### 1. 上下文管理
角色与目标定义 / 信息选择与裁剪 / 上下文结构化组织（分层）
### 2. 工具系统
给模型什么工具 / 什么时候调用工具 / 工具结果如何重新喂回模型
### 3. 执行编排
理解目标 → 判断信息是否足够 → 必要时获取外部信息 → 基于结果继续分析 → 生成输出 → 检查输出是否满足要求 → 不满足则修正或重试
### 4. 状态与记忆
当前任务进行到哪一步 / 哪些中间结果应该保留 / 哪些内容应该形成长期记忆
### 5. 评估与观测
输出验收 / 环境验证（实际操作页面/跑交互） / 自动测试 / 过程观测 / 质量归因
### 6. 约束、校验与失败恢复
约束 / 校验 / 恢复（分析原因→重试/切换路径/回退）
## 关键实践
### Anthropic: Context Reset vs Compaction
- **Compaction**：同一Agent，历史变短，心理状态延续
- **Reset**：直接换干净上下文的新Agent，交班时交接清楚
- 对于某些模型（如 Claude Sonnet 4.5），Reset 才能真正"清空包袱、重新出发"
### Anthropic: Generator + Evaluator 分离
- 模型自评偏乐观 → 生产与验收必须分离
- Evaluator 不是读代码打分，而是实际操作页面/跑交互
### OpenAI: 渐进式披露
- AGENTS.md 只有 ~100 行（目录页），详细文档在仓库里
- CI 自动校验文档新鲜度和交叉引用
- "文档园丁" Agent 定期扫描过时文档
### LangChain: Harness 改造效果
- 底层模型完全不变，仅改造 Harness
- Terminal Bench 2.0：52.8 → 66.5（Top30 → Top5）
## 核心洞察
> 真正决定 AI 产品上限的，也许是模型；
> 但真正决定 AI 产品能否落地、能否稳定交付的，往往是 Harness。
> AI 落地的核心挑战，正在从"让模型显得聪明"，转向"让模型在真实世界里稳定工作"。
## Harness Engineering 与上下文分层：信息治理的深层机制

Harness Engineering 之所以能解决"别跑偏"的问题，根本上是因为它建立了一套**信息分层治理机制**——不同类型的信息在不同的生命周期阶段、以不同的策略被处理，而不是一股脑塞进上下文窗口。

上下文分层的必要性源于一个根本矛盾：**Context Rot（上下文腐化）**。当无关内容占据上下文的多数篇幅时，Agent 的决策质量会系统性下降，无论模型有多强大。这不是能力问题，而是信号噪音比问题——关键信息被稀释了。^[raw/articles/harness-engineering-framework.md]

上下文分层体系通常包含五个层次：

**常驻层（Permanent）**：身份定义、核心约束、项目级约定。这层信息的特点是使用频率极高、稳定性极强，每次对话都需要存在，但内容短小精悍。常驻层如果太重，Context Rot 会在第一轮就发生。

**按需加载层（On-demand）**：Skills 和领域知识文档。默认只注入名称和描述（OpenClaw 的渐进式披露实践），任务真正需要时才读取完整内容。这个设计避免了上下文膨胀，同时保证了能力按需可得。

**运行时注入层（Runtime）**：当前任务的目标描述、用户最新反馈、工具调用结果。这层信息只存在于当前任务的生命周期内，任务结束即释放。

**记忆层（Memory）**：跨会话积累的经验、被否决的方案、已确认的结论。当前任务需要参考历史判断时从中检索，用完即回到存储状态而非持续占用上下文。

**系统层（System）**：确定性逻辑、编译器/类型检查器输出、lint 结果。这层不由模型生成，而是由确定性系统产生，是验证模型输出的外部基准。

上下文分层与 Harness 六层结构之间的关系在于：上下文分层是第 1 层（上下文管理）的具体实现方式，而 Harness 的其他五层（工具系统、执行编排、状态与记忆、评估与观测、约束校验与失败恢复）则在这一分层体系之上构建了完整的运行保障。Thin Harness Fat Skills 哲学在这一语境下的含义是：框架层（上下文分层机制）应该薄，而能力层（Skills/知识）的丰富度决定系统的实际智能水平。

## Generator-Evaluator 模式：反馈环的本质与自我评估陷阱

Harness Engineering 中最被低估的设计模式是 **Generator-Evaluator 分离**（Anthropic 实践），它揭示了 AI 系统反馈环设计的核心原理。

**为什么 Generator 和 Evaluator 必须分离？** 模型在生成内容时是 Generator，在评估自己生成的内容时存在系统性偏差——自我评估偏乐观。模型会为自己的输出辩护、解释合理性、放大成功面、淡化失败面。这是 LLM 的固有倾向，而非个别模型的问题。^[raw/articles/harness-engineering-framework.md]

Generator-Evaluator 分离的核心要点：
- **Generator** 负责推进任务：理解目标、调用工具、生成中间结果和最终输出
- **Evaluator** 负责验收结果：不读 Generator 的"心理活动"，直接操作环境验证实际效果
- **反馈环**：Evaluator 的验证结果重新注入 Generator 的下一轮决策

这与软件工程中"写代码的人不该自己测试自己的代码"原则完全对应。单元测试由独立的测试工程师编写，才能发现开发者因"确认偏见"而忽略的边界 case；同样，Agent 的输出由独立的 Evaluator 验证，才能避免 Generator 的自我辩护效应。

**Evaluator 的三种实现形态**：

第一种是**环境验证型**（Anthropic 实践）：Evaluator 实际执行操作而非读取文本。Generator 生成修改文件的代码，Evaluator 用 Playwright 或类似工具实际访问修改后的页面，确认功能正常运行^[raw/articles/harness-engineering-framework.md]。这种方式最接近真实用户场景，但实现成本最高。

第二种是**确定性工具型**：Evaluator 调用编译器、类型检查器、lint 工具、测试套件等确定性系统，机械地验证输出是否符合规范。Martin Fowler 强调的"架构规则交给 linter 和 CI 机械执行"正是这一形态。这种方式的优势在于零主观性，但覆盖范围受限于已有工具的完备性。

第三种是**独立模型型**：用另一个模型（通常同型号但不同实例，或更弱的型号）专门做验证评估。OpenAI 在某些任务中使用了这一模式。优势是实现相对简单，劣势是引入了额外的模型调用成本和两个模型之间的一致性问题。

**反馈环的稳定性条件**：Generator-Evaluator 之间的反馈环要稳定工作，需要满足三个条件：

第一，**验证结果必须可操作**。Evaluator 给出"失败"的判断后，Generator 必须有明确的重试路径或回退方案。模糊的"效果不好"不如具体的"类型检查报错 line 42"有价值。

第二，**环内信息不能腐化**。Generator 和 Evaluator 共享的上下文信息（如任务目标、历史决策）如果出现漂移，反馈环会逐步偏离正确方向。这要求 Harness 提供稳定的状态管理，而非依赖 Generator 的自我记录。

第三，**最大循环次数必须有上限**。反馈不是无限的——如果 Generator-Evaluator 循环 N 次仍未收敛，系统应该主动终止并上报，而不是无限重试。Harness 的第 6 层（约束、校验与失败恢复）提供的就是这一层的保障。

Generator-Evaluator 模式与 Harness 六层结构的对应关系：Generator 主要工作在第 2 层（工具系统）和第 3 层（执行编排），Evaluator 主要工作在第 5 层（评估与观测），两者通过第 4 层（状态与记忆）共享信息，并通过第 6 层（约束、校验与失败恢复）获得兜底保障。

## 相关页面
- [[entities/thin-harness-fat-skills|Thin Harness Fat Skills]] — Harness 哲学的另一种表达
- [[entities/openclaw-prompt-context-harness|OpenClaw Prompt/Context/Harness]] — OpenClaw 的 Harness 实践
- [[entities/claude-code-prompt-context-harness|Claude Code Prompt/Context/Harness]] — Claude Code 的 Harness 实践
- [[entities/agentcore-harness|AgentCore Managed Harness]] — AWS 托管 Harness 平台
- [[entities/skill-design-patterns|Skill 设计模式]] — Skills 是 Context Engineering 的典型实践
## 相关实体
- [[entities/agent-principle-architecture-engineering-practice|你不知道的 Agent 原理架构与工程实践]]
- [[entities/harness-engineering-让-coding-agent-可靠完成长程任务-v2|Harness Engineering: 让 Coding Agent 可靠完成长程任务]]
- [[entities/agent-reliability-context-drift-tool-hallucination|Agent Reliability: Context Drift & Tool Calling Hallucination]]
- [[entities/harness-engineering-long-term-agent-tasks|Harness Engineering：让 Coding Agent 可靠完成长程任务]]
- [[entities/从多智能体编排到ai自主决策资损防控体系的架构演进|从多智能体编排到AI自主决策：资损防控体系的架构演进]]
- [[entities/agent-harness-architecture-deep-dive-aksahy|Agent Harness 解析：智能体架构深度拆解]]
- [[entities/ai-native-时代-研发组织何去何从|AI Native 时代 —— 研发组织何去何从]]
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei|长周期 Agent 详解：从 Ralph Loop 到可接管 Harness]]
- [[queries/harness-peer-review-framework|Harness Design Peer Review Framework]]

- [[entities/agent-era-architect-skills-guide|Agent 时代架构师技能指南]]
- [[entities/design-patterns-for-ai-agents-2026|Design Patterns for AI Agents 2026]]
- [[entities/ai-agent-engineer-capability-map|AI Agent 工程师能力地图]]
- [[entities/martin-fowler-ai-rd-harness-nondeterminism|Martin Fowler AI 研发 Harness：非确定性承重层]]

## 新增关联实体
- [[entities/didi-ibg-customer-experience-llm-quality-inspection-3-pipelines]]
- [[entities/gpt-image-2神级提示词分享]]

## 关联实体

**上游依赖**:
- [[entities/thin-harness-fat-skills]] — 提供基础理论/方法
- [[entities/openclaw-prompt-context-harness]] — 提供基础理论/方法
- [[entities/claude-code-prompt-context-harness]] — 提供基础理论/方法

**下游应用**:
- [[entities/skill-design-patterns]] — 具体应用场景
- [[entities/agent-principle-architecture-engineering-practice]] — 具体应用场景
- [[entities/harness-engineering-让-coding-agent-可靠完成长程任务-v2]] — 具体应用场景

### Supplementary: 北京大学 Harness Engineering 课件（2026-07）
北京大学 AI 肖睿团队（李晴）发布的 82 页 Harness Engineering 培训课件，从 Prompt→Context→Harness→Loop 四阶段系统梳理演进路径，提出 Harness 7 大问题和 ETCLOVG 七层架构。 ^[raw/articles/peking-university-harness-engineering-taming-agent-2026.md]

**平行协作**:
- [[entities/ai-native-时代-研发组织何去何从]] — 替代/补充方案
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei]] — 替代/补充方案
- [[entities/agent-era-architect-skills-guide]] — 替代/补充方案

## 所属 MOC

- [[moc/amazon-aws-ai|Amazon Aws Ai]]
- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
