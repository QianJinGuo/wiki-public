---

title: "超级AI背后的秘密武器：Agent Harness深度解析"
type: entity
created: 2026-07-04
updated: 2026-09-23
tags: [wechat, ai]
rating: v7c8
sources:
  - raw/articles/超级ai背后的秘密武器agent-harness深度解析
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 超级AI背后的秘密武器：Agent Harness深度解析

**来源**: Unknown

**发布日期**: 2026-04-15^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]


**原文链接**: https://mp.weixin.qq.com/s/Y0kRzN1DbX7My3cz4eVR1Q ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

---

## 你的 AI 为什么总在演示时掉链子？

你做过一个聊天机器人。也许还接上了几个工具，跑个 ReAct 循环。演示的时候一切正常。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]


然后你尝试把它做成生产级产品，轮子就开始掉了：模型三步之后就忘了自己干了什么，工具调用悄无声息地失败，上下文窗口里塞满了垃圾数据。 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

问题不在你的模型。问题在模型周围的一切。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]


LangChain 证明了这一点：他们只改了 LLM 外围的基础设施（同样的模型，同样的参数），排名就从 TerminalBench 2.0 的前 30 名之外一路飙升到第 5 名。还有一个研究项目，让 LLM 自己优化基础设施，最终达到 76.4% 的通过率，超过了手工设计的系统。 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

这套基础设施，现在有个专门的名字： Agent Harness 。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]


## 什么是 Agent Harness？

这个术语是 2026 年初正式提出的，但概念早就存在了。Harness 是包裹 LLM 的完整软件基础设施：编排循环、工具、记忆、上下文管理、状态持久化、错误处理和防护机制。 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

Anthropic 的 Claude Code 文档说得很直白：SDK 就是"驱动 Claude Code 的 Agent Harness"。OpenAI 的 Codex 团队用同样的框架，明确把"Agent"和"Harness"等同起来，指的都是让 LLM 有用的非模型基础设施。 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

我很喜欢 LangChain 的 Vivek Trivedy 那句经典公式： "如果你不是模型，那你就是 Harness。" ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

这里有个容易混淆的区别。"Agent"是涌现出来的行为——那个有目标导向、会使用工具、能自我纠正的实体，是用户实际交互的对象。Harness 是产生这种行为背后的 machinery。当有人说"我做了个 Agent"，他们的意思是"我做了个 Harness，然后接了个模型"。 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

Beren Millidge 在 2023 年的文章里把这个类比讲得更精确。他把原始 LLM 比作一台只有 CPU、没有内存、没有硬盘、没有输入输出设备的计算机。上下文窗口充当内存（快但容量有限），外部数据库充当硬盘存储（容量大但慢），工具集成充当设备驱动。 Harness 就是操作系统 。 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

正如 Millidge 写的："我们重新发明了冯·诺依曼架构"，因为这对任何计算系统来说都是自然的抽象。 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

## 工程三层论

围绕着模型，有三个同心圆式的工程层次：

- •
  提示工程
  ：雕琢模型收到的指令

- •
  上下文工程
  ：管理模型能看到什么、什么时候看到

- •
  Harness 工程
  ：涵盖以上两者，再加上整个应用基础设施——工具编排、状态持久化、错误恢复、验证循环、安全执行和生命周期管理 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

Harness 不是提示的包装器。它是让自主 Agent 行为成为可能的完整系统。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]


## 生产级 Harness 的 11 个核心组件

综合 Anthropic、OpenAI、LangChain 和更广泛的实践社区，一个生产级的 Agent Harness 有 12 个不同的组件。 ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

### 1. 编排循环

这是心脏跳动。它实现了思想 - 行动 - 观察（TAO）循环，也叫 ReAct 循环。循^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]


^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

→ [[raw/articles/超级ai背后的秘密武器agent-harness深度解析|原文存档]] ^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md]

## 深度分析

