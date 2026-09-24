---

title: "Harness Evolution vs 多跑几遍实证"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: ['agent', 'harness', 'evaluation']
sources: [raw/articles/harness-evolution-vs-test-time-scaling-2026]
confidence: 0.7
vxc: 49
score: v=7/c=7/stars=4
provenance_state: extracted
---

# Harness Evolution vs 多跑几遍实证



# 复杂的Harness Evolution，甚至不如多跑几遍

机器之心 2026-09-16 12:00 四川

当 “自我进化” 的 Agent 得分提高时，我们怎么知道它是真的学会了更好的工作方式，而不是因为它比普通 Agent 多获得了几次尝试机会？ ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

  * 论文标题：Rethinking the Evaluation of Harness Evolution for Agents

  * 论文链接：<https://arxiv.org/abs/2607.12227>



  * 代码：<https://github.com/rethinking-harness-evolution>



  * 博客：<https://yikee.github.io/harnessevolution/>




一个 Agent 第一次没把任务做对。

接下来，你有五份额外的推理预算，可以有很多种花法。

你可以让它把同一道题重新做几遍，从多个结果里挑最好的；可以把上一次失败的过程交给它，让它接着修；也可以做一件听起来更 “高级” 的事 —— 让 Agent 分析自己的失败，修改 Prompt、工具、Memory 和控制逻辑，甚至自动重写整个运行框架。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

最后一种思路，正在成为 Agent 研究里一个很热门的方向：Harness Evolution。

它背后的愿景非常吸引人…

## 核心内容



# 复杂的Harness Evolution，甚至不如多跑几遍

机器之心 2026-09-16 12:00 四川

当 “自我进化” 的 Agent 得分提高时，我们怎么知道它是真的学会了更好的工作方式，而不是因为它比普通 Agent 多获得了几次尝试机会？ ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

  * 论文标题：Rethinking the Evaluation of Harness Evolution for Agents

  * 论文链接：<https://arxiv.org/abs/2607.12227>



  * 代码：<https://github.com/rethinking-harness-evolution>



  * 博客：<https://yikee.github.io/harnessevolution/>




一个 Agent 第一次没把任务做对。

接下来，你有五份额外的推理预算，可以有很多种花法。

你可以让它把同一道题重新做几遍，从多个结果里挑最好的；可以把上一次失败的过程交给它，让它接着修；也可以做一件听起来更 “高级” 的事 —— 让 Agent 分析自己的失败，修改 Prompt、工具、Memory 和控制逻辑，甚至自动重写整个运行框架。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

最后一种思路，正在成为 Agent 研究里一个很热门的方向：Harness Evolution。

它背后的愿景非常吸引人。今天是工程师观察 Agent 为什么失败，然后修改 Prompt、添加工具、设计 Memory、调整 workflow；未来，也许这些工作可以交给 Agent 自己。Agent 不只是完成任务，还能够修改支撑自己工作的系统，从而一轮比一轮更强。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

但这里隐藏着一个很容易被忽略的问题：

当 “自我进化” 的 Agent 得分提高时，我们怎么知道它是真的学会了更好的工作方式，而不是因为它比普通 Agent 多获得了几次尝试机会？ ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

AI2 和华盛顿大学的一项最新研究，专门把这个问题摆到了台面上。

研究者做的事情并不复杂：把 Harness Evolution 和最简单的 “多跑几遍” 放到尽可能公平的条件下，给它们相同的反馈、相近的 inference budget，然后看看究竟谁更有效。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

结果有些出人意料。在他们的实验中，复杂的 Harness Evolution 并没有稳定胜过简单的 test-time scaling。更值得注意的是，当进化出来的 Harness 被拿去解决从未参与优化的新任务时，提升只剩下了很小的一部分。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

这意味着，我们可能需要重新理解过去一些所谓 “Agent 自我进化” 的提升。

Harness，其实就是模型外面的那套 “操作系统”。一个大模型真正成为 Agent，通常不只是因为模型本身。模型外面还有一整套系统：Prompt 怎么写、能调用哪些工具、有没有 Memory、如何验证结果、失败以后是否重试、什么时候停止、下一步看到什么信息…… 这些东西加起来，可以理解成 Agent 的 Harness。同一个模型，底层参数完全不变，仅仅因为 Harness 不同，表现就可能出现明显差异。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

过去 Harness 大多依赖工程师手工优化。Agent 跑一次任务，工程师阅读 trajectory，发现某个工具调用方式不合理，于是修改 Prompt；发现 Agent 总忘记前面的信息，就加入 Memory；发现它不会检查答案，再增加 verifier。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

Harness Evolution 想做的，就是把这套人工迭代过程自动化。Agent 运行任务，观察结果，总结失败经验，自动修改自己的 Harness，再运行一轮，然后继续修改。Meta-Harness、AHE、AEVO 等工作，都在探索类似的方向。如果这条路最终走通，它的意义很大：Agent 系统可能开始具备某种形式的自动工程优化，甚至成为递归自我改进的一部分。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

问题是 —— 这种 “进化” 本身也是一种搜索。真正应该比较的，不是 “进化前” 和 “进化后” 假设一个普通 Agent 只运行一次，得分是 70。然后 Harness Evolution 允许系统连续运行五轮：分析失败、修改 Harness、重新执行、再次分析…… 最后得分变成 75。我们能不能因此说：Harness Evolution 让 Agent 提升了 5 分？ ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

不一定。因为在第二种情况下，系统不只改变了 Harness，同时也使用了更多推理计算、执行了更多 trajectory、获得了更多反馈。真正公平的对照应该是：如果不修改 Harness，也给普通 Agent 五次运行机会，会发生什么？例如，同一道任务直接独立运行五次，从中选择最好的一个。或者第一次失败以后，让 Agent 看着之前的结果继续修改，连续修五轮。这些方法没有复杂的 “自我进化” 模块，本质上只是增加测试时计算，因此通常被称为 test-time scaling。这也是这篇论文想回答的核心问题： ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

在反馈和计算预算基本一致的情况下，

修改 Harness 本身究竟贡献了多少额外收益？

五份预算，四种花法。研究者在 Terminal-Bench 2.1 上做了对照实验。为了尽量减少其他因素干扰，所有方法都从一套非常简单的初始 Harness 出发：只有一个 bash 工具，没有额外 Skills，没有 Middleware，也没有持久化 Memory。使用的模型包括 Claude Opus 4.6、GPT-5.4 和 GPT-5.4 mini。每个任务统一获得 (K=5) 的计算预算。别只是：这五份预算怎么用。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

第一种是 Parallel Sampling。Harness 从头到尾都不修改。同一个任务独立执行多次，再从候选结果中选择最终答案。可以把它理解为：不会就多做几遍。 ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

第二种是 Sequential Refinement。Harness 同样固定，但后一次执行能够利用前一次留下的结果和反馈。不是从… ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]

→ [[raw/articles/harness-evolution-vs-test-time-scaling-2026|原文存档]] ^[raw/articles/harness-evolution-vs-test-time-scaling-2026.md]