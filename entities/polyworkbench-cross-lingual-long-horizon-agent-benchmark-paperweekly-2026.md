---

title: "PolyWorkBench：跨语言长程工作流 Agent 评测基准"
created: 2026-09-09
updated: 2026-09-09
type: entity
tags: [benchmark, long-horizon, cross-lingual, agent-eval, workflow]
sources: [raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026]
confidence: 0.7
---

# PolyWorkBench：跨语言长程工作流 Agent 评测基准

## 概述

PolyWorkBench 是由**北京交通大学与腾讯微信 AI** 研究团队提出的 Agent 评测基准，论文题为 *PolyWorkBench: Benchmarking LLM Agents for Cross-Lingual Long-Horizon Workflows*，arXiv:2607.06008（代码与榜单见 GitHub / 项目主页）。它聚焦的是跨语言、长时程的 Agent 工作流评测：与传统的"一道题单一语言"不同，PolyWorkBench 要求 Agent 在**指令语言（Instruction Language）≠ 证据语言（Evidence Language）≠ 输出语言（Output Language）** 的真实工作场景中，从头到尾完成一项完整工作。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

文章以一个典型企业场景切入：员工收到**韩文任务指令**，需要阅读**中文日志和英文技术文档**，调用数据库与监控工具进行分析，最终生成一份**中英双语报告**。这正是 PolyWorkBench 与既有多语言 benchmark 的关键区别——语言不再只是任务的输入/输出属性，而是贯穿 Agent 整个执行轨迹的一部分。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

## 任务构建与评估体系

PolyWorkBench 构建了 **67 个长程工作流任务**，全部来自真实工作场景、人工设计，并经翻译与计算机从业人员审计，覆盖五个典型领域：**Commerce（COM）、Knowledge（KNW）、Legal（LEG）、Localization（LOC）、Manufacturing（MFG）**，涉及跨境业务、市场分析、合同审查、软件本地化、质量管理、生产报告等场景，覆盖 **10 种语言**（英、中、日、韩、法、德、西、俄、越南、阿拉伯）。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

任务要求 Agent 在**多语言文档、Spreadsheet、Database、Browser、Code Editor** 等异构资源与工具之间进行迭代式推理，经历「理解任务 → 检索信息 → 多语言理解 → 规划 → 工具调用 → 信息整合 → 生成结果」的完整工作流，最终交付报告、表格、法律文档、翻译文档等结构化成果——而非"回答一道题"。平均每个任务约含 **3.4 个输入文件、2.3 种语言**。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

评估上，PolyWorkBench 没有简单采用单一 LLM Judge，而是建立**三层评价体系**：
- **Grade**：通过结构化规则评价任务完成程度（主要排名指标）；
- **Pytest**：对可执行结果进行自动化测试；
- **LLM-as-Judge**：进一步评价语义质量与自然语言表现（辅助诊断信号）。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

这一设计同时回答了"任务有没有完成、程序有没有跑通、结果在语义上是否合理"三个问题，且不要求唯一标准答案，允许 Agent 通过不同执行路径达成同一目标。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

## 核心发现：强模型 ≠ 强 Agent

实验在 67 个任务上测试了多个**模型 × Agent Harness** 的组合，表现最好的是 **Claude Opus 4.8 + ClaudeCode（Pass@1 = 92.3%）**，其次是 **GLM-5.2 + Codex（88.7%）**。但更值得关注的是：**同一个模型换一个 Harness，性能可能发生巨大变化**——Claude Opus 4.8 在四种 Harness 下 Pass@1 从 92.3% 一路降到 69.8%，差距达 22.5 个百分点。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

这说明 Agent 的最终能力不仅取决于底座模型，也与规划、工具调用与执行框架的协同密切相关。在长程任务中，**强模型 ≠ 强 Agent**——模型与 Harness 的搭配同样决定能否把事情做完。这一点与 [[roadmapbench-long-horizon-agentic-software-development]]、[[mirrorcode-long-horizon-benchmark-epoch-ai-metr]] 等长时程评测的观察相互印证：长程能力更多取决于执行编排而非单点模型能力。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

## 跨语言失败模式

若只看总分数，容易忽略 Agent 在不同领域与语言上的能力差异。在**领域维度**上，Knowledge／Legal／Manufacturing 等任务一些强模型可达 0.85～0.95 的 Grade，但 **Commerce** 上多个模型明显下降——因为该类任务常同时包含数值核对、跨货币计算、Spreadsheet 操作与严格格式约束，一个局部错误就可能导致整体交付失败。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

在**语言维度**上，较强 Agent 在十种语言间表现相对稳定，而部分中等模型在**俄语（RU）、西班牙语（ES）、德语（DE）** 上明显下降，固定模型最好与最差语言的差距可超 30 个 Grade points。多语言失败主要来自两类问题：
- **语言理解错误**：从源语言材料中误读数字、实体等关键信息，并把错误一路带入后续流程；
- **跨语言协同错误**：初始理解正确，但随着多轮工具调用与信息整合，源语言信息与最终目标语言输出逐渐偏移。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

这两种模式正是传统多语言问答 benchmark 难以捕捉的，也是 PolyWorkBench 的核心贡献。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

## 定位与展望

PolyWorkBench 以 67 个任务、5 个领域、10 种语言，将 LLM Agent 放入更接近真实企业环境的跨语言长程工作流中，结果显示当前 Agent 在此类环境中仍面临明显挑战——模型能力、Agent Harness、任务领域与语言都会影响最终完成质量。它把 Agent 评测从"会不会回答这个问题"推向"能不能在复杂、多语言、跨工具环境中把事情从头到尾做好"，属于 [[agent-evaluation-benchmark-frameworks]] 与 [[agentic-rl-frameworks-practices-long-horizon-wolfe-2026]] 所描述的 Agent 测评下一阶段。长程记忆与检索策略对其同样关键（类比 [[remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026]]），而评测宽度也呼应 [[perplexity-wandr-benchmark-research-agents-wide-deep-2026]] 对"广而深"测评的追求。 ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]

→ [[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026|原文存档]] ^[raw/articles/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026.md]