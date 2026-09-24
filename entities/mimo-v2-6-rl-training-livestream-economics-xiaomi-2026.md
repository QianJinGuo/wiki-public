---
title: "小米 MiMo-V2.6 强化学习直播：RL 扩展的三条轴与「每 1 个百分点多少钱」"
created: 2026-09-17
updated: 2026-09-24
type: entity
tags: [mimo, xiaomi, reinforcement-learning, post-training, agentic-rl, training-economics, data-mixture, rl-scaling]
sources: [raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026, raw/articles/xiaomi-mimo-v26扩展强化学习规模迈向自我提升, raw/articles/mimo-v26-shixiaoyao-hands-on-review-2026]
confidence: 0.8
provenance_state: extracted
---

# 小米 MiMo-V2.6 强化学习直播：RL 扩展的三条轴与「每 1 个百分点多少钱」

> **Background**：本文基于夕小瑶科技说 2026-09-17 报道提炼。小米 MiMo 大模型负责人罗福莉公开把 MiMo-V2.6 的强化学习（RL）训练过程做成实时直播（训练仪表盘 `mimo.xiaomi.com/rl`，数据直连训练器日志），本文提取其中可迁移的 RL 训练工程信息：扩展轴、单步成本结构、数据配比，以及「能力提升的定价」这一此前少有公开的度量。

## 直播披露的核心问题：RL 到底还能扩展到多远

小米把这一轮训练的问题设定为一个公开实验：**强化学习到底还能扩展到多远**。被训练的是 MiMo-V2.6 的两个版本（MiMo-V2.6-Pro 与 MiMo-V2.6-Flash），两者在同一个仪表盘上并行跑训练线，页面上的花费、步数、token 数、数据配比与评测成绩实时变化。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

团队称本轮在三个维度做了扩展：**算力**（每步约 20 亿 token，1568 个提示词 × 16 次推演，完全异步）、**环境与工具框架**（多任务智能体强化学习，单次运行混合多个框架）、**评分器算力**（智能体组内学分分配，奖励基于测试用例与评分细则）。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

从训练工程角度看，这三条轴对应的是 agentic RL 的三个已知约束面——rollout 吞吐、环境异质性、奖励计算成本；把它们同时放大并公开读数，是这篇报道最接近一手数据的部分。

## 单步成本结构：模型答题与更新参数各自计价

一个「Step」的机制是：模型被放进带代码库、终端和测试用例的环境里，自己改代码、跑测试，逐题打分；一批任务全部做完后汇总分数、更新一次参数。每一步模型把 1568 道题各做 16 遍，产生约 2.5 万条答题记录，由此一步的开销被拆成三块——**模型答题、评分器打分、更新参数**。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

成本读数：Pro 每步约 **7.4 万美元**、Flash 每步约 **2 万美元**；Pro 一步耗时构成为模型答题 58 分钟 + 更新参数 64 分钟，合计 2 小时 06 分。截至报道时两条训练线累计花费超过 **113 万美元**（约合报道标题的「两天烧掉 800 万」人民币量级）。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

给两个对照锚点：DeepSeek-R1 的 RL 阶段花费 29.4 万美元、用 512 张 H800；MiniMax-M1 的 RL 阶段花费 53.5 万美元、同样 512 张 H800 跑了三周。小米两条线两天花掉 113 万美元，比这两家加起来还多。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

报道同时给出一个反直觉的成本归因：贵并不完全因为卡用得多，**主要贵在题型**——一道题平均 55 轮工具调用、9 万 token 上下文，一步要做 2.5 万道，「光是让模型把题做完，就比对答案贵得多」。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

## 数据配比：代码 2/3、视觉 13.1%、对话 3%

仪表盘把每一步喂入的 25 个数据集按类别列出，这是国内厂商首次公开强化学习阶段的数据配比：**代码占三分之二，视觉占 13.1%，对话只占 3%**。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

这一配比与训练任务形态一致——训练题目是真实软件工程任务（与 DeepSWE v1.1 的 113 道长程任务同类），因此代码类环境占绝对多数，而通用对话数据被压到很低比例。对做 agentic RL 的团队，这提供了一个可对照的「后训练数据配比」先例。

## 能力收益的定价：训练通过率每涨 1 个百分点多少钱

仪表盘公开了一个名为**训练通过率**的指标：当前步的 1568 道题、每题 16 次尝试中，模型平均有多少任务能通过，并同时显示相对第一步的涨幅。Pro 到第 11 步训练通过率 **61.4%**、比第 1 步提升 4.9 个百分点，即前 10 次更新平均一步涨约 0.5 个百分点；该段训练累计花费约 70 万美元，折算下来 **Pro 的训练通过率每提高 1 个百分点约花 14 万美元**。Flash 到第 16 步训练通过率 60.3%、比第一步提高 8.9 个百分点，平均一步涨约 0.6 个百分点，折算 **每 1 个百分点约 3.8 万美元**，约为 Pro 的四分之一，且增长速度还没有明显放缓。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

评测侧读数：Pro 训练到第 10 步得 63.72 分，追平 DeepSeek-V4-Pro；Flash 第 12 步得 60.77；在 DeepSWE v1.1 官方榜单（pass@1）上，MiMo-V2.6-Pro 离榜首差 12 分、离 Claude Fable 5 差 7 分。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

## 公开故障：显存问题与训练线重启

直播把训练故障也放进了通知栏，例如「Pro 训练因一个节点的显存问题正在重启」。对长跑 RL 训练而言，把「故障—重启—继续」的过程公开，本身是这套仪表盘设计的一部分：训练过程从「竣工照」变成了「工地监控」。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

## 可迁移判据

