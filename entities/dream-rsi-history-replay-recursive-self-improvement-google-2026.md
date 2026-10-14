---
title: "Dream-RSI：把历史记录当模拟器，让 Agent 在「做梦」中优化探索策略（Google）"
created: 2026-09-17
updated: 2026-10-14
type: entity
tags: [dream-rsi, recursive-self-improvement, rsi, exploration-policy, history-replay, discovery-tree, agent-search, google, self-evolving]
sources: [raw/articles/dream-rsi-evolving-worlds-history-replay-xhs-2026, raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026]
confidence: 0.75
provenance_state: extracted
---

# Dream-RSI：把历史记录当模拟器，让 Agent 在「做梦」中优化探索策略（Google）

> **Background**：本文综合两份同日中文解读（小红书 @乌萨奇今天读paper了吗 的论文转录、量子位 鱼羊 的技术梳理），论文为 Google《Dream-RSI: Recursive Self-Improvement through Evolving Worlds》。两份来源的核心数字互相印证（Lasso 550→317 调用、3587.1→2931.0ms 等），因此本文按多源综合页处理。

## 核心命题：历史记录本身就是模拟世界

Google 的研究者对 RSI（递归自我改进）给出一个关键定位：**RSI 的核心驱动力其实是「高效探索」**，因此如何管控并优化探索策略是一个关键问题。现有 AI 科研系统大多已实现一个 Loop（生成方案 → 跑实验 → 看结果 → 改方案 → 再跑实验），但随着搜索空间扩大，固定探索策略无法自适应，而在线策略优化又受制于迭代时间长、反馈滞后、代价高。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]

Dream-RSI 抓住的是此前未被充分利用的东西——**历史**。一次 Agent 探索任务会留下丰富记录，形成一棵**发现树（Discovery Tree）**，树上保存每次尝试的分支、代码、反馈、得分与所花计算量。研究者换了个视角：这一堆历史日志本身就是一个模拟器。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]

因此，在新的搜索策略下（例如「先跑 C，C1 不够好再走 A」），不必从头重跑——C、C1、A、A1 的结果都能直接调用，只要让新策略在旧树上「重新走一遍」，就能算出「如果当时这么做，需要多少计算量、能达到什么效果」。**在 Agent 已探索过的区域里，历史就是世界。**^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]

## 三步循环：真实探索 → 做梦 → 把赢家派出去

整个系统分三步。**第一步，真实探索**：当前探索策略控制一个 Coding Agent，决定开哪些分支、继续哪些方案、跑多少并行任务、何时结束，所有过程记为发现树。**第二步，开始做梦**：引入另一个大语言模型负责开发探索策略，每生成一种新方案都不重跑实验，而是在过去积累的所有发现树上「回放」结果，最终综合答案质量、搜索成本、并行效率选出更好的动态策略。**第三步，把赢家重新派出去**：新策略进入真实环境指挥下一轮探索，由于策略已变化，它可能走到上一代未抵达的位置，从而产生新的发现树、继续扩大下一轮的「梦境」。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]

循环因此闭环为：真实探索 → 更多历史 → 更多模拟世界 → 在模拟世界中优化探索策略 → 更好的策略产生新的历史。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]

这一结构把自进化拆成两层：**对象层**改「找到什么解」，**元层**改「怎样寻找解」；本文的价值在于首次把元层做成持续循环，而底层模型、评价器与工具接口保持不变，只改变探索策略。^[raw/articles/dream-rsi-evolving-worlds-history-replay-xhs-2026.md]

## 一个反直觉的实验结论：历史不该被写进提示词

论文对照了「把历史总结成搜索方向写进提示词」的做法，结论是**轨迹总结成搜索方向反而限制多样性**——在 ConvDiv 任务上弱于可交互的模拟器。历史的正确用法是**低成本比较多种搜索方式**，而不是告诉智能体往哪搜。^[raw/articles/dream-rsi-evolving-worlds-history-replay-xhs-2026.md]

## 实验读数：跨三类任务的调用量与性能

