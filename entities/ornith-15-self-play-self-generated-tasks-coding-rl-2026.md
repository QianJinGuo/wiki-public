---
title: "Ornith-1.5：模型自出题、自搭脚手架、自跑轨迹的编码 RL 闭环"
created: 2026-08-27
updated: 2026-10-06
type: entity
tags: [rl, self-play, post-training, coding-agent, grpo, curriculum-learning, synthetic-data, open-weights]
sources: [raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Ornith-1.5：模型自出题、自搭脚手架、自跑轨迹的编码 RL 闭环

> 新智元 2026-08-27 报道。Ornith 于 2026-08 一次性放出 397B MoE / 35B-A3B MoE / 9B Dense 三档开放权重，训练流程引入"模型自己生成训练任务 + 自己搭解题脚手架 + 自己跑解题轨迹，三段一起丢进强化学习"的自我出题闭环。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

## 核心创新：出题、搭台、解题三段闭环

Ornith-1.0（2026-06）已把 agent 脚手架（AI 解题时的工作台：工具怎么给、任务怎么拆、出错怎么重试）变成可学习的对象——每步强化学习先根据任务和上一轮用过的脚手架改出一版新的，再在这版脚手架上跑解题轨迹，奖励回传两段。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

1.5 把"题目从哪来"这一环也接了过去：给定代码库 + 任务类型高层说明 + 模型过去解题记录，系统生成一批新题，专挑比它已做出来的更难一档、戳能力缺口；题目定下后模型再为它生成或改进专属脚手架（指令、工具、拆解策略、编排逻辑）；最后在任务和脚手架双重条件下跑出解题轨迹，奖励从轨迹反向回传全部三段，统一用 GRPO 优化。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

## 防"送分题"机制：任务奖励三信号相乘

模型自我出题最大的破绽是"给自己出送分题刷分数"。Ornith 把任务奖励拆成三个信号并**相乘**——任何一项接近零，整道题奖励归零：^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

- **有效性**：脚手架跑不跑得起来、高置信度正确解能否通过、明显错误解是否被判失败、判分逻辑与任务描述是否对得上。当硬门槛，不合格直接判零，专拦"看着很难其实判不出对错"的废题。
- **前沿难度**：对每道题采样若干条轨迹算出经验成功率，越接近 0.2 目标值奖励越高——"一道好题模型十次应做成两次"。做成八次说明已会没有训练价值；一次都做不成则连一条可学成功轨迹都攒不出。难度靶子随模型能力自动爬升，无需人工调参。
- **新颖性**：权重最低，职责是去冗余、别老出同一道题变体，并非奖励怪题。

整套"自我提升"实质是**课程（curriculum）的自动演进**，且只发生在训练阶段——从 Hugging Face 下载的权重是死的，不会在设备上边干活边改自己。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

## 评测成绩与"两把尺子"的严谨辨析

自测 Terminal-Bench 2.1：1.0 为 77.5，1.5 的 397B 跑到 86.1（两个月涨 8.6 个百分点，官方注明取五次独立运行平均值）。但文章重点辨析了自测与官方验证榜的不可比性：^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

- 官方验证榜（截至 2026-08-19 共 17 项）里**没有 Ornith-1.5**——该项目自测成绩未上官方榜。
- Ornith 表里的 Opus 4.8 为 85.0，是他们用 Terminus-2 框架自己跑的；而官方榜上 Anthropic 提交的 Claude Code + Opus 4.8 经核验是 78.9%（第 5），榜首是 Claude Code + Fable 5 的 83.8%。同一个 Opus 4.8 两张表相差 6.1 分。
- 榜单规则明确"不得修改超时或资源配置"；Ornith 自测配置为 4 小时超时、32 核 CPU、48GB 内存，且"还动了配置，包括官方点名锁死的超时和资源"——项目自测与官方验证榜不能放进同一个排名读。
- 许可证：MIT 标的是权重；GitHub 仓库目前主要是 README/LICENSE/展示素材，未公开 1.5 完整训练实现、训练任务集或可复现评测脚本。"开放权重 ≠ 全套开源"。

## 中小尺寸档位的工程意义

- **35B-A3B MoE**（每 token 激活约 3B）：Terminal-Bench 2.1 跑 67.8，对比 Gemma 4-31B 的 42.1、Muse Glimmer-30B 的 51.7；SWE-bench Verified 79.0 对 52.0。3B 激活打赢 31B 稠密模型，是发布里最划算的一档。
- **9B Dense**（冲端侧）：自测 Terminal-Bench 46.2、SWE-bench Verified 70.6、GPQA Diamond 86.4；HF 5.63GB 压缩版 + 苹果 MLX 格式已放出。但非全面压过 Gemma 4-31B——MCP-Atlas 54.2 对 55.0、Toolathlon-Verified 41.2 对 52.8。"编码和推理能靠训练方法把体量差补回来，工具调用补不回来。"^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

## 行业意义：开放模型追赶闭源旗舰的第三条路

过去开放模型追赶闭源旗舰基本靠"堆参数 + 抄配方"两条路（照别人的卷子抄答案）。Ornith-1.5 提供第三条路：**自己造题**。高质量智能体训练任务一直是稀缺品，人工整理速度跟不上模型吃题速度。谁能量产好题，谁就握着持续变强的燃料。这也引出开放问题：当出题、搭台、解题都交给模型，人类留在这个闭环里的位置是什么？^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

## 深度分析

### 三信号相乘为何是对"奖励黑客"的结构性防御

把三个信号**相乘**而不是加权求和，是这个设计里最容易被低估的一步。加权和下，模型仍可把权重低的新颖性压到零、把前沿难度刷满，总奖励照样上涨——这正是自我出题范式最怕的"漂移"。乘法把每项都变成了乘数：任何一项趋零，整题归零，模型没有"用一项换另一项"的套利空间。这本质上是用奖励函数的代数结构替掉了人工审计——不需要人盯着题库查废题，投机路径在数学上被封死。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

### 0.2 成功率靶子的双重身份

前沿难度信号把"经验成功率最接近 0.2"设为最优，这实际是教育心理学里"最近发展区"（zone of proximal development）的 RL 版本：任务恰好悬在"会与不会之间"，每一次成功都携带最大梯度信息。妙处在于它同时是两个东西——既是单题的质量过滤器（全对或全错的题都没有训练价值），又是全课程的自适应调度器（某类题做熟了、成功率上移，出题端被迫加码，难度曲线自动爬升）。人工课程设计里最难调和的"难度设定"与"进度推进"被同一个标量信号统一了。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

### 与其他自博弈范式的分工对比

与 [[entities/qwen-skill-self-play-hyman-2026|阿里 Qwen Skill-SP 自博弈]] 和 [[entities/searchmaster-grounded-regulated-self-play-jd-2026|SearchMaster 接地自博弈]] 相比，Ornith-1.5 的差异化在于它把"任务生产"本身变成了可优化对象：Skill-SP 的自博弈发生在固定任务的对手之间，SearchMaster 的自博弈围绕搜索轨迹，而 Ornith 让出题端与解题端同时接受 [[concepts/grpo-policy-optimization-2026|GRPO 策略优化]] 的梯度。代价也藏在这一点里——三个可学习段互相耦合，任何一段塌掉（出题端退化、脚手架失效）都会污染另外两段的训练信号，系统比单段自博弈更脆弱、更依赖三信号护栏的持续有效。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

### 可验证性的边界：这套叙事有多少可复核

对照"两把尺子"的辨析，深度分析必须指出这套闭环的实证基础目前不可独立复核：训练实现、任务集、评测脚本均未公开，86.1 是自测而非官方榜核验值。三段闭环与三信号机制作为**设计描述**是自洽的，但"两个月涨 8.6 个百分点究竟多少归功于自我出题、多少归功于更多算力或更长训练"无法从公开材料分解。对采用者而言，这意味着复现风险集中在最核心的一环——恰恰是那个没有开源的出题机制。这也是 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 可验证推理]] 社区反复强调的：没有可复现的判分环境，课程自动演进就是一句无法审计的承诺。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

