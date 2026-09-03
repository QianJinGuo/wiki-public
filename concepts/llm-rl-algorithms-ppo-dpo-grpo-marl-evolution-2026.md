---
title: "LLM RL 算法演进图谱：从 PPO 到 DPO 到 GRPO 再到 MARL（2026 综述）"
created: 2026-06-05
updated: 2026-08-29
type: concept
tags: [rlhf, ppo, dpo, grpo, rlvr, marl, agent-rl, alignment, deepseek, prm, orm, agent-lightning, agent-r1, verl, openrlhf, trl, reinforcement-learning, instruction-tuning, r1, rlaif, rft, post-training]
sources:
  - raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl
related:
  - "[[concepts/reinforcement-fine-tuning-rft|Reinforcement Fine-Tuning (RFT)]]"
  - "[[concepts/agent-backend-unification|Anthropic Managed Agents 架构：脑手分离设计]]"
  - "[[concepts/ahe-agentic-harness-engineering|Agentic Harness Engineering (AHE)]]"
  - "[[concepts/llm-pretraining-vs-sft|LLM Pretraining vs SFT]]"
  - "[[concepts/catastrophic-forgetting|Catastrophic Forgetting]]"
  - "[[concepts/scaling-laws|Scaling Laws]]"
  - "[[entities/agentcore-harness|AgentCore Harness]]"
confidence: 0.92
provenance_state: extracted
summary: 2026 年 LLM 后训练 RL 算法完整演进图谱：PPO+RLHF→DPO→GRPO+RLVR→MARL 的算法演进去重史、PRM/ORM 信用分配之争、Agent-Lightning/Agent-R1/verl 三大智能体训练框架对比、以及 2026 年面向前沿模型的后训练栈配方。
description: 基于 DeepHub IMBA 综述（原作者 Shivam Agrahari）的算法演进全景图，覆盖 PPO/DPO/GRPO 三个里程碑算法的核心思想、局限性、典型场景、PRM/ORM 之争、MARL 信用分配三杠杆、Agent-Lightning vs Agent-R1 两条训练路径、verl/OpenRLHF/TRL 训练引擎与 2026 后训练栈配方。
---

# LLM RL 算法演进图谱：从 PPO 到 DPO 到 GRPO 再到 MARL（2026 综述）

> 本文是 [[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|DeepHub IMBA 综述]] 的算法合成页。它将 LLM 强化学习五年的演化压缩成一条"删除史"——TRPO 删脆弱性、PPO 删二阶数学、DPO 删奖励模型、GRPO 删 critic、结果奖励删按步标注、Agent-Lightning/Agent-R1 删重写智能体、MARL 删静态环境。配合 [[concepts/reinforcement-fine-tuning-rft|RFT 概念]] 作为前置阅读（本文是其算法层的深度补全）。

## 一、核心论点：LLM 强化学习的 2026 全景

LLM 强化学习过去五年的演化可以浓缩为"一连串的删除"——每一代算法把前一代昂贵的部分剥离掉。2026 年训练一个前沿开源模型的合理后训练栈是分层组合的：**SFT → DPO 风格/拒答 → GRPO + RLVR → 智能体 RL 框架**；每一层解决不同的问题，对应不同的奖励来源。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

**关键转变**：奖励信号的来源从"人类标注"（RLHF）迁移到"可校验正确"（RLVR）——"单元测试通过"、"\\boxed{} 与标准答案一致"、"SQL 返回正确的行"成为奖励信号的标准载体。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

## 二、四个时代：算法与奖励信号的同步演化

| 时代 | 主流算法 | 奖励来源 | 捕捉的信号 | 主要失败模式 | 瓶颈 |
|------|---------|---------|-----------|------------|------|
| 2022–2023 | PPO + RLHF | 人类成对偏好训出的 RM | 人类会更喜欢哪个答案 | 谄媚、对 RM 钻空子 | 人类标注者 |
| 2023–2024 | DPO | 直接作用在偏好对 | 同上，去掉 RM | 不探索，被数据集边界框住 | 偏好数据质量 |
| 2024–2026 | GRPO + RLVR | 验证器（测试运行器/数学校验器/正则/judge） | 答案是否可被证明正确 | 对验证器钻空子、能力隧道化 | 验证器设计 |
| 2025 至今 | GRPO + LLM-judge | 更强模型作为 judge | 在更聪明模型看来是否正确 | judge 偏见、模式坍塌 | judge 校准 |

