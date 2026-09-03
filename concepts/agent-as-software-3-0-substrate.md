---
title: "Agent 作为 Software 3.0 基础设施"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, agent, software-3-0, substrate, infrastructure, karpathy]
sources: [entities/karpathy-vibe-coding-agentic-engineering, entities/agent-harness-architecture, entities/hermes-agent]
---

## 定义

Agent 作为 Software 3.0 基础设施：当 LLM 是新一代运行时（Karpathy Software 3.0），agent harness 就是这个运行时的「操作系统 + 标准库」——决定应用怎么调度资源、访问工具、维持状态。我们正在搭 3.0 时代的 Linux/POSIX，标准还没固化。

## 核心范式

- **LLM = 计算单元**：相当于 CPU，但是概率性、上下文敏感、token 计费
- **harness = OS**：调度、资源管理、I/O 抽象、状态持久化
- **tool catalog = 标准库**：file / network / database / domain 工具的标准接口
- **agent app = 用户态程序**：调用 harness 提供的接口，专注业务逻辑

## 背景与提出

如果 Software 3.0 的「代码」是自然语言 prompt，那 Agent 就是 3.0 的运行时（runtime）——类似于 JVM 是 Java 的运行时、浏览器是 JavaScript 的运行时。^[entities/karpathy-vibe-coding-agentic-engineering] Agent 作为 substrate（基础设施层）的观点意味着：未来的软件不是「在 OS 上运行」，而是「在 Agent 上运行」——Agent 成为人和操作系统之间的新抽象层。

## 范式细节

Agent 作为 substrate 有 4 个核心属性。自然语言接口：用户用自然语言描述意图，agent 编译为系统调用。这和 shell 的区别是：shell 需要精确命令语法，agent 接受模糊意图并自行推断。工具调用抽象：agent 把文件 IO、网络请求、数据库操作等封装为 tool calls，用户不需要知道底层细节。这和 API 的区别是：agent 可以动态选择用哪个工具、组合多个工具、在工具失败时自动换方案。状态持久化：agent 维护跨操作的工作状态——用户不需要手动管理中间文件。这和传统脚本的区别是：状态是语义级的（「我在做什么」）而非语法级的（「哪些文件被修改了」）。可观测执行：每一步 agent 行为都有 trace/日志，用户可以审计。这是 substrate 的信任基础——如果 agent 做了什么你不知道，substrate 就退化成了黑箱。^[entities/agent-harness-architecture]

## 局限与反对声音

Agent-as-substrate 最大的问题是非确定性：同样的输入可能产生不同的输出（LLM 的概率性），这和传统 runtime 的「相同输入 = 相同输出」根本不同。操作系统可以保证 ls 永远返回文件列表，但 agent 不能保证「列出文件」永远用 ls 而不是 find 或 glob。第二个问题是延迟：LLM 推理延迟（秒级）远高于传统 runtime（微秒级），不适合实时交互场景。第三个问题是成本：每次「系统调用」都要花 token，而传统 OS 调用几乎免费。这三个问题决定了 agent substrate 不会替代 OS，而是在 OS 之上增加一层——就像浏览器没有替代 OS 但成为了新的应用平台。

## 现实案例

Hermes Agent 在某些使用模式下已经接近 substrate：用户通过自然语言指挥 Hermes 操作文件系统（读/写/搜索）、运行终端命令、浏览网页、发消息——所有操作系统级别的操作都被 agent 封装了。但 Hermes 也展示了 substrate 的边界：当用户需要精确控制时（git rebase、数据库迁移），还是要退回传统命令行——因为 substrate 的非确定性在高精度操作中是负担而非便利。^[entities/hermes-agent]

## 现实案例

Agent 作为 Software 3.0 substrate 的三个产品级证据。Hermes Agent 是 substrate 的完整实现：LLM 是计算单元、cronjob + skill 是 OS、terminal/browser/file 是标准库、用户的 prompt 是「用户态程序」。整个 Hermes 的工程目标就是「让 LLM 像 CPU 一样可被编程」。^[entities/hermes-agent] Claude Code 的 Auto Mode 走得更远：用户自然语言描述需求，agent 自动选择工具调用、跨多文件编辑、运行测试、提交 commit——这是把自然语言作为系统调用入口的实证。^[entities/claude-code-core-internals] AutoGPT / BabyAGI 在 2023 年的早期尝试证明了 substrate 思路的可行性但暴露了缺陷——没有 verifier gate、没有人介入点、没有成本控制，导致 agent 在 substrate 上无限循环烧 token。这些失败案例反过来定义了「substrate 必须有什么」：预算限制、人工 gate、checkpoint 机制。三件套在 2026 年的 Agent 框架中已是默认设计。^[entities/agent-harness-architecture]

