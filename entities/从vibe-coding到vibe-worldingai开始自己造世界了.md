---

title: "从Vibe Coding到Vibe Worlding：AI开始自己“造世界”了"
created: 2026-09-01
updated: 2026-09-01
type: entity
tags: [ai, agent, harness, evaluation, mcp, multimodal]
sources: [raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了]
confidence: 0.75
provenance_state: extracted
---

# 从Vibe Coding到Vibe Worlding：AI开始自己“造世界”了


# 从Vibe Coding到Vibe Worlding：AI开始自己“造世界”了

腾讯技术工程 2026-08-31 17:36 广东

和 AI 对话

> 如果说 **Vibe Coding** 让「写代码」这件事从「敲键盘」变成了「和 AI 对话」，那么今天要介绍的这项工作，可能正在把同样的故事在 **3D 世界构建** 领域重演一遍。

**开源地址：**

  * 项目主页：https://usail-hkust.github.io/VibeWorlding-Gym/

  * Code（沙盒环境与训练框架）：https://github.com/usail-hkust/VibeWorlding-Gym

  * Dataset（VWE-Bench 评测基准）：https://huggingface.co/datasets/usail-hkust/VWE-Bench

  * Model（VibeWorlder 模型合集）：https://huggingface.co/collections/usail-hkust/vibeworlder




香港科技大学（广州）联合腾讯 AI 平台部团队，提出了 **VibeWorlding** ：一个可以像 Agent 写代码一样，通过多轮对话、调用工具、观察渲染反馈来**自主构建和编辑交互式 3D 世界** 的多模态智能体框架。你只需要说一句「给我造一个阴森的荒漠神殿」，或者「把桌子旁边的那把椅子往前挪两米」，剩下的检索 3D 资产、摆放物体、避免碰撞、渲染确认，统统交给 Agent 完成。 ^[raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了.md]

这套系统的核心不是又一个「文本转 3D」的生成模型，而是一整套**可训练、可验证、可复现** 的基准与强化学习框架 —— 同时开源了 **6,828** 条多模态用户query、**2,616** 个高质量 3D 资产、**323** 个人工标注的种子3D世界，以及一整套沙盒环境和双重约束验证器。实验结果显示，经过强化学习后训练的开源模型 VibeWorlder-30B-A3B，在综合 Pass@1 指标上**反超了 GPT-5.5 和 Qwen3.8-Max** ，成为目前该任务上表现最好的模型。 ^[raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了.md]

 _图 1：VWE-Bench 与 VibeWorlding-Gym 总览_

###  为什么造 3D 世界需要一次范式转移？

游戏关卡、数字孪生、具身智能仿真环境……这些场景都依赖大量**可交互、可编辑的 3D 世界** ，但传统的人工搭建方式效率极低。近年来，随着多模态大模型（MLLM）的兴起，学界开始探索用 AI 智能体自动化完成这一过程，大致分为两条路线： ^[raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了.md]

  * **固定工作流（Fixed Workflow）：** 如 SceneCraft、3D-GPT，把构建过程拆成「规划 - 生成 - 评审」的流水线，每个阶段配一个专门的子智能体。

  * **自主智能体（Autonomous Agentic）：** 如 SAGE、SceneWeaver、SceneReVis，把 3D 资产检索、编辑、渲染都封装成工具集，让智能体自主决策、多轮交互、自我修正。




但这些工作普遍存在两个尴尬的现实：一是评测用的查询**过于理想化和简单** ，难以反映真实用户天马行空、模糊不清、甚至互相矛盾的表达；二是几乎所有框架**都不开源** ，学界既无法公平对比，也无法系统研究「训练是否真的能提升这些底层能力」这个更根本的问题。 ^[raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了.md]

_一个可行的 3D 世界，既要「物理上站得住」（不悬空、不穿模），也要「语义上说得通」（符合用户意图、风格协调）。这种多维度的正确性，仅靠人工规则或者一个通用的 LLM 裁判都很难兼顾——这正是 VibeWorlding 要解决的核心难题。_ ^[raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了.md]

###  VibeWorlding 到底是什么？

这一研究，把整个「3D 世界构建」任务，形式化为一个**多轮、多模态、工具集成的推理过程** ：给定一个多模态请求（纯文本描述，或者「现有世界 + 编辑指令」），智能体需要自主推断用户意图、规划场景布局、调用 3D 工具（检索 / 添加 / 旋转 / 平移 / 删除 3D 资产），并在每一轮结束后观察沙盒返回的多模态反馈（3D 地图文本 + 五视角渲染图），如此循环，直到最终产出一个完整的交互式 3D 世界。 ^[raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了.md]

**这套框架包含两大组件：**

  * **VWE-Bench（VibeWorlding Evaluation Benchmark）：** 一个覆盖 2,616 个高质量 3D 资产、323 个人工标注种子世界、6,828 条逆向合成用户查询的评测基准，同时区分「有标准答案可验证」与「开放式需靠规则打分」两类任务。

  * **VibeWorlding-Gym：** 一套联合多模态强化学习后训练框架，把 3D 资产检索、编辑、渲染统一封装为 MCP 工具，并配套一个「双重约束验证器」，同时支撑公平评测和可扩展的 RL 奖励计算。




#### ▍ 6,828 条查询是怎么「反向」造出来的？

VWE-Bench 的构建走的是「人机协作」路线，分三步走：

  * **第一步，高质量 3D 资产合成。** 美术标注员先确定 3,148 个概念 3D 资产的名称与类别，用 Gemini 3.1-flash-image 生成参考图，再用腾讯混元 3D（Hunyuan3D 3.1）把图片转成 3D 网格，经质量过滤和真实尺度标注后，最终沉淀 2,616 个可检索、跨越 20 个语义类别的高质量 3D 资产。

  * **第二步，种子世界标注。** 专业美术用这批 3D 资产手工搭建了 323 个「粗糙但功能完整」的种子 3D 世界，资产数量从 8 个到 258 个不等，覆盖不同复杂度。

  * **第三步，逆向查询合成。** 让 MLLM「倒着看」种子世界——既可以读一个世界然后写出「如何从零构建它」的描述（对应从零构建任务），

→ [[raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了|原文存档]] ^[raw/articles/从vibe-coding到vibe-worldingai开始自己造世界了.md]