研究团队把 Dream-RSI 放到 8 个发现任务、横跨算法工程、数学优化与 GPU Kernel 优化三个方向，最直接的对照组是 **Recursive Fixed Exploration**（同模型、同 Evaluator、同初始策略与资源约束，但永远按第一轮固定策略继续搜索）。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]

- **算法工程（Lasso regularization path）**：Gemini 3.1 Pro 下，固定探索消耗 550 次 Agent 调用、六个数据集平均运行时间 3587.1ms；Dream-RSI 用 317 次调用做到 2931.0ms。Gemini 3.7 Flash 下，固定策略 3200 次调用、平均 2516.7ms，Dream-RSI 用 1879 次、降到 2350.6ms。与 SimpleTES（51200 次调用预算）相比，部分设置下调用量差出最高 **162 倍**。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]
- **GPU Kernel**：VGG16 与 LayerNorm 任务上分别以 **2.43× / 1.79×** 更少的生成次数达到相近性能；ConvDiv 与 ConvMax 上在接近预算下把性能分别提高 **2.09× / 1.44×**。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]
- **数学优化**：Sum Difference 拿到 1.145427（高于论文列出的多个基线），Circle Packing 打到 2.635983；Auto Correlation 上 SimpleTES 仍是 SOTA，但 Dream-RSI 在结果相近的情况下把调用量从 51200 压到 1000。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]
- **数据点（两份来源互证）**：Lasso 搜索 Gemini-3.1-Pro 550→317 调用、3587.1→2931.0ms；Flash 3200→1879 调用、最优 Avg 2350.6ms；Table 1 中 Sum Diff 1.145427 为全场最优。^[raw/articles/dream-rsi-evolving-worlds-history-replay-xhs-2026.md]

## 可迁移判据

- **把「历史」当模拟器，而不是当记忆**：agent 自进化系统的历史轨迹可用于**离线比较多种探索策略**（成本/质量/并行效率三维打分），不必为每个候选策略付真实执行成本。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]
- **元层与对象层分离**：固定底座模型、评价器与工具接口，只让「探索策略」这一层持续迭代——这是把 RSI 做成工程闭环的前提，也呼应既有 agent 自我改进机制的分类讨论（见 [[concepts/agent-self-improvement-loops|Agent 自改进循环]]、[[entities/agent-self-improvement-six-mechanisms|六种自我改进机制]]）。^[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026.md]
- **省的是调用，不是精度**：多数任务以显著更少的 Agent 调用达相近或更好结果（最高 162 倍差距），说明在搜索空间大的科研 agent 场景里，「探索策略」本身比「换更贵的底座模型」更值得优化。

## 深度分析

### 历史记录即模拟世界的命题

「历史即模拟器」这一命题的要害在于重新划定了模拟的覆盖范围：它不承诺模拟 Agent 没去过的区域，只对**已探索过的区域**成立。发现树里每个节点都带着三元组（结果、得分、计算量），这恰好构成一个可查询的环境模型——新策略在旧树上重放时，「如果当时先跑 C 会怎样」不再是猜测，而是对已存节点的查表与组合。这使它区别于世界模型类方法：后者要学习对未见过状态的泛化，Dream-RSI 只需对历史做反事实重放，代价几乎为零，但适用范围也严格被历史的覆盖面框定。这也解释了系统为什么必须闭环——真实探索的职责不只是解题，而是**扩大模拟器的覆盖面**。

### 做梦-探索-外派三步循环

三步循环的分工值得细看。真实探索阶段，探索策略是执行者；做梦阶段，策略开发 LLM 是**离线优化器**——它对候选策略的评估完全建立在重放之上，天然可并行、无真实执行成本；外派阶段则是唯一的**在线验证**，新策略走到旧策略未覆盖的位置，补齐模拟器的盲区。关键的非对称在于：做梦可以无限次犯错（重放免费），外派只能少量试错（真实执行昂贵）。这个结构实际上把「探索-利用」权衡从单次任务内部，上移到了「梦境搜索 vs 真实外派」的元层面。与在线学习方法（如 [[entities/areal-2-agentic-rl-online-learning-self-evolving|AReaL 2]]）相比，Dream-RSI 把反馈滞后问题转化为离线批优化问题；它延续了 [[concepts/agent-self-improvement-loops|Agent 自改进循环]] 中的「经验回放」思路，但重放对象从「单条轨迹」换成了「整棵可交互的搜索树」。

