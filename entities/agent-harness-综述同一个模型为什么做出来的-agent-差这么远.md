---

title: "Agent Harness 综述：同一个模型，为什么做出来的 Agent 差这么远"
type: entity
created: 2026-07-04
updated: 2026-10-09
tags: [wechat, ai]
rating: v8c8
sources:
  - raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远
  - raw/articles/harness-what-is-model-outside-execution-system-ruofei
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Agent Harness 综述：同一个模型，为什么做出来的 Agent 差这么远

**来源**: 架构师

**发布日期**: 2026-04-19^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]


**原文链接**: https://mp.weixin.qq.com/s/h49UiGERvz8BMkMW0_4Gwg ^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

---

架构师（JiaGouX）

我们都是架构师！

架构未来，你来不来？

最近多看几篇 Agent 文章，就会反复遇到同一个词： Harness 。^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]


但这个词越讲越糊。

有人把它理解成工具系统。有人把它理解成 Prompt 外面那层壳。也有人把它理解成多 Agent 编排、Memory、Sandbox、Hooks、Skills 这些东西的总和。 ^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

这些说法都沾边，但都还没有落到主要承重的地方。^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]


这两天反复读 Akshay 写的  The Anatomy of an Agent Harness  ，最有启发的地方，不是又盘了一遍产品名单，而是它把视角从“模型有多强”挪到了“模型外面那套系统到底在干什么”。 ^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

原文信息量很大，也铺得很开。我们不做逐段翻译，有兴趣可以直接看原文。我们换另外一个角度来看：^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]


同一个模型，为什么做出来的 Agent 会差这么远。^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]


我自己的理解是： Harness 是在模型和真实交付之间，补上一套可运行、可恢复、可验证、可治理的软件系统。 ^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

把这个问题想清楚以后，Harness 不再像一个什么都能往里装的热词了。^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]


## 太长不看版

- • Harness 是包住模型的整套运行系统：主循环、工具、上下文、状态、权限与错误、验证。

- • Prompt 管怎么表达任务，Context 管模型看到什么，Harness 管系统怎么跑、怎么停、怎么纠偏。

- • 2026 年大家都在讲 Harness，因为模型能力上来之后，瓶颈从“能不能答”转向“能不能稳定交付”。

- • 同一个模型，只换 Harness，不换权重，结果可能差出一个量级。LangChain 只换外围基础设施，就从 TerminalBench 2.0 前 30 名外拉到第 5。

- • 模型和 Harness 是协同演化的。Claude Code 的模型在训练阶段就把特定 Harness 放进了训练回路。

- • Harness 的演进方向是变薄，Manus 半年内重建五次，每次都在做减法。但 Harness 不会消失。

- • 如果一个 Agent 还不稳定，值得先检查的，通常是 Harness。

## Harness 不只是一层壳

我们先把三个词分开看。

- •
  Prompt Engineering
  ：解决的是“怎么对模型说”。

- •
  Context Engineering
  ：解决的是“让模型在这一轮看到什么”。

- •
  Harness Engineering
  ：解决的是“整套系统怎么运行，怎么持久化，怎么验证，怎么兜底”。 ^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

这三层是包含关系。

Prompt 更像指令。

Context 更像喂给模型的工作台。

Harness 更像操作系统。 Akshay 引用了 Beren Millidge 2023 年的类比：裸的 LLM 是一颗没有 RAM、没有磁盘、没有 I/O 的 CPU。上下文窗口充当内存，外部数据库充当磁盘，工具集充当设备驱动。让这台机器持续跑起来的，是外面这套调度、执行、校验和保护机制。 ^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

Harness 回答的是一个工程问题：怎样把一个无状态、会推理的模型，变成一个能持续交付结果的系统。 ^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

聊天模

^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