> **关键洞察**：算法变廉价的速度（DPO 去掉 RM、GRPO 去掉 critic）远快于奖励信号变精确的速度（人类偏好 → 可验证 → LLM judge）。这也是为什么"验证器工程"会成为 2026 年最稀缺的能力之一。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]


## 三、PPO+RLHF：一切的开始

### 3.1 标准流程（SFT → RM → PPO）

1. **SFT**——用一小批人类撰写的示范数据微调基础模型。
2. **奖励模型（RM）**——给标注者看两组模型输出，问哪一个更好，再训练一个奖励模型 r(x, y) 来预测这种偏好。
3. **PPO**——把奖励模型当作环境，从策略中采样回复，用 RM 打分，再用 PPO 更新策略；加一项相对 SFT 策略的 KL 惩罚。

策略目标包含期望奖励减去 β·KL 散度惩罚。β 是最常调的那个旋钮。

### 3.2 为什么它有效

偏好数据比示范数据好收集得多。问标注者"这两个里哪个更好"很便宜；让他从头写出最完美的回答则不便宜。InstructGPT 论文的著名结论今天仍然成立：一个 1.3B 的 PPO 微调模型被偏好的程度高于 175B 的基础 GPT-3。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

### 3.3 四大实操痛点

- **显存负担**——四个模型同时挂在显存里（策略/参考/RM/critic），70B 策略约等于 280B 参数总量。
- **奖励黑客**——策略会找出 RM 的每一处弱点（长答案/项目符号/Markdown 标题）。
- **分布漂移**——RM 是在原始 SFT 输出上训练的，策略前移时 RM 越来越不可靠，但 loss 曲线看不出。
- **超参数脆弱**——裁剪比、KL 系数、价值损失权重、学习率、group size、rollout batch size，任一调错，训练悄无声息地劣化。

### 3.4 PPO 没死

ICML 2024 的 *Is DPO Superior to PPO for LLM Alignment?* 研究表明：在数据质量保持一致时，**PPO 在数学任务上仍然比 DPO 高约 2.5%，在通用 benchmark 上高 1.2%**。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

**PPO 仍是正确答案的场景**：
- 需要**探索**（数学、代码、长程推理），而不只是**模仿**偏好。
- 拥有高质量、稳健的奖励模型或可信的验证器。
- GPU 预算够把四个模型同时挂在显存里。

## 四、DPO：把奖励模型删掉

### 4.1 核心技巧

2023 年 Rafailov 等人的核心观察：在标准 RLHF 假设下（Bradley–Terry 偏好模型 + KL 正则），最优策略与隐式奖励函数之间存在闭式关系。因此可以**先学 RM 再 PPO** 两步合并成一步——直接定义在偏好对 (x, y_w, y_l) 上的监督损失，本质是 log σ(被选与被拒回复的 β·对数概率比之差) 的交叉熵。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

没有奖励模型，没有 rollout，没有 critic，没有 PPO 循环。

### 4.2 DPO 的甜点

- **便宜**——同样的数据，算力开销通常比 PPO 少 2–4 倍。
- **稳**——本质上是监督损失，画曲线看着它往下走就行。
- **风格塑造**——拒答行为、语气、格式遵循、闲聊任务的有用性。
- **β 落在 0.1–0.5**——太低策略漂走，太高几乎不动。
- **多轮 DPO 效果远好于一次性**——基于最新策略重新采样偏好对。

### 4.3 DPO 的硬局限

**不做探索**。如果正确答案从未出现在数据集里，DPO 不会凭空发明出来。**在那些模型必须自己发现更优路径的任务上（数学、代码、智能体轨迹），DPO 很快就触顶了**。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