- **RL 成本要按「答题 / 评分 / 更新」三段拆账**，只看卡数会把成本归因错：这类任务的开销主要在 rollout 与评分（55 轮工具调用、9 万 token/题），而非参数更新。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]
- **「能力每涨 1 个百分点花多少钱」是可公开、可对照的度量**，比裸 benchmark 分数更能判定 RL 是否还在有效区间；Pro 与 Flash 的 14 万 vs 3.8 万美元/1pp 差异说明同代不同尺寸模型的 RL 性价比可以差 4 倍。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]
- **异步 rollout + 多任务混合环境 + 组内学分评分器**是本轮三维扩展的工程落点，与既有 agentic RL 框架讨论中的吞吐/环境/奖励三约束面一致，可与 [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 框架实践对比]]、[[entities/agentic-rl-seven-lessons-six-frameworks|七条经验/六框架]] 互读。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

## 与前序小米大模型工作的关系

把时间线往前拉，这是小米今年在大模型上的第四波动作：3 月匿名模型 Hunter Alpha 登顶 OpenRouter 调用榜后被认领为 MiMo-V2-Pro；4 月 27 日 MiMo-V2.5 系列以 MIT 协议开源（Pro 版总参数 1T、激活 42B、上下文 100 万 token），同日宣布三年投入超 600 亿元；6 月发布 UltraSpeed 版本（1T 参数输出速度推到每秒 1000 token）并公开后训练方法 MOPD 论文。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

其中推理侧的系统优化已有沉淀（见 [[entities/mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo-V2.5 推理系统全链路优化]]），编码 harness 侧的实践见 [[entities/mimo-code-xiaomi-coding-harness-2026|MiMo Code 编码 Harness]]；本页补充的是**后训练/RL 阶段的公开账本**这一此前缺位的一环。^[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026.md]

## 相关

- [[entities/graviton-optimize-agentic-rl-sandbox-architecture-cost|Agentic RL 沙箱架构与成本优化]]
- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 框架长程实践]]
- [[entities/mimo-code-xiaomi-coding-harness-2026|MiMo Code：编码 Harness]]
- [[entities/mimo-v2-5-inference-system-optimization-hybrid-swa|MiMo-V2.5 推理系统优化]]



## 第 2 来源 — 小米官方发布：MiMo-V2.6 正式开源（2026-09-22）

小米 2026-09-22 正式发布并开源 MiMo-V2.6 系列（Pro + Flash 两个原生全模态模型），官方将其定位为探索 RSI（递归自我改进）路径的关键一步：以可验证的复杂任务为基础，规模化扩展强化学习（RL）算力，让模型在持续的探索与反馈中不断拓展智能边界。^[raw/articles/xiaomi-mimo-v26扩展强化学习规模迈向自我提升.md]

- **成绩**：MiMo-V2.6-Pro 在 Artificial Analysis Intelligence Index（AA 综合智能指数）取得 46 分，超过 Kimi K3 和 Qwen3.8 Max，成为当前最强的开源模型^[raw/articles/xiaomi-mimo-v26扩展强化学习规模迈向自我提升.md]
- **RSI 路线声明**：官方明确「在一个智能容易被复制的时代，选择把算力投进真实环境，让模型在反馈中一次次试错、自己学会」——与第 1 来源直播数据（6 天 Live RL 训练、113 万+美元花费）互为印证：直播是过程，本篇是结果^[raw/articles/xiaomi-mimo-v26扩展强化学习规模迈向自我提升.md]
- 互补角度 3 条：(1) 官方 RSI 定位与 RL 算力扩展路线的正式声明；(2) AA 指数 46 分超越 Kimi K3/Qwen3.8 Max 的开源最强成绩单；(3) Pro/Flash 双版本原生全模态定位确认

→ [[raw/articles/xiaomi-mimo-v26扩展强化学习规模迈向自我提升|第 2 来源原文存档]]

## 第 3 来源：夕小瑶编辑部实测（2026-09-24 merge）

夕小瑶编辑部在 MiMo-V2.6-Pro/Flash 正式发布后做了任务级实测，与第 2 来源的"官方成绩单"互补：

- **基准确认（第三方口径一致）**：AA 智能指数 46 分、DeepSWE v1.1 超 Fable 5，Pro 版 RL Dashboard 终局 72.6 分——与第 2 来源官方数据吻合^[raw/articles/mimo-v26-shixiaoyao-hands-on-review-2026.md]
- **成本坐标（独家数据点）**：Menlo Ventures 投资人实测口径——MiMo-V2.6-Pro 使用成本约为 Kimi K3 的 1/15、GLM 5.3 的 1/6、DeepSeek V4 的 1/2，在"AA 智能指数 × 单任务成本"图上处于左上角最优象限^[raw/articles/mimo-v26-shixiaoyao-hands-on-review-2026.md]
- **架构评价（Raschka）**：GQA + 128-token 窗口 SWA 的经典组合，Sebastian Raschka 评价"simply the best (for now)"——进步主要来自数据与后训练，注意力变体只是效率优化，但这类优化能在训练/推理上省数百万美元^[raw/articles/mimo-v26-shixiaoyao-hands-on-review-2026.md]
- **实测复核（评审纪律）**：复杂 Coding 任务（千万级孪生素数 WebGL 渲染、USGS 实时地震 3D 观测台）编辑部做了人工正确性复核——交付的确实是真孪生素数、真实 API 数据接入，非 benchmark 榜单分^[raw/articles/mimo-v26-shixiaoyao-hands-on-review-2026.md]

→ [[raw/articles/mimo-v26-shixiaoyao-hands-on-review-2026|第 3 来源原文存档]]

→ [[raw/articles/mimo-v2-6-rl-training-livestream-economics-xiaomi-2026|原文存档]]