## 实践启示

把 agent 当作 substrate 用，工程团队需要重新设计三层组织。运行时层（agent harness）：投入类似 OS 内核团队规模的工程资源——核心 loop、状态管理、错误恢复、调度。这是 substrate 的「不可替代层」，必须自建或深度定制。Hermes Agent 选择自建但用最小实现（约 1000 行核心 + 插件扩展）。标准库层（tool catalog）：投入类似 POSIX / Win32 API 团队的规模——文件、网络、数据库、领域工具的统一接口。每个 tool 必须有清晰的 schema、错误处理、性能预算。Hermes Agent 的 tool 设计哲学是「LLM 友好」——tool description 写得详细、参数 schema 标准化、错误信息人类可读，这是 substrate-friendly 的关键。应用层（agent apps）：业务团队基于 substrate 写 prompt + spec + skill，专注业务逻辑。Hermes 的 skill 体系就是应用层的标准——任何业务能力都可以封装为 skill，被任何 agent 加载。组织上需要「substrate team」横跨所有应用，确保 API 演进不破坏下游——这是软件工程里 platform team 的角色在 agent 时代的对应。

## 与相邻概念的区分

和「agent 是 AI 应用」的区别：「AI 应用」关注具体业务（客服 agent / 编程 agent / 搜索 agent），substrate 关注底层基础设施（agent 怎么调度、怎么执行、怎么持久化）。应用层创新靠业务洞察，substrate 层创新靠系统工程。和「WebAssembly / Container / VM」的区别：这些是 Software 1.0/2.0 的 runtime（执行字节码 / Linux 进程 / 中间代码），agent 是 Software 3.0 的 runtime（执行自然语言 prompt）。关键区别：传统 runtime 的输入是确定性代码（相同输入产生相同输出），agent runtime 的输入是概率性 prompt（相同输入可能产生不同输出）。这个概率性让 agent substrate 的设计哲学完全不同——不能像 JVM 那样追求「一次编写到处运行」，必须接受「一次编写到处运行但结果可能不同」。和「LLM Framework」（LangChain / LlamaIndex）的区别：Framework 是工具集（让开发者更快写 agent 应用），substrate 是运行时（让 agent 应用能跑起来）。Framework 选择属于应用层决策，substrate 选择属于基础设施决策——更基础、更难改、影响面更大。

## Substrate 的未来 1-2 年

Substrate 抽象层在 2026-2027 年大概率会出现标准化——类似 POSIX 之于 OS。可能的标准化方向：tool calling 接口标准化（OpenAI API 兼容已成事实标准），memory 接口标准化（Letta / MemGPT 风格），prompt 格式标准化（Hermes / Anthropic prompt cache），verification 接口标准化（LangSmith / Helicone 风格）。标准化后会出现「substrate 兼容认证」——和 W3C / POSIX 认证类似，证明某个 agent 框架符合 substrate 标准。substrate 标准化的赢家是「最早定义 API 的人」——这个窗口期现在仍然开放。Hermes Agent 在 cronjob / skill / memory 三个维度的早期投入就是为了卡位未来的 substrate 标准。substrate 标准化的输家是「闭源专有 API」——如果某个 agent framework 用专有 API 而非标准 API，会被市场边缘化（类似当年不用 POSIX 的 Unix 厂商）。

## 在 wiki 中的关联

- [[concepts/software-3-0-stack|Software 3.0 stack]]
- [[entities/karpathy-vibe-coding-agentic-engineering|Karpathy 范式]]
- [[entities/agent-harness-architecture|agent harness architecture]]
- [[entities/hermes-agent|Hermes Agent]]
- [[concepts/harness-engineering-paradigm-shift|harness 范式迁移]]

## 进一步阅读

- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/agent-harness-architecture]]
- [[entities/hermes-agent]]

## 所属 MOC

- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