这个局限正是 GRPO 要填的那个空。


## 五、GRPO：把 critic 也一起删掉

### 5.1 核心机制

**Group Relative Policy Optimization（GRPO）**，由 DeepSeek 提出、被 DeepSeek-R1 推到聚光灯下：根本不需要学习出来的价值函数，直接用**一组样本本身**作为基线。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

对单个 prompt x：
1. 从当前策略中采样一组 G 个 rollout：y_1, ..., y_G（典型 G = 8–64）
2. 用奖励函数对每个 rollout 打分 r_i = R(x, y_i)
3. 计算组内归一化的优势：A_i = (r_i − mean(r)) / std(r)
4. 代入 PPO 风格的 clipped 目标 + KL 惩罚

**本质**：把 PPO 里的 value network 换成"同一个 prompt 下，组里其它样本的表现"。

### 5.2 关键优势

- **没有 critic → 显存减半**——7B 训练从 16 张 H100 降到 8 张 H100。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]
- **天然契合可校验奖励的对比性**——二值奖励（正确/错误）经组内白化后产生干净的对比信号。
- **优势稳定**——组归一化消解了大部分 reward scale 麻烦。
- **配推理任务很顺**——长链式思维 + 大 G + 好的验证器 = 2025–2026 绝大多数开源推理模型背后方式（DeepSeek-R1、Qwen、OLMo 3 等）。

### 5.3 GRPO 的常见坑

- **组大小 G**——越大优势方差越低，但 rollout 数线性增加。公开参数大多 G = 16–32。
- **全零或全一的组**——标准差为 0，优势爆炸或消失。分母加 epsilon + 过滤退化 prompt。
- **β 调得太低**——策略会偏离连贯语言。DeepSeek 风格 β 落在 0.001–0.04。
- **奖励形态决定一切**——0/1 奖励与稠密奖励训出来差异巨大；过程奖励又是另一种。

### 5.4 GRPO 的变体

DAPO、GSPO、Dr. GRPO 等都是 GRPO 的小幅改进——核心思想没变：用一组 rollout 作为基线。

## 六、PRM vs ORM：信用分配之争

### 6.1 两种奖励形态

- **ORM（Outcome Reward Model）/ 终答式**——每个 rollout 一个标量挂在最终答案（单元测试通过、\\boxed{} 匹配、SQL 正确行）。
- **PRM（Process Reward Model）/ 步骤式**——每一步一个分（CoT 里每一步单独打分，由见过步骤级人类标注的分类器完成）。

### 6.2 PRM 的"高开低走"

直觉上 PRM 显然更好——长链式思维中只错一步也能得到干净、局部化的修正信号。2023 年 OpenAI *Let's Verify Step by Step* 在 PRM800K（数十万条数学步骤标注）上训的 PRM，在 MATH 类问题的 best-of-N 采样上以可观幅度领先 ORM。差不多一年里"PRM 就是方向"成为共识。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

接着 **DeepSeek-R1 落地，方法简单得让人不好意思**：基于规则的结果奖励 + GRPO，没有 PRM 没有步骤级标注。R1 论文说得很简单——他们试过 PRM，发现难以规模化（标注成本、奖励黑客、"一步"如何定义的模糊性），于是坚持用结果奖励。即便如此按这种方式训出的模型依然展现出了丰富的逐步推理。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

2025 年 *Is PRM Necessary?* 进一步证实：当前 PRM 应用到 DeepSeek-R1 和 QwQ-32B 这样的最前沿模型时，表现**实际不如简单的多数投票**。**纯结果式的 RL 看上去本就把步骤级的判别能力作为副产品诱导了出来。**

### 6.3 ORM + GRPO 为什么就够了

GRPO 对同一个 prompt 采样 G 个 rollout 并组内归一化。当 G 足够大、prompt 足够难时，自然会同时拿到成功和失败的 rollout。**组相对优势已经在做一种隐式的步骤级信用分配**——成功轨迹和失败轨迹会共享一部分早期决策，梯度会偏向地落在它们开始分叉的那些步骤上。

