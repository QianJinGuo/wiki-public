---
title: "Software 3.0 技术栈"
created: 2026-06-12
updated: 2026-08-01
type: concept
tags: [concept, software-3-0, karpathy, llm, paradigm, stack]
sources: [entities/karpathy-vibe-coding-agentic-engineering, entities/karpathy-llm-wiki-v2-2026]
---

## 定义

Software 3.0 是 Karpathy 给「LLM 即可执行运行时」时代的命名：Software 1.0 是人手写代码、Software 2.0 是数据训练神经网络权重、Software 3.0 是自然语言 prompt 编程一个通用语言模型。三代叠加而非替代。

## 核心范式

- **1.0 = 显式逻辑**：Python/Java 等，确定性、可调试、能 grep
- **2.0 = 隐式权重**：训练管线、数据集、超参，输出是模型 checkpoint
- **3.0 = 自然语言**：prompt、context、工具调用，输出是 LLM 的实时推理结果
- **叠加效应**：现代 agent 系统三层共存，1.0 包裹 3.0 调用 2.0 的模型

## 背景与提出

Karpathy 在 2023 年一篇广为流传的推文中提出了「Software 1.0 → 2.0 → 3.0」的框架。1.0 是人类手写 Python/C++ 的「显式编程」时代。2.0 是训练神经网络权重的「隐式编程」时代——开发者不再写逻辑，而是写训练管线和数据集。3.0 是用自然语言 prompt 编程一个通用模型的时代——开发者不再写代码，而是写指令和上下文。

这个框架提出的背景是 2022-2023 年大模型能力爆发：GPT-3/4、Claude 1/2 相继发布，LLM 第一次可以承担完整的功能性任务（写文章、写代码、做推理），而不仅仅是对已有内容的检索或润色。这让 Karpathy 意识到「程序」的定义正在被扩展——不再只是静态代码文件，而是一个包含指令、状态、工具调用和环境反馈的完整运行时。 ^[entities/karpathy-vibe-coding-agentic-engineering]

Software 3.0 的真正含义在 2026 年被 Karpathy 进一步澄清：上下文窗口即程序。程序不再只是一个代码文件，而是一个包含指令、状态、工具调用和环境反馈的完整上下文。编程的核心问题从「怎么写代码」变成了「哪一段文字应该复制给你的 Agent」。这个转变带来了两个实际后果：第一，程序边界扩大了——一段安装说明、一组测试环境、一个日志文件，都可能成为程序的一部分。第二，context window 成为人操控 LLM 解释器的「把手」，上下文的质量和结构直接影响模型表现。 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]

## 范式细节

三层不是替代是叠加。Software 1.0（~1970s-2010s）：确定性、可调试、能 grep——操作系统、数据库、协议栈至今是 1.0 的天下，不会因为 3.0 的出现而被取代。Software 2.0（~2010s-2020s）：统计性、数据驱动、需要大规模训练——所有需要计算机视觉、NLP、推荐系统的场景都是 2.0 的领地。Software 3.0（~2020s-）：自然语言驱动、上下文敏感、概率性推理——AI 编码 Agent、问答系统、知识管理工作流是 3.0 的首批应用。现代 agent 系统三层共存：1.0 的框架包裹 3.0 的运行时，调用 2.0 的模型。

具体量化来看，一个典型的 Claude Code 会话在 Software 3.0 层面的 token 消耗：system prompt 动态组装约 500-2000 token，工具描述（30+ 工具完整 prompt）约 5000-10000 token，对话历史和工具输出在 200K context window 内动态压缩，LLM 实时推理决定工具调用和代码生成。这个过程里，1.0 代码（终端执行、文件系统操作）和 2.0 模型（Claude Sonnet 推理）共同支撑了 3.0 的 prompt 运行时。 ^[entities/claude-code-core-internals]