### 反直觉结论「历史不进提示词」的技术机制

把历史总结成搜索方向写进提示词反而变差，论文给出的机制解释是：长程科研搜索同时存在大量并行路线，过早总结「高层结论」相当于给未来探索注入强先验，会**提前堵死那些看似无望、实则可能有价值的分支**——多样性的损失发生在提示词层面，且不可逆。而把同样的历史交给模拟器，历史只被用来**比较**候选策略的优劣，从不直接命令 Agent 往哪搜；搜索方向仍由策略在真实执行时即时决定。这与记忆设计的一条反面经验一致：并非所有经验都该被沉淀成语义结论（参见 [[concepts/when-not-to-memory-architecture|何时不应做记忆架构]]、[[concepts/episodic-vs-semantic-memory-agent|情节记忆与语义记忆]]）——Dream-RSI 保留了历史的情节形态（完整轨迹），拒绝过早压缩成语义规则。

### 可迁移判据的适用边界

「历史当模拟器」的判据有三条隐含前提，越界即失效。其一，**任务过程可记录、可重放**：GPU kernel 和数学优化这类任务的每次尝试都有确定性的代码与评分，重放才可信；若探索涉及真实物理实验或不可复现的外部调用，历史节点无法等价重演。其二，**探索成本远高于重放成本**：判据的红利来自调用次数的压缩（最高 162 倍），若单次执行本身便宜，离线策略优化的工程开销可能不划算。其三，**搜索空间可被策略差异化遍历**：发现树必须能被不同策略走出不同路径，否则模拟器只是在反复验证同一片区域。此外，论文实验中底座模型、评价器与工具接口始终未变，严格说这是 harness 层的自进化而非模型自进化（对照 [[entities/rrsi-regularized-recursive-self-improvement-agent-harnesses|R²SI：正则化递归自改进]]），把结论外推到「模型权重也要进化」的场景需要额外论证。

## 实践启示

1. **审计你的 Agent 留下了什么**：如果多轮 agent 系统没有把每次尝试的（代码、反馈、得分、成本）结构化存档，它就永远失去离线比较策略的机会。发现树的最小单元是「分支 + 结果 + 成本」，先落盘再谈优化。
2. **新策略先在旧历史上重放评分，再付真实执行成本**：候选策略的价值评估可以做到近乎零边际成本，规则应是「重放排名靠前才允许上线一次真实探索」，而不是让每个候选策略都真实跑一遍。
3. **不要把历史总结写进主提示词**：特别是在并行路线多的长程搜索任务里，显式语义指导会提前收缩搜索空间。历史的正确消费方是策略评估器，不是执行 Agent。
4. **元层与对象层分开迭代**：固定模型、评价器与工具接口，只让探索策略这一层循环进化，能显著缩小每次改进的回归范围，也让失败可归因——这是把 RSI 做成可持续工程闭环的结构前提。
5. **把探索的产出物视为模拟器而非日志**：日志用于事后审计，模拟器用于反事实推演；两者对存储格式的要求不同（后者需要可重放的完整上下文），设计存储时就应按模拟器的需求来。

## 相关

- [[entities/ai-recursive-self-improvement-nanogpt-prime-intellect|nanoGPT × Prime Intellect 的 RSI 实践]]
- [[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026|AI4AI 综述：组合缺口与 RSI]]
- [[concepts/ai-self-improvement-bootstrapping|AI 自我改进的启动难题]]
- [[entities/areal-2-agentic-rl-online-learning-self-evolving|AReaL 2：在线学习与自进化 Agentic RL]]
- [[entities/agent-self-improvement-six-mechanisms|Agent 自我改进六机制]]

→ [[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026|原文存档（量子位）]]
→ [[raw/articles/dream-rsi-evolving-worlds-history-replay-xhs-2026|原文存档（小红书论文转录）]]
