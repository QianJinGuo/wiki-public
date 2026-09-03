---
title: "多智能体团队协作"
created: 2026-06-12
updated: 2026-08-29
type: concept
tags: [concept, multi-agent, team, coordination, openclaw, collaboration]
sources: [entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏, entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2, concepts/openclaw-architecture]
---

## 定义

多智能体团队协作是把单个 agent 的工作拆给一组专业化 subagent，每个负责一类任务，由 orchestrator 统一调度。本质是用「角色专业化 + context 隔离」换取「单 agent 能力 × N」，但要付出 token 成本 N+ 和编排复杂度的代价。

## 核心范式

- **角色分工**：planner / coder / reviewer / tester 各负其责，prompt 高度专业化
- **context 隔离**：每个 subagent 只看自己任务相关 context，避免污染
- **orchestrator 派活**：根据任务类型分发给合适的 subagent，收集结果聚合
- **协作代价**：token 用量倍增、并发竞争、subagent 结果合并冲突

### 背景与提出

多智能体团队协作的思想源头可以追溯到 1990 年代的「分布式人工智能」（Distributed AI）研究，尤其是 Anita Woolf 提出的「多智能体系统」框架。但彼时的 agent 是规则驱动的符号系统，协作逻辑靠手工编写，无法扩展到复杂任务。进入 LLM 时代后，LLM 本身具备了理解自然语言指令、进行复杂推理的能力，使得「让 agent 调用 agent」变得可行——这就是 2023-2024 年间爆发式增长的多智能体协作框架（AutoGen、ChatDev、OpenClaw 等）的技术背景。

OpenClaw（龙虾）框架是 [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|完全指南]] 中详细记录的一种多智能体实现，由独立开发者社区维护，特点是极度轻量和可定制。它的核心洞察是：单个 LLM 的能力有上限（context window、推理深度、专业领域覆盖），但通过「角色分工 + 独立 context + orchestrator 统一调度」，可以把多个专业化 LLM 的能力叠加，突破单 agent 的能力天花板 ^[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]。

这个方向被广泛采用的根本原因是 AI 编程助手场景的刚需：一个生产级代码任务需要同时理解需求（researcher）、写代码（coder）、检查质量（reviewer）、写测试（tester）——让同一个 LLM 同时扮演四个角色，要么 prompt 过于复杂难以稳定执行，要么角色之间相互干扰。专业化拆解后，每个角色的 prompt 简单、输出稳定、容易评测。

### 范式细节

**角色分工**的实现质量直接决定整个系统的上限。[[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw]] 的实践建议是：每个角色对应一个独立的 agent instance，有自己的 system prompt（通常 200-500 字）、自己的 tool set（只给 coder 提供 file writing tool，只给 reviewer 提供 read tool）、以及自己的输出 schema（结构化 JSON 或 markdown，便于下游解析）。一个反面案例是「全知型 orchestrator」——所有角色共享一个全功能 agent instance，结果是角色边界模糊，orchestrator 倾向于自己完成所有任务而不是分派出去 ^[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]。

**context 隔离**在技术实现上依赖进程级分离：每个 subagent 运行在独立的进程/容器中，拥有独立的 context window，主 agent 无法直接读取 subagent 的中间推理过程。这带来了一个工程权衡——subagent 的思考过程对主 agent 不透明，如果 subagent 推理方向错误，主 agent 要到拿到结果后才发现，时已经浪费了 token 和时间。改进方向是让 subagent 输出「推理摘要」而非「最终答案」，主 agent 通过摘要判断是否继续或重试。

**orchestrator 派活**的核心能力是任务拆解。拆解的质量取决于 orchestrator 对子任务边界的理解——好的拆解产生独立性强、依赖关系少的子任务（并行度高），差的拆解产生高度依赖的子任务（orchestrator 变成串行瓶颈）。AutoGen 框架的论文数据显示，任务拆解质量对整体吞吐量的影响可达 3-5 倍。实践中的经验法则是：每个子任务应该「能被一个 agent 在一次 context window 内完成」，如果任务需要跨越多次 context 交互才能完成，说明拆解粒度不够细。

**协作代价**中最容易被低估的是「结果合并冲突」。当 N 个 subagent 并行返回结果时，它们对「任务边界的理解」可能存在重叠或缝隙——reviewer 说「这段代码需要重写」，但 coder 已经写了新版本，两个结果合并时产生矛盾。Orchestrator 需要有冲突解决能力，但这个能力本身也是一个 LLM 调用，增加了延迟和 token 消耗。实践中，schema 约束（每个角色的输出格式严格规定，减少歧义）和 pre-mortem（subagent 开始前，orchestrator 明确告知「你负责 X，不负责 Y」）是减少合并冲突的主要手段。

### 局限与反对声音

多智能体协作最大的批评是「token 成本抵消了能力收益」。五个 agent 并行运行，五个 system prompt、五份 tool 描述、五个 context window，各自的 LLM 调用成本叠加——一个原本可以单 agent 完成的小任务，用多 agent 架构后 token 消耗增加 5-10 倍。在 token 成本敏感的场景，这个 overhead 可能是不可接受的。

第二个系统性问题是「信任边界不清导致的循环依赖」。A agent 需要 B 的输出才能继续，B 需要 C 的输出，C 又需要 A 的某个中间结果——orchestrator 如果没有提前识别并打破这个循环，整个系统就会死锁。分布式多智能体系统中，循环依赖是比结果冲突更难解决的 bug，因为它不报错，只是永远不会输出。

第三个批评集中在「专业化的角色定义本身就是脆弱的」。当任务落在角色定义的空白地带时，没有 agent 负责处理。[[concepts/agent-role-specialization|agent 角色专业化]]的进阶模式（动态角色创建）试图解决这个问题，但引入了新的复杂性：orchestrator 需要决定何时创建新角色、新角色的 prompt 怎么写、谁来验证新角色的输出质量。

### 现实案例

[[concepts/openclaw-architecture|OpenClaw 架构]] 是多智能体团队协作最典型的参考实现。它的 team 由一个 orchestrator（主控）和三个 worker（researcher/coder/reviewer）组成，通过共享的「tool 消息总线」通信。[[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw 完全指南]] 里记录了一个完整案例：自动化代码审查任务，先由 researcher 用 @web 工具收集相关技术文档，coder 读取后写实现，reviewer 对照 researcher 的文档检查实现质量，orchestrator 收集三方结果后生成最终报告。这个 pipeline 在中等复杂度任务上相比单 agent 方案，错误率降低约 40%，但 token 成本增加 6 倍——是典型的「质量换成本」权衡 ^[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]。

multi-agent orchestration 层的另一个成熟案例是 ChatDev 的流水线：需求分析 → 代码撰写 → 代码审查 → 测试编写 → 打包发布，五个环节各一个 agent，串行执行。虽然 token 成本高，但任务的线性依赖关系天然避免了并发冲突，是多智能体架构入门的最佳实践场景。

## 在 wiki 中的关联

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏|OpenClaw 完全指南]]
- [[concepts/openclaw-architecture|OpenClaw 架构]]
- multi-agent orchestration
- [[concepts/subagent-spawning-pattern|subagent spawning 模式]]
- [[concepts/orchestrator-worker-architecture|orchestrator-worker 架构]]

## 进一步阅读

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]]
- [[concepts/openclaw-architecture]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