> **核心洞察**：用结果奖励的 GRPO 大致是在不为标注买单的前提下做过程监督。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

### 6.4 PRM 仍能胜出的四种情形

- **长程智能体任务**——40 步浏览-编码-重新规划工作流，"最终答案正确"过于稀疏。
- **工具使用的安全性**——惩罚特定中间行为（错误 API 调用、PII 泄露、不该改的文件），结果奖励来得太迟。
- **多智能体训练 MARL**——ORM + GRPO 那个信用分配小把戏在多智能体时悄悄失效。
- **推理时搜索**——PRM 在推理时仍有用，可指导 beam search / MCTS / best-of-N。

### 6.5 选择决策树

- **从结果奖励开始**——更便宜、更简单、更易调试。
- **在数学、代码、SQL 以及结构化任务上，结果奖励几乎总是足够**。
- **只有看到证据才加过程奖励**——看失败 rollout：结构正确却卡在特定中间错误 → PRM 可能有用；以各种方式失败 → 结果奖励 + 更多 rollout 更管用。
- **如果一定要做 PRM，优先 ThinkPRM 这样的生成式 / LLM-as-PRM 思路**，而不是 PRM800K 风格的判别式分类器（标注成本才是真正的杀手）。

> **重要澄清**："PRM vs ORM"与"稠密 vs 稀疏奖励"**不是同一根轴**。可有稀疏 ORM（二值正确/错误），也可有稠密 ORM（按部分得分细则分级）。**让 PRM 独特的是逐步打分，不是信号的稠密度**。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]


## 七、面向 LLM 的 MARL：当环境就是其它模型

### 7.1 三种 MARL 范式

1. **自我博弈做推理**——一个模型在带可校验结果的任务上，与不断进步的自我拷贝对弈。
   - **SPIRAL** 通过多轮零和博弈（井字棋、Kuhn Poker、简单谈判）训练单个 LLM，**8 个推理 benchmark 上最多 10% 提升**。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]
   - 概念上是把 AlphaZero 应用到语言上。
2. **协同进化的角色智能体**——多个专门化智能体各自有策略、一起训练。
   - **SAGE** 闭环跑 Challenger/Planner/Solver/Critic 四角色，**LiveCodeBench +8.9%、OlympiadBench +10.7%**。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]
   - Challenger 的真正新颖：系统在自己变强的过程中，**自己发明出自己的训练分布**。
3. **学习多智能体拓扑**——已有投票/辩论多智能体系统时，MARL 学习哪个智能体该跟哪个对话、什么时候对话。
   - **Agent Q-Mix** 在 CTDE 范式下把智能体通信视为合作型 MARL，用 QMIX 风格价值分解。**Humanity's Last Exam 上以 Gemini-3.1-Flash-Lite 报告 20.8%**——超过手工设计的 pipeline。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

### 7.2 MARL 真正的难题：信用分配

> "MARL 项目要么在这里成功，要么在这里悄悄崩掉。"

**单智能体 GRPO 那个信用分配小把戏（同一策略下、跨 rollout 的组内方差）在 MARL 时悄悄失效**。一个智能体团队共同产出一条轨迹，拿到一个团队级奖励——通常稀疏、常常是二值。问题随之变尖锐：^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

- 团队拿到 reward = 1，到底是谁的决定起了作用？
- 团队拿到 reward = 0，是规划者拆解得不好、求解者写的代码不对，还是批评者没抓到 bug？
- 如果均匀把梯度回推到每个智能体的 token 上，每个智能体拿到的信号都是嘈杂的，而且其中大部分内容其实关于的是其它智能体。

### 7.3 三种实用杠杆

