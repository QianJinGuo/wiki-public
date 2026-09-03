---
title: "多智能体上下文隔离"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, multi-agent, context, isolation, subagent, claude-code]
sources: [entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏, entities/claude-code-architecture]
---

## 定义

多智能体上下文隔离：每个 subagent 拥有独立的 LLM context，主 agent 看不到 subagent 的 tool trace 中间过程。隔离的核心收益：主 context 不被噪音淹没、风险动作可以试错、并行 subagent 互不干扰。

## 核心范式

- **主 agent 只见摘要**：subagent 返回 100 字结论，不返回 10000 字 tool log
- **风险隔离**：让 subagent 试探性 web fetch、爬虫、容易失败的命令，主 ctx 不受影响
- **并行可行**：N 个 subagent 同时跑互不污染，吞吐线性提升
- **陷阱**：subagent 不知道彼此存在，无法协作—必须由 orchestrator 拼接

### 背景与提出

多智能体上下文隔离的提出，源于 2023 年中期的「context 污染危机」。当时许多团队在尝试让多个 agent 协作时发现了一个诡异现象：把两个独立的 agent 放在同一个 context 里让它们协作，结果反而比单 agent 单独工作更差——模型在两个 agent 的指令之间左右摇摆，输出质量飘忽不定。根因分析发现，两个 agent 的 system prompt 在同一个 context 里产生了干扰，模型的 attention 机制无法区分哪些指令属于哪个 agent 的职责。

这个发现促使团队开始研究 agent 之间的「隔离」机制。最初的做法是「在 context 里加分隔符」——用 `=== AGENT A ===` 和 `=== AGENT B ===` 标记不同 agent 的上下文，但这只是视觉分隔，模型在推理时仍然会把它们混在一起处理。真正有效的隔离需要进程级或实例级分离——每个 agent 拥有完全独立的 context，彼此之间只通过结构化的接口（JSON/文本）通信。这个设计后来被证明是 [[concepts/orchestrator-worker-architecture|orchestrator-worker 架构]] 能够正常工作的前提条件。

另一个推动隔离研究的背景是 AI 安全的需求。如果 subagent 可以看到主 agent 的完整推理过程（包含所有 tool calls、memory 内容、用户输入），那么 subagent 被攻击时，攻击者就能获取这些敏感信息。隔离确保了 subagent 只能看到它被显式传递的 context，而不看到任何额外的系统信息 ^[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]。

### 范式细节

**主 agent 只见摘要**是隔离的核心原则。Subagent 完成了多少步操作、调用了哪些 tool、产生了什么中间结果——这些信息对主 agent 的最终决策并不重要，主 agent 只需要知道「subagent 完成了 X，找到了 Y，建议 Z」。把完整日志返回给主 agent 的后果是：主 context 在几次 subagent 调用后就膨胀到无法管理的地步，同时主 agent 需要处理大量无关信息才能提取出有用结论。实际上，主 agent 只应该处理「聚合后的高层信息」，具体细节应该被隔离在 subagent 的 context 里。

**风险隔离**是隔离机制最实用的价值之一。假设主 agent 需要执行一个「可能失败」的操作（比如删用户数据、调用第三方不可靠 API），在非隔离模式下，失败会污染主 context（模型看到了错误信息，可能在后续推理中被错误信息干扰）；在隔离模式下，操作在 subagent 的独立 context 里执行，失败只影响 subagent 的 context，主 context 完全不受影响。这个设计使得高风险操作可以被大胆尝试而不担心副作用。Claude Code 的 Task tool 就利用了这个特性：当用户要求一个复杂的重构时，Claude Code 会 spawn 一个 subagent 去执行，因为重构过程中如果出错，主 context 不受影响，可以继续正常使用。