演示与生产之间的鸿沟，本质上是**无状态性 + 环境漂移**的组合效应。LLM 本身完全无状态：每次调用结束，模型对之前发生的一切"失忆"。演示环境里任务短、工具少、上下文干净，这种失忆不致命；而生产环境里任务链条长（10 步流程、每步 99% 成功率，端到端只剩约 90.4%）、工具会静默失败、上下文会腐烂——关键内容掉进窗口中段时，模型性能下降超过 30%（Chroma 证实了斯坦福 "Lost in the Middle" 的发现）。两股力叠加，就是"演示正常、上线掉链子"的机理。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md:24-30, 94-96]

从这个角度看，Harness 工程是一门让 Agent 行为**可重复**的工程学科。模型能力是概率性、非确定性的；Harness 的职责是用确定性的基础设施——持久化状态、分级重试、验证循环、权限控制——把概率性输出约束到可接受的方差内。LangChain 在 TerminalBench 2.0 上不动模型、只改外围基础设施就从 30 名外冲到第 5 名，是"可重复性来自系统而非模型"的最直接证据。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md:28-30, 72-76]

工程三层论的责任边界可以这样划：提示工程只对"这一次输入"负责；上下文工程对"模型能看到什么、什么时候看到"负责；Harness 工程对"整个系统能否可靠交付结果"负责。Millidge 的冯·诺依曼类比让边界更直观——模型是 CPU，Harness 是操作系统：操作系统不替 CPU 计算，但没有它 CPU 无法与任何外设协作。把应用层逻辑（业务规则、UI）和 Harness 层（编排、状态、防护）混在一起，正是多数自研 Agent 稳定性问题长期修不好的根因。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md:44-64]

11 个核心组件实际是一张生产就绪检查清单。按"缺失后最先让系统崩塌"排序，最常缺的三个是：**上下文管理**（组件 4，静默失败的头号来源）、**错误处理**（组件 8，错误累积速度远超直觉，且需区分瞬态/LLM 可恢复/用户可修复/意外四种类型）、**验证循环**（组件 10，Boris Cherny 指出给模型自验方式可将质量提升 2-3 倍）。编排循环、工具、输出解析这三项最简单、多数人能写到；真正把玩具与生产分开的，是记忆、状态管理、防护机制和子 Agent 编排这几个"重基础设施"。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md:96, 142-148, 158-164]

## 实践启示

给个人 Agent 搭**最小可用 Harness**，只需四件事：(1) 一个 while 循环实现 TAO 循环（编排循环，"傻瓜循环"即可）；(2) 少量带清晰 schema 的工具定义；(3) 会话结束后把关键状态写入文件、下次启动读回（记忆 + 状态管理，Claude Code 式的"git 提交 + 进度文件"就够用）；(4) 每个工具调用包一层异常处理，失败作为错误消息喂回模型而不是崩掉（错误处理）。四件齐了，个人级 Agent 就能跨过"演示可用"到"日常可用"的门槛；其余组件按需再加。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md:72-74, 136-138, 142-146]

审计现有 Agent 的 Harness 缺口，按 11 组件逐项问四个问题：有没有？失效了会发生什么？能观察/调试吗？恢复路径是什么？——重点盯三处：上下文是否无限增长而没有压缩/掩码/即时检索策略（组件 4）；工具失败是被吞掉还是被模型看见、四种错误类型是否区分对待（组件 8）；有没有任何形式的输出验证循环——规则验证、视觉反馈或 LLM 评委（组件 10）。三项里有两项答不上来，这个 Agent 就还在玩具区间。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md:142-148, 158-164]

自建顺序上，**先建验证循环，再建上下文管理，最后才考虑子 Agent**。验证循环是性价比最高的单点投入——基于规则的反馈（测试、linter、类型检查器）几乎零成本，就能把质量拉高 2-3 倍。上下文管理次之：压缩 + 即时检索两条策略能覆盖大部分上下文腐烂场景。子 Agent 编排和防护机制的完整架构属于多 Agent / 多用户阶段的事，个人阶段过度建设是常见的时间陷阱。^[raw/articles/超级ai背后的秘密武器agent-harness深度解析.md:98-118, 158-164]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