### 人类留在闭环里的位置

出题、搭台、解题三段全部可学习之后，人类角色从"内容生产者"退到了"结构设定者"：设定有效性硬门槛的判分逻辑、固定 0.2 这个难度靶子、决定新颖性权重——所有元参数仍然人工锚定。这套机制与其说取消了人类，不如说把人的判断压缩进了奖励函数的定义层，一次设定、长期生效。真正的风险不在闭环内，而在闭环外：自动演进的课程只会追逐模型当前能力缺口能表达的题目，模型"不知道自己不知道什么"的盲区，三信号里没有任何一项负责照亮。^[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026.md]

## 相关实体与概念

- [[entities/qwen-skill-self-play-hyman-2026|阿里 Qwen Skill-SP 自博弈]] — 同属 self-play 训练范式
- [[entities/searchmaster-grounded-regulated-self-play-jd-2026|SearchMaster 接地自博弈]] — 同属自博弈搜索 Agent 训练
- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 框架实践]]
- [[concepts/grpo-policy-optimization-2026|GRPO 策略优化]]
- [[concepts/reinforcement-fine-tuning-rft|强化微调 RFT]]
- 合成数据生成
- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR 可验证推理]]

→ [[raw/articles/ornith-15-self-play-self-generated-tasks-coding-rl-2026|原文存档]]