| 杠杆 | 核心机制 | 适用场景 | 代表工作 |
|------|---------|---------|---------|
| **过程奖励（PRM 风格）** | 训验证器，单独给每个智能体贡献打分 | 失败模式与角色相关时杠杆率最高 | 标准 PRM 工程 |
| **价值分解（VDN/QMIX/COMA）** | 学联合价值函数，按智能体分解 | 角色稳定、贡献大致可加 | Agent Q-Mix |
| **轨迹分解（POMDP/分层）** | 把多步轨迹分解为 state-action-reward 转移，奖励沿轨迹图传播 | 工程侧、不训单独 PRM | Agent-Lightning LightningRL |

> **MARL 规则**：**纯结果奖励的 MARL 只在团队规模小（2–3 智能体）+ 轨迹短 + 足够多团队级 rollout 时才安全**。超出这个范围，必须在信用分配上投入。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]



## 八、智能体 RL 框架：Agent-Lightning vs Agent-R1

### 8.1 共同问题

到 2025 年，多数生产环境里的智能体都是用 LangChain / AutoGen / CrewAI / OpenAI Agent SDK / Microsoft Agent Framework 拼出来的。把这种智能体接到 GRPO 循环上通常意味着用训练友好的格式重写一遍——没人想做这件事。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

2024–2026 年浮现出一个小小的"智能体 RL"框架生态，专门解决这个问题。领先的两个在设计上做出了实质性不同的选择：

### 8.2 Agent-Lightning：框架无关、由可观测性驱动

微软研究院 2025 年 8 月开源（v0.3.0）。核心思想：完全不应该需要去改智能体代码，把智能体视作黑盒，通过可观测性钩子捕获交互，再把 trace 转成 trainer 可消费的标准 state-action-reward 转移。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

三个组件：
- **Algorithm**——决定要跑哪些任务、如何从结果中学习（支持 RL / APO / SFT）
- **Runner**——在任务上运行智能体，记录轨迹（你正在用的框架原封不动）
- **LightningStore**——共享存储和消息队列

算法层亮点：LightningRL 在多步轨迹上做**分层信用分配**（直接对应 MARL 信用分配问题线）。允许在多智能体系统里选择性优化某一个智能体，并在子策略之间混用 RL / APO / SFT。

适合：已有 LangChain/AutoGen/CrewAI/Agent Framework 代码库的团队。

### 8.3 Agent-R1：步级 MDP、端到端有主张

中科大开源（2026 年 3 月 v0.1.0，GitHub ~1.4k 星）。核心思想：智能体训练的最佳形态是把**每一步交互当作一等的 RL 转移**——拥有自己的 state、action、observation——而不是当作越长越大的 token 序列。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

Step-level MDP 抽象：每一步存自己的 prompt 和 response；下一个 observation 由环境决定而不是直接拼接 token；步骤之间的上下文可被截断、摘要、重写或增强。标准 RL 循环 obs → action → step → next_obs 干净映射到智能体训练。

两个关键设计：
- 原生支持过程奖励，配合 PRIME 启发式奖励归一化，过程奖励与结果奖励可干净组合（直接对应 MARL 信用分配问题线）。
- 自定义优化器路线——长出 PSPO（Proximal Sequence Policy Optimization），把 token 级优化与序列级智能体交互对齐。

已长出的智能体：PaperScout（学术论文搜索）、TableMind（工具增强的表格推理）、Cast-R1（智能体式时间序列预测）。

适合：从零构建工具型智能体的团队，希望对环境定义、步骤结构、奖励塑形完全控制。

### 8.4 共享训练引擎

| 框架 | 定位 |
|------|------|
| **verl**（字节火山引擎 RL） | 2025–2026 开源 RLHF/GRPO/智能体 RL 事实上的分布式训练骨干。Agent-R1 直接建在它上面 |
| **OpenRLHF** | 早期通用 RLHF 训练框架，今天仍广泛用于单策略训练 |
| **TRL**（Hugging Face） | 中小规模做 DPO 和 PPO 的主力工具，更像最先拿起来的那套微调工具箱 |

### 8.5 周边框架

