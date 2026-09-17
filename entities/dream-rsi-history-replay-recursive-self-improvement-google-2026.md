---
title: "Dream-RSI：把历史记录当模拟器，让 Agent 在「做梦」中优化探索策略（Google）"
created: 2026-09-17
updated: 2026-09-17
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

## 相关

- [[entities/ai-recursive-self-improvement-nanogpt-prime-intellect|nanoGPT × Prime Intellect 的 RSI 实践]]
- [[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026|AI4AI 综述：组合缺口与 RSI]]
- [[concepts/ai-self-improvement-bootstrapping|AI 自我改进的启动难题]]
- [[entities/areal-2-agentic-rl-online-learning-self-evolving|AReaL 2：在线学习与自进化 Agentic RL]]
- [[entities/agent-self-improvement-six-mechanisms|Agent 自我改进六机制]]

→ [[raw/articles/dream-rsi-history-replay-recursive-self-improvement-qbitai-2026|原文存档（量子位）]]
→ [[raw/articles/dream-rsi-evolving-worlds-history-replay-xhs-2026|原文存档（小红书论文转录）]]