→ [[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远|原文存档]] ^[raw/articles/agent-harness-综述同一个模型为什么做出来的-agent-差这么远.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

## 深度分析

**1. 同一模型分岔的真正机制：上下文才是每次实际执行的"程序"。** 裸模型只是一个无状态的概率分布，它每次跑出的行为完全由喂进去的上下文决定。两个产品用同一组权重，但主循环组装上下文的方式不同——工具定义怎么写、Observation 怎么回写、历史怎么压缩、失败信息是否显式保留——等于在运行两份不同的"程序"。从这个角度看，"同一个模型换 Harness 结果差一个量级"（LangChain 在 TerminalBench 2.0 上从 30 名外跳到第 5）并不是神秘现象，而是上下文组装策略差异的直接体现。Harness 的核心工作，就是把模型的通用推理分布约束到一条可预期的执行轨迹集合上，这也是 [[concepts/context-management-agent-systems|Context Management]] 与 [[concepts/context-engineering|Context Engineering]] 在 Harness 体系内地位持续上升的原因。

**2. 验证回路决定误差是被清除还是被复利。** 10 步任务、每步 99% 成功率，全链路只剩约 90%——误差随步数复利累积，这是所有长任务 Agent 的宿命方程。工具给了模型行动能力，只有验证才给了模型纠错能力：没有外部反馈回路，失败会沿着轨迹静默传播，最终产出"看着像完成了"的交付物；有外部验证（测试、lint、截图、真实 API 响应），每一步的误差才有机会被就地清除而不是累积。Boris Cherny 给出的 2-3 倍质量提升，本质上是把误差模型从"累积"改成了"收敛"。而验证必须外移的原因也很硬：让生成器自验自己，它最容易放过自己。参见 [[concepts/verifier-driven-development|Verifier-Driven Development]]。

**3. Harness 的厚度悖论，以及它被写进权重后的身份转变。** Harness 太薄，稳定性只能靠模型自觉；太厚，则笨重昂贵且与当前模型强绑定。Manus 半年重建五次都在做减法、Vercel 在 v0 上删掉 80% 的工具反而更好、Claude Code 靠懒加载砍掉 95% 上下文——这些案例共同指向一个判断：Harness 是针对当前模型能力欠拟合而搭的脚手架，模型每上一个台阶，就应该拆掉一批不再承重的结构。但 Claude Code 把特定 Harness 放进训练回路这件事，把这个悖论推向了更深的张力：当 Harness 成为训练分布的一部分，它就不再是"待拆除的脚手架"，而是被固化进权重的永久构件——换一套工具实现反而掉分。于是 Harness 同时具有两种身份：对下一代模型是临时 scaffolding，对当前模型是 permanent fixture。何时拆、何时焊死，没有公式，只有持续评测，这正是 [[concepts/harness-engineering-framework|Harness Engineering]] 至今未收敛的核心难题。

**4. Harness Engineering 是 30 年软件工程复杂性问题的最新实例。** 设计模式解决对象协作的复杂性，分层架构与 DDD 解决业务边界的复杂性，微服务解决分布式运维的复杂性，Harness 解决的是一个会推理、会执行、还会不断消耗上下文预算的系统的复杂性——对象一直在换，问题从未变过：把复杂系统变成可控系统。这也解释了为什么工程师对 Harness 有天然的熟悉感。它的真正价值不在于让模型更能干，而在于让系统重新变得可设计、可治理、可拆边界；评判一个 Harness 的标准，不是组件数量，而是任务是否可见、动作是否可控、状态是否可恢复、结果是否可验证。

**5. 未解的开放问题。** 综述读完仍有三处边界模糊：其一，"记忆是提示、不是事实"的原则正确，但记忆权重与真实状态校验的比例如何随任务长度动态调整，尚无成熟做法（参见 [[concepts/agent-memory-architecture|Agent 记忆架构]]）；其二，Translation Layer 只解决接入、不解决等价，跨 Provider 迁移的质量关卡只能靠重新评测兜底，意味着 Harness 与 Provider 之间始终存在一层不可抹平的语义损耗；其三，榜单成绩（76.4% 通过率的 Harness 搜索实验）不能直接等同于真实产品体验，Harness 优化的评估方法学本身还是一门不成熟的学科。

## 补强（2026-09-05）：Translation Layer 解决接入、不解决等价 + Earendil 四元定义（若飞）

2026-09-04 若飞《Harness 到底是什么》在既有综述的"同一个模型换 Harness 结果不同"主线上，补两个此前零覆盖的维度：^[raw/articles/harness-what-is-model-outside-execution-system-ruofei.md]

- **Earendil《What is a Harness?》四元定义**：把 Harness 最小结构拆成 System Prompt（给规则与背景）、Tools（让模型读写执行）、Agentic Loop（把调用与结果串成循环）、Translation Layer（处理不同模型接口/消息/工具格式差异）四项；并给出架构视角——Harness 既是模型运行环境，也是模型与真实系统之间的执行边界。^[raw/articles/harness-what-is-model-outside-execution-system-ruofei.md]

- **Translation Layer 解决接入、解决不了等价**：接口接通 ≠ 行为等价。不同模型对系统消息、工具定义、推理信息、提示词缓存、流式事件处理各不相同，有些字段还要下一轮原样带回；Pi 作者 Mario Zechner 将跨 Provider 上下文交接称为 `best effort`（尽力转换、不承诺语义无损），Armin Ronacher 也因统一 SDK 无法抹平模型/Provider 侧差异而改直接掌握各家 SDK。**模型迁移三道关**：①API 能调用、②任务状态与产物能带走、③换完后质量/成本/安全过线——前两道靠适配与自身数据边界，最后一道只能重新评测。^[raw/articles/harness-what-is-model-outside-execution-system-ruofei.md]

- **安全边界与动作/证据双链**：提示词只是第一道约束、不是最后一道安全边界（Pi 的 Project Trust 不是沙箱，默认继承启动用户权限，真实隔离靠容器/VM/microVM/远程沙箱）；代码任务走"动作向外（模型→校验权限→系统调用）、证据向内（输出/结果/产物/审批放回上下文）"两条方向相反的链，工程难题多出在两条链交接处。^[raw/articles/harness-what-is-model-outside-execution-system-ruofei.md]

- **架构设计五问**：谁拥有业务事实/谁批准动作与权限校验/中断后从哪恢复、结果不明找谁确认/模型说"完成"后哪份证据才算完成/Provider 工具提示词变化后拿什么重新评估质量。^[raw/articles/harness-what-is-model-outside-execution-system-ruofei.md]

→ [[raw/articles/harness-what-is-model-outside-execution-system-ruofei|原文存档]]