- **RAGEN**——更早的智能体 RL 框架之一，在概念上影响 Agent-R1 和 Agent-Lightning
- **MARTI / MARTI-v2**——开放多智能体强化训练与推理框架，带树搜索增强 + 32K token 序列级 GSPO 损失
- **FlexMARL**——大规模 LLM-MARL 训练框架，异步 rollout-训练 pipeline **~7.3× 加速**
- **MARL-GPT**——StarCraft、GR Football、POGEMA 的 MARL 基础模型早期尝试

## 九、2026 后训练栈配方

每一阶段都是不同的问题，对应不同的奖励。请按这个方式来对待它们。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]

| 目标 | 训练配方 | 原因 |
|------|---------|------|
| 基础模型礼貌地遵循指令 | SFT → DPO | 便宜、稳定，正好契合风格与语气 |
| 加入拒答或安全行为 | DPO | 偏好对是天然格式 |
| 在数学/代码/逻辑上推动推理 | GRPO + RLVR + 结果奖励 | 可校验终答奖励 + 组基线，压倒一切 |
| 端到端训工具型智能体 | 在智能体轨迹上做 GRPO，框架二选一（Agent-Lightning / Agent-R1） | 长轨迹 + 模糊"最终答案"需要步骤级信号 |
| 用不起 critic 但想要 PPO 探索 | GRPO | 同样探索能力，显存少一半 |
| 非常好的 RM + 大 GPU 预算 | PPO | 硬任务仍最好 |
| 多角色工作流（规划者/求解者/批评者） | 先单智能体饱和再上 MARL；MARL 时必须预算按步/按智能体奖励 | 团队级结果奖励在多智能体间得不到可训练梯度 |

## 十、未来发展（2026–2027）

- RLHF 不会死——它会变成一层薄薄的专门化层（风格、语气、品牌口吻、拒答行为）。其余都在向可校验迁移。
- 验证器工程成为独立学科——Sandbox 工程师、judge 设计师、评分细则校准者。**验证器是新的数据集**。
- 语言版 AlphaZero 真正到来——强基础 + 自我对弈 + 验证器 + 树搜索是配方，前沿实验室明显都在朝它收敛。
- 长程智能体 RL 是下一次飞跃——多日运行的智能体，会浏览、写代码、做实验、再修订，在完整轨迹上用 RLVR 训练。智能体 RL 框架层让前沿实验室之外的人也能做。
- 开源栈持续缩小差距——TRL、OpenRLHF、verl、Open-Instruct、Agent-Lightning、Agent-R1、RAGEN、MARTI、FlexMARL 让资金充足的开源团队能做的事与前沿实验室差距少见地小。
- 奖励黑客成为对齐里的中心问题——模型越来越聪明、验证器仍然不完美时，策略在边际上总会比信号更聪明。^[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl.md]



## 十一、与现有概念的关系

- **前置阅读** [[concepts/reinforcement-fine-tuning-rft|RFT 概念]]——覆盖 RLHF/RLVR/RLAIF 二级概览；本文是**算法层深度补全**（PPO/DPO/GRPO 演进 + ICML 2024 数据 + Agent-Lightning/Agent-R1 框架对比 + MARL 信用分配三杠杆）。
- **应用层关联** [[concepts/agent-backend-unification|Anthropic Managed Agents 架构]]——从工程/产品视角看智能体基础设施（Session/Harness/Sandbox），与本文算法训练视角互补。
- **训练基础设施** [[concepts/ahe-agentic-harness-engineering|Agentic Harness Engineering (AHE)]]——Harness 设计原则与本文的 RL 训练循环是同一问题的不同抽象层。
- **后训练全流程** [[concepts/llm-pretraining-vs-sft|LLM Pretraining vs SFT]]——本文专注后训练阶段的 RL 部分，与预训练/SFT 形成完整训练图谱。
- **遗忘问题** [[concepts/catastrophic-forgetting|Catastrophic Forgetting]]——RLHF/DPO/GRPO 后训练一个常见副作用是灾难性遗忘；本文未展开但在遗忘概念中有相关讨论。
- **规模化原理** [[concepts/scaling-laws|Scaling Laws]]——验证器工程成为独立学科的趋势，与"数据和算力是唯一瓶颈"的传统 scaling law 视角形成有趣对照。
- **AWS 实践** [[entities/agentcore-harness|AgentCore Harness]]——AWS Bedrock AgentCore 与本文 verl/OpenRLHF/TRL 是同一生态系统的不同实现。

