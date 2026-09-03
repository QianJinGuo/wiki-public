---

title: "不止AutoResearch！策略与Harness共进化，EvoTrainer跑通自主闭环"
created: 2026-09-01
updated: 2026-09-01
type: entity
tags: [ai, agent, harness, memory, mcp, multimodal]
sources: [raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环, raw/articles/evotrainer-co-evolving-llm-policies-training-harness-agentic-rl-2026]
confidence: 0.75
provenance_state: extracted
---

# 不止AutoResearch！策略与Harness共进化，EvoTrainer跑通自主闭环


# 不止AutoResearch！策略与Harness共进化，EvoTrainer跑通自主闭环

原创 让你更懂AI的 2026-08-31 13:51 北京

训练系统也开始自进化

## 

自动跑实验只是第一步。EvoTrainer 开始让 AI 自己分析 Reward、Rollout 和失败轨迹，再反过来升级训练策略与 Harness。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

EvoTrainer 是⼀种让⼤语⾔模型训练策略与 Training Harness 协同进化的⾃主训练⽅法。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

它不只是在 Agentic RL 中⾃动跑实验、调参数，更关键的是让系统能随着模型变强，持续提升发现、诊断和解决训练问题的能⼒。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

论文标题：

EvoTrainer: Co-Evolving LLM Policies and Training Harnesses for Autonomous Agentic Reinforcement Learning ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

论文地址：

<https://arxiv.org/abs/2606.03108>

代码地址：

<https://github.com/AlibabaResearch/DAMO-ConvAI/tree/main/EvoTrainer> ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

核⼼观点

传统⾃主训练中，Training Harness 往往被视为固定基础设施：Reward 定义、样本过滤、回测逻辑、诊断⽅式⼀旦设定，后续训练只是在这套静态框架内搜索更优参数或配⽅。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

但在⻓程、多步、强交互的 Agentic RL 任务中，这种假设并不成⽴。

随着策略不断变强，原有 Harness 的局限会逐步暴露：

  * 原先有效的 Reward 可能失去区分度；

  * 原先⾜够的评估⽅式可能⽆法识别投机⾏为；

  * 原先粗粒度的诊断逻辑难以解释新的失败模式；

  * 原先的样本筛选机制可能⽆法持续提供⾼信息量训练信号。




因此，真正的⾃主训练不应只优化模型参数，⽽应让训练策略与 Training Harness 处于同⼀个协同进化闭环中： ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

  * 策略变强，暴露新的瓶颈；

  * Harness 感知瓶颈，升级诊断与回测能⼒；

  * 新的 Harness 再反过来为策略提供更可靠的优化信号。




EvoTrainer 的核⼼贡献，就在于将这种“训练决策系统本身也要持续进化”的思想系统化，并将其落地到 Training-Time。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

相关⼯作

现有⼯作⼤致覆盖三条路线：

a. ⾃主训练：AI ⾃动提出⽅案并执⾏实验。

b. 推理侧 Harness 优化：优化模型推理时的⼯具、上下⽂和脚⼿架。

c. 优化 RL 算法：针对不同任务设计更合适的 RL 机制。

2.1. AutoResearch：⾃动化实验搜索 [1]

AutoResearch 证明了 Agent 已具备接管训练的能⼒，它通过“修改代码 → 启动训练 → 验证指标 → 决策保留/回滚”的闭环，实现了训练⽅案的⾃动探索。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

然⽽，这种范式的本质仍是“基于静态标尺的快速试错”。

Agent 仅将验证集分数作为唯⼀的筛选依据，⽽承载评估与诊断功能的 Harness 在整个过程中始终保持不变。这导致系统缺乏对训练过程的深层理解： ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

  * ⽆法甄别信号质量：⾼分可能源于数据泄露或评估漏洞，⽽⾮真实能⼒提升；

  * 浪费失败样本价值：低分分⽀被直接丢弃，其中蕴含的诊断线索未被提取；

  * 缺乏能⼒沉淀：每⼀轮搜索都是独⽴的，训练中形成的排查经验⽆法转化为可复⽤的诊断技能。




换⾔之，AutoResearch 是⼀个⾼效的“实验搜索器”，但并⾮⼀个能持续提升训练理解能⼒的 “进化系统”。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

它优化的是“找到下⼀个⾼分配置”的效率，⽽我们关注的是“升级发现问题与定义问题”的能⼒本身。

2.2. 推理侧 Harness 演化：Meta-Harness [2] 与 AHE [3]

Meta-Harness 与 AHE 共同确⽴了 “Harness 本身可作为优化对象”的重要范式，为 EvoTrainer 提供了关键的⽅法论基础。Meta-Harness 率先打破了 “Harness 即固定基础设施”的传统认知，通过 Agent 搜索并修改推理阶段的上下⽂组织与⼯具链代码，利⽤历史执⾏轨迹持续提升下游任务表现；AHE 则进⼀步系统化了这⼀⽅向，将 Middleware、⻓期记忆与 System Prompt 纳⼊可编辑范围，并明确引⼊可观测性信号作为改动有效性的验证依据，证明了⻓期⾃动改进必须依赖过程证据⽽⾮仅看最终结果。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

然⽽，这两项⼯作的演化边界均严格限定在 Inference-Time。当我们将视⻆转向 Agentic RL 的训练场景时，会发现推理侧的演化逻辑存在根本性的信号断层： ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

  * 优化⽬标错位：推理侧追求单次任务完成率，⽽训练 Harness 需保障⻓期学习信号的稳定性与区分度；

  * 诊断维度缺失：推理侧关注输出正确性，却⽆法感知 Reward 失效、Rollout ⾏为崩塌或低信息量样本等训练动⼒学问题；

  * 经验⽆法跨阶段迁移：推理侧积累的 Prompt 优化或执⾏链路调试经验，难以转化为训练过程中对异常信号的审计、过滤与回测能⼒。




换⾔之，现有⼯作证明了 Harness 演化的可⾏性与过程证据的价值，但仍未跨越从“推理执⾏优化”到“训练策略协同进化”的鸿沟。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

EvoTrainer 正是要将这种“过程感知 + 系统演化”的思想，真正移植到 Training-Time Harness 中，使其成为与训练策略双向驱动的协同进化对象。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

⽅案

现有⼯作分别验证了 Agent 参与训练（AutoResearch）、Harness 动态演化（MetaHarness）以及过程证据驱动改进（AHE）的可⾏性，但尚未将这三者在 Agentic RL 的训练场景中统⼀。 ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

EvoTrainer 的核⼼出发点，

→ [[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环|原文存档]] ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]

→ [[raw/articles/evotrainer-co-evolving-llm-policies-training-harness-agentic-rl-2026|第 2 来源原文]] ^[raw/articles/不止autoresearch策略与harness共进化evotrainer跑通自主闭环.md]