MenuGen 案例最能说明 Software 3.0 的本质：旧范式需要 OCR → 图像生成 → 重新排版 → 部署的多步中间件，而 Software 3.0 版本直接把菜单照片喂给 Gemini，让模型输出带菜品图的新菜单——中间结构被模型原生能力吞掉了。Karpathy 的判断是：「我的整个 MenuGen 都是多余的。它还停留在旧范式里。」 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering] 这个案例说明：Software 3.0 的机会不在于「把 10 步压成 3 步」，而在于「以前根本不可能存在的东西」——模型直接覆盖整个任务，中间层失去必要性。

## 局限与反对声音

三层模型在解释「AI 是独立范式」上很有说服力，但这个框架本身不是严格学术的——它是思想实验，不是严格分类法。批评者指出：1.0 和 3.0 的边界很模糊（LLM 本身是 2.0 训练的，但被当作 3.0 运行时），3.0 的「代码」和 1.0 的「代码」比三层划分暗示的更重叠——项目里 1.0 和 3.0 代码在同一仓库、同一 PR 里共存，这本身就说明三层是视角而非分类。

更深层的批评是「Software 3.0 混淆了工具和运行时」。Karpathy 把「写 prompt」类比为「写代码」，但两者的关键差异在于：代码是确定性的、可 fork 的、可版本化的；而 prompt 是上下文敏感的、同样的 prompt 在不同 context 下产生完全不同的输出。这使得 Software 3.0 的「程序」不具备 Software 1.0 的可复现性——同一个 prompt 在不同会话里可能产生截然不同的结果，这在传统软件工程里是不可接受的。

另一个实践层面的问题是：当程序员把「写代码」换成「组织 prompt 和 context」，原有的代码审查流程（PR、code review、CI）如何适配？目前的答案是「还在探索」——多数团队仍然用 1.0 的流程来管理 3.0 的产物，导致上下文工程的质量控制体系远不如代码工程成熟。 ^[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]

## 现实案例

在一个典型的 Hermes Agent 技能（skill）中，三层完整叠加：skill_view 读取 SCHEMA.md（1.0 的确定性文件 IO），terminal 执行 shell 命令（1.0 的操作系统调用），LLM 决定生成什么输出（3.0 的自然语言推理），Hermes cron 引擎调度（1.0 的进程管理）。Agent 系统工程师的核心能力是在一个系统里同时理解这三层，知道哪层错了、哪层该用哪层的工具来修。 ^[entities/hermes-agent]

Claude Code 的 24 种 Hook 事件是三层叠加的另一个具体案例：`PreToolUse` 和 `PostToolUse` 是 1.0 的拦截器模式，`ToolSearch` 的延迟加载是 3.0 的上下文工程，`AgentTool` 的 7 种执行模式是 2.0/3.0 混合的 subagent 路由。三层没有明确边界，而是在 Hook 这个统一接口下交织运行。 ^[entities/claude-code-core-internals]

LLM Wiki 的文件树结构也是三层叠加的具体体现：`raw/` 是 1.0 的只读存储（追加写入，不修改原文），`concepts/entities/` 是 3.0 的知识编译产物（由 LLM 动态生成），`log.md` 是 2.0/3.0 的操作记录（由 AI 自动维护）。V2 引入的四层记忆机制（工作记忆/情节记忆/语义记忆/程序记忆）进一步在知识管理层面映射了 Software 三层结构——工作记忆对应当前 context window（3.0 的即时运行时），情节记忆对应压缩后的会话摘要（2.0 的统计压缩），语义记忆对应持久化的 entity/concept 页面（1.0 的文件持久化）。 ^[entities/karpathy-llm-wiki-v2-2026]

## 在 wiki 中的关联

- [[entities/karpathy-vibe-coding-agentic-engineering|Karpathy 范式演化]]
- [[concepts/vibe-coding-paradigm|vibe coding（3.0 的入口）]]
- [[concepts/agentic-engineering-paradigm|agentic engineering（3.0 的工程化）]]
- [[concepts/agent-as-software-3-0-substrate|Agent 作为 3.0 substrate]]
- [[concepts/harness-engineering-paradigm-shift|harness 是 3.0 的运行时]]

## 进一步阅读

- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/karpathy-llm-wiki-v2-2026]]

## 所属 MOC

- [[moc/layer-0-foundation|Layer 0 Foundation]]