## 相关实体

- [[entities/aws-sagemaker-sft-dpo-tool-calling]]
- [[entities/agentic-rl-token-in-token-out]]
- [[entities/alphaxiv-reinforcement-learning-for-rlms]]
- [[entities/deepseek-v4-training-methodology]]

## 十二、独家数据点速查

| 数据点 | 来源论文/工作 | 数值 |
|-------|--------------|------|
| PPO vs DPO 数学任务优势 | ICML 2024 *Is DPO Superior to PPO for LLM Alignment?* | PPO 高 2.5% |
| PPO vs DPO 通用 benchmark 优势 | 同上 | PPO 高 1.2% |
| InstructGPT 1.3B PPO vs 175B 基础 | OpenAI InstructGPT 论文 | 偏好程度胜出 |
| DPO 算力节省 | DeepHub IMBA 综述 | 比 PPO 少 2–4× |
| GRPO 显存节省 | 同上 | 比 PPO 少一半（7B 训练：8 H100 vs 16 H100）|
| DeepSeek GRPO β 范围 | DeepSeek 训练实践 | 0.001–0.04 |
| DPO β 典型范围 | 综述经验 | 0.1–0.5 |
| GRPO G 大小 | 公开参数 | 16–32 |
| SPIRAL 自我博弈提升 | SPIRAL 论文 | 8 个推理 benchmark 最多 +10% |
| SAGE 协同进化 | SAGE 论文 | LiveCodeBench +8.9% / OlympiadBench +10.7% |
| Agent Q-Mix | Agent Q-Mix 论文 | Humanity's Last Exam + Gemini-3.1-Flash-Lite = 20.8% |
| FlexMARL 加速 | FlexMARL 论文 | 异步 pipeline 约 7.3× |
| Agent-R1 GitHub 星 | 综述记录（2026-03） | ~1.4k |
| Agent-Lightning 版本 | 综述记录（2025-08） | v0.3.0 |
| Agent-R1 版本 | 综述记录（2026-03） | v0.1.0 |

> **置信度** confidence: 0.92——原作者 Shivam Agrahari 完整署名 + DeepHub IMBA 译本 + 8500 字完整覆盖 + 引用 6+ 篇具体论文（ICML 2024 / OpenAI Let's Verify / R1 / Is PRM Necessary / SPIRAL / SAGE / Agent Q-Mix / FlexMARL）。
> **provenance_state**: extracted（事实性算法综述，无合并/推断成分）。

## 关联实体

**上游依赖**:
- [[entities/agentcore-harness]] — 提供基础理论/方法
- [[entities/agentcore-harness]] — 提供基础理论/方法

**下游应用**:
- [[entities/aws-sagemaker-sft-dpo-tool-calling]] — 具体应用场景
- [[entities/agentic-rl-token-in-token-out]] — 具体应用场景
- [[entities/alphaxiv-reinforcement-learning-for-rlms]] — 具体应用场景

**平行协作**:
- [[entities/deepseek-v4-training-methodology]] — 替代/补充方案
- [[entities/deepseek-v4深度拆解一篇论文同时做了五件大事]] — 替代/补充方案
- [[entities/trajectory-balance-asynchrony-tba-bengio-papweekly]] — 替代/补充方案


→ [[raw/articles/2026-llm-rl-algorithms-deeplog-imba-ppo-dpo-grpo-marl|原文存档]]

## 新增关联实体
- [[entities/deepseek-v4深度拆解一篇论文同时做了五件大事]]
- [[entities/trajectory-balance-asynchrony-tba-bengio-papweekly]]

## 所属 MOC

- [[moc/llm-core-technology|Llm Core Technology]]