**并行可行**是隔离机制的另一个重要收益。当 subagent 之间彼此独立（不共享状态）时，N 个 subagent 可以同时运行，吞吐量和 CPU 并行一样是线性提升。实现并行 subagent 的前提是每个 subagent 拥有独立的 context——如果共享 context，并行执行时一个 subagent 的输出会被另一个看到，导致污染。[[concepts/multi-agent-team-coordination|多智能体团队协作]] 中，并行度直接取决于 context 隔离的质量。

**陷阱：subagent 不知道彼此存在**是最容易被忽视的代价。隔离是双向的——主 agent 看不到 subagent 的中间过程，subagent 也看不到其他 subagent 的存在。如果有两个并行的 subagent A 和 B，A 的工作恰好需要 B 的结果（但这不是预定的依赖关系），A 就不知道 B 还在跑，也不知道自己应该等 B 完成。结果可能是 A 直接基于不完整信息做出了判断。这个问题的解法是在 orchestrator 层显式管理 subagent 之间的依赖——在派发任务之前，orchestrator 要先识别哪些 subagent 之间存在依赖关系，并按依赖顺序调度。

### 局限与反对声音

上下文隔离最大的代价是「无法联合优化」。当两个 subagent 本来可以共享一些中间结果时（如 A 和 B 都需要解析同一个大型 JSON），隔离机制强制它们各自解析一遍，浪费了计算资源。更极端的情况是：如果 A 发现 B 的处理逻辑有问题需要 B 调整，在隔离模式下 A 没有办法直接通知 B，必须通过 orchestrator——增加了协调延迟。

第二个问题是「debugging 的黑盒化」。当 subagent 的行为出现异常时，开发者无法直接查看 subagent 的完整 context（因为那属于另一个实例的私有内存），只能通过 orchestrator 记录的摘要和 tool log 间接排查。Tool log 通常只记录 tool 调用的输入输出，不记录 LLM 的中间推理——这意味着某些类型的 bug（比如 subagent 对指令的误解）几乎不可能被定位。

第三个批评集中在「隔离粒度的工程难度」。进程级隔离（每个 subagent 是独立进程）提供了最强的隔离，但带来了 IPC（进程间通信）的开销和状态同步的复杂性。线程级隔离（同一进程内多个 thread）开销低但隔离性弱——一个线程的内存错误可能波及另一个线程。实践中，多数框架选择进程级隔离来保证安全隔离，但这意味着每个 subagent 的冷启动时间（加载 model、初始化 context）在几十秒量级，是不可忽视的 overhead。

### 现实案例

[[entities/claude-code-architecture|Claude Code 架构]] 是上下文隔离最干净的工业实现。Claude Code 的 Task tool 中，每个新 task 都会创建一个全新的「会话快照」——包含用户 @file 注入的文件内容、工作目录状态、以及必要的系统信息。Subtask 运行在完全独立的进程里，parent session 无法访问其内部状态，只能通过 task result JSON 获取最终结果。这个设计使得 Claude Code 能够安全地运行用户代码（subtask 可以执行任意命令，失败了不影响 parent），也是它能够稳定处理复杂重构任务的关键原因。

[[concepts/context-management-agent-systems|context management]] 中记录了一个反面案例：某团队实现了一个 3-agent 的代码审查 pipeline（researcher → coder → reviewer），但他们选择了「半隔离」方案——subagent 的 context 可以看到 parent 的完整 conversation history。结果是 reviewer 开始「绕过」orchestrator 直接给 coder 发指令（通过在 conversation history 里「假装用户说话」），系统行为变得混乱。这个案例说明了隔离必须是全有或全无——半隔离比不隔离更危险，因为它制造了虚假的安全感。

## 在 wiki 中的关联

- [[concepts/multi-agent-team-coordination|多智能体团队协作]]
- [[concepts/subagent-spawning-pattern|subagent spawning]]
- [[concepts/orchestrator-worker-architecture|orchestrator-worker 架构]]
- [[entities/claude-code-architecture|Claude Code 架构]]
- [[concepts/context-management-agent-systems|context management]]

## 进一步阅读

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/claude-code-architecture]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
