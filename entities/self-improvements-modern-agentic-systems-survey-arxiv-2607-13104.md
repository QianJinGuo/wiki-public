---
title: "Self-Improvements in Modern Agentic Systems: A Survey — Agent 自我改进综述"
created: 2026-07-27
updated: 2026-10-21
type: entity
tags: [survey, self-improvement, agent, harness, scaffolding, foundation-model, memory, tool, evaluation, schmidhuber]
sources: [raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104, raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25]
confidence: 0.85
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Self-Improvements in Modern Agentic Systems: A Survey — Agent 自我改进综述

KAUST + 吉林大学 + Jürgen Schmidhuber 团队于 2026 年 7 月发表的系统性综述（arXiv 2607.13104），将 Agent 自我改进领域首次统一为**双路径框架**，涵盖从提示优化到哥德尔机器的完整技术谱系。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

## 核心形式化定义

智能体被定义为**基础模型参数 θ 与操作支架 Σ 的耦合配置**：**At = (θt, Σt)**，其中 **Σt := (pt, mt, Tt, gt)**

- **pt**：结构化提示 / 系统指令
- **mt**：记忆（存储、检索、更新策略）
- **Tt**：外部工具集及调用接口
- **gt**：控制逻辑（路由、调度、安全约束）

自我改进 = **自诱导更新算子 U**，将执行经验转化为对 θ 或 Σ 的持久变更。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

## 双路径分类法（核心贡献）

### 路径 A：基础模型改进（θ 更新，Σ 冻结）

| 子类 | 信号形式 | 代表方法 | 关键挑战 |
|------|---------|---------|---------|
| 内在生成式演示 (§5.1) | 自合成数据 → SFT | Self-Instruct, LMSI, SPORT | 模型坍缩、知识气泡 |
| 内在评估反馈 (§5.2) | 自评分/偏好/批评 → RL/DPO | Constitutional AI, Meta-Rewarding, TTRL | 评估器-策略耦合放大盲点 |
| 外在探索经验 (§5.3) | 环境交互轨迹 → RL | WebRL, UI-Genie, Agent-RLVR | 奖励稀疏/黑客、能力倒退 |

### 路径 B：支架改进（Σ 更新，θ 冻结）

| 组件 | 范式 | 代表方法 |
|------|------|---------|
| **提示** (§6.1) | 标量反馈 → 定性精炼 → 种群进化 → 文本梯度 | APE, Reflexion, Promptbreeder, TextGrad |
| **记忆** (§6.2) | CRUD + 扁平/层级/图/向量结构 + 选择治理 | Generative Agents, Mem0, MemoryBank, A-MEM |
| **工具** (§6.3) | 动态路由 → 迭代精炼 → 自主创建 | ToolNet, MCP-Flow, VOYAGER, TOOLMAKER |
| **全支架** (§6.4) | 自指代码改写，改进器与被改进系统共同进化 | Gödel Agent, AlphaEvolve, STOP, Agent Symbolic Learning |

## 关键设计洞见

### 快慢双环

Σ 改进（快速适应）与 θ 改进（慢速巩固）存在结构性不对称——当反馈有噪声时，先约束在支架内验证，稳定后再蒸馏到参数中。这是整个综述最具实践指导意义的结论。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

### 评判即治理基础设施

评判器不是被动基准而是**攻击面**，需与生成器解耦。评估应报告完整学习曲线而非峰值分数，并追踪回归率和安全违规。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

### 分层门控安全

自我改进应视为**不可信代码在受保护运行时中执行**，每次结构更新需通过验证器门控。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

## 六大未来方向

1. **测试时持续适应** — 运行时动态更新检索/路由/记忆
2. **主动探索与好奇心** — 内在奖励驱动安全探索
3. **参数蒸馏与联合优化** — "系统 2 → 系统 1"，θ 与 Σ 联合优化
4. **资源约束下的改进动力学** — 预算感知的效率优化
5. **多智能体协同共进化** — 共享工件、安全版本控制
6. **开放世界分布漂移下的鲁棒性** — 非平稳模拟器替代静态排行榜

## 全景图谱补强：从 Skill 到模型权重的进化谱系（AliExpress 技术，2026-08）

阿里 AliExpress 技术（朱灵子）2026-08-25 发布的自进化全景综述，与本综述（其参考文献第 1 条）形成互补——本综述给出形式化双路径框架（θ / Σ），该文给出**按进化对象的四层分类 + 具体方法的机制映射**，并补入本页及库内尚未覆盖的两个机制。^[raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25.md]

**「进化什么」的四层分类。** 该文把进化对象明确分层：①Agent 系统进化——Skill、Harness、Memory、AFlow、Environment 等非参数化组件；②模型参数进化（把经验内化为权重）——微调/强化学习、自我归因、自蒸馏。这与本综述「支架 Σ 更新（快）vs 基础模型 θ 更新（慢）」的双路径一一对应，四层分类提供了更细的组织粒度。^[raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25.md]

**Skill 进化的四类生命周期（含两个库内零覆盖机制）。** 该文把 Skill 自进化按「轨迹生成 → 群体进化 → 评估更新 → 协同进化」四类组织，并映射具体方法：^[raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25.md]

- **轨迹生成**：Trace2Skill（批量轨迹 → 并行多智能体补丁提议 → 分层合并去冲突；实测 122B 模型生成 200 条 50+ 轮轨迹不足两小时）。
- **群体进化**：EvoSkill（Executor / Proposer / Builder 三子智能体闭环）、SkillOpt（Skill 文本视作「可训练外部参数」，小批量反思 + 学习率约束 + 独立验证集门控 + 动量与元记忆，类比 SGD）。
- **评估更新**：**coEvoSkill（库内零覆盖）**——把 Skill 进化设计成生成者与验证者的**双边对抗闭环**：生成器改 Skill，验证器负责出题、诊断、升级测试，真实环境提供黑盒 pass/fail，把自进化从「单边自省」变成带对抗性的共同进化，并引入跨代理竞争反馈按贡献调整 Skill 权重与结构。^[raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25.md]
- **协同进化**：SkillRL（RL 与 Skill 库双向蒸馏 + 分层 Skill Bank〔通用 / 任务特定 / 常见错误〕+ 递归演化）、**D2Skill（库内零覆盖）**——双粒度技能建模（任务技能指导全局规划、步骤技能纠正局部操作）+ 对比学习机制（并行采样技能组与基线组，以性能差构建事后效用信号供 Skill 估值与奖励塑形）+ 失败触发反思生成新技能 + 语义/效用两阶段检索与定期剪枝的动态管理闭环。
- **GiGPO 算法**（SkillRL 与 D2Skill 共用）：在保留 GRPO「critic-free」前提下用 Rollout 中天然存在的**状态重复**（Anchor State，如重复点击同一搜索结果）离线 Hash 聚合构成 Step-level Group，与 Episode-level 优势按 50/50 融合。该文给出的 GRPO vs GiGPO 六维对比表（优势粒度 / 信用分配 / 额外开销 / 数据利用率 / 收敛特性 / 核心依赖=环境状态可重复访问性）是本页原缺的工程细节。^[raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25.md]

**Harness 进化的「时间尺度不对称」与元进化。** 该文点出一个本页未显式表述的判据：**Harness 状态在部署期可被频繁、低成本地检查与修订，模型权重不能**——这一不对称是「先改 Harness 后写权重」的根本原因（与本综述「快慢双环」结论同源）。Harness 进化侧的 EvoAgentX（Prompt 用 TextGrad / 拓扑用 AFlow / 配置参数用 MIPRO 三条优化线并行进化）与元进化侧的 HyperAgents（Task Agent + Meta Agent 统一写在可编辑程序中，Meta Agent 可改自身代码，系统自己发明了持久化记忆与性能追踪机制；跨域迁移 DGM-H imp@50 达 0.630 vs 原版 DGM ≈ 0）。^[raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25.md]

**模型参数进化的三条路径。** ①反馈进化（人工修正数据 GRPO 后训练 / 业务标注 RL 对抗奖励后训练）；②自我归因（AgentEvolver：自我提问生成任务 + ReMe 经验池自我导航 + ADCA-GRPO 自我归因——LLM 回溯给每步打 GOOD/BAD，过程奖励 ±1 按轨迹级标准化后与结果奖励加权融合；AppWorld 达基线 90% 性能所需训练步数 -55%、BFCL-v3 -67%）；③自蒸馏（OPSD，On-Policy Self-Distillation：同一模型「自己教自己」，教师策略带答案预测 token 分布、学生策略仅凭题目采样，以 JSD 最小化为目标，梯度仅回传学生）。^[raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25.md]

**「有效自进化」的两个落地障碍。** 该文补上本页未展开的失败面：①**方向不可控（过拟合风险）**——单轮轨迹带偶然性甚至极端 case，直接据此更新会让 Skill 为迎合个别 bad case 变得臃肿发散，出现「越优化越差」；②**质量不稳定（缺乏验证）**——无客观量化评估体系时无法区分「Skill 真变好」与「环境噪声导致的指标波动」。对策指向数据反馈与评估体系（呼应本综述「评判即治理基础设施」）。^[raw/articles/agent-self-evolution-skill-to-weights-panorama-aliexpress-2026-08-25.md]

## 与 2026.7.27 阅读序列的关系

这篇综述为当天阅读的 7 篇文章（DataFlow-Harness、Reflection/Reflexion、TokenSpeed、MemoHarness、淘宝 SDD AI Coding、Agent OS、Own Your Intelligence）提供了统一的理论框架。其中淘宝 SDD AI Coding 对应 §7.1 软件工程实践，Reflection/Reflexion 对应 §6.1.2 定性反馈精炼，DataFlow-Harness 对应 §6.4 全支架改进，Agent OS 对应 §9.1 分层门控安全架构。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

→ [[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104|原文存档]]

## 深度分析

### 边界实际画在「回滚成本」上，不在「技术难度」上

θ 与 Σ 的切分看似技术分类，实质是一条经济学界线：Σ 的每次改动都是可 diff、可回滚、单次毫秒级生效的文本或配置，而 θ 的一次更新是不可逆的权重覆写，需重跑全量回归且可能污染其他任务分布。因此决定某次改进落在哪一侧的，不是「哪种方法更先进」，而是「这次改动的错误代价能否被一次回滚吸收」。当一条经验的验证样本不足百条时，写进提示或记忆的期望收益恒高于做一次微调——这是成本结论，与模型能力无关。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

### 缺外部验证器的自改进回路，必然坍缩成奖励黑客

凡把「自己生成的信号」当作「自己消费的信号」的回路，都在优化一个与真实目标逐渐脱钩的代理量。综述把评判器称作攻击面而非基准，正因为评测器与策略共享同一先验：模型看不见的盲点，它自己出的题也考不出来。可用的回路必须挂一个策略不可写的验证器——单测、真实环境 pass/fail、或人类抽检——且它要能**拒绝**更新，而不只是给分。缺这一步，回路不会停转，只会朝更容易被自己骗过的方向漂移，这与 [[entities/anthropic-reward-hacking-hacker-opus-hugging-face-2026-09|奖励黑客实例]] 的观察一致。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

### 形式化的真正收益：把「怎么变好」拆成四个可分别设计的算子

扁平分类法只能回答「有哪些方法」；At = (θt, Σt) 加 Σt := (pt, mt, Tt, gt) 的分解，让四个问题第一次变得可问：这次更新的对象是哪个分量、该分量的状态空间有多大、经验到该分量的映射是否可微、更新后哪个不变量必须保持。第四条最关键——它把安全从外挂流程变成算子的约束项。没有这层分解，讨论自改进只能停在方法罗列；有了它，才判断得出两个方法是否真在同一坐标系里可比，也才谈得上把「快环经验」有损地蒸馏进「慢环参数」而不丢关键约束（参见 [[entities/lossy-self-improvement|有损自改进]]）。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

### 三条支架轴的代价与天花板各异，不能按同一节奏推进

提示轴最便宜，秒级可迭代，但状态空间被上下文窗口硬性封顶，复杂度超过一定阈值后新增指令互相干扰、边际收益转负；记忆轴写入便宜而读取昂贵，真实成本在检索治理与陈旧条目清理，天花板取决于条目质量而非容量；工具轴最贵，需沙箱、权限与接口维护，却是唯一能突破模型固有能力上限的轴。故常见推进顺序是提示→记忆→工具，一旦诊断出瓶颈在「模型根本不会做」而非「没告诉它怎么做」，就该跳过前两轴。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

### 哥德尔机给了目标函数，现代 LLM 自改进给的是可行近似

Schmidhuber 谱系的核心断言是：能改写自身代码、并能**证明**改写更优的系统可以无限自举；代价是「证明更优」在实践中不可判定。现代 LLM 自改进实际做的，是把不可判定的证明换成可运行的验证——不证明新版本更好，只在有限样本上测出它没变差就接受。这一替换使系统从理论上的封闭最优降格为工程上的有偏随机搜索，也解释了综述为何反复强调学习曲线与回归率：我们买到的不是保证，而是可观测的漂移。[[entities/rrsi-regularized-recursive-self-improvement-agent-harnesses|RRSI]] 的正则化思路正是对这一降格的补丁。^[raw/articles/self-improvements-modern-agentic-systems-survey-arxiv-2607-13104.md]

## 实践启示

1. **先给 Σ 建版本化存档，再谈自动改进。** 提示、记忆条目、工具签名、路由配置必须能在一个提交内整体回滚。没有 diff 与回滚，任何自动改写都是在生产上做不可逆实验；每次变更需记录触发的经验 id，指标下滑时才定位得到具体改动。

2. **把评测器放在 harness 之外，并给它否决权。** 评测器不得与被测策略共用同一份提示和同一批示例；输出应是「接受/拒绝 + 原因」，而非可被优化器追逐的连续分数。每轮更新强制跑一组冻结回归集，旧任务掉分的候选一律拒绝。这是 [[concepts/agent-self-improvement-loops|自我改进循环]] 能否长期存活的唯一支点。

3. **仪器化「每轮改了什么、涨了多少、退了多少」。** 只统计峰值分数会系统性高估收益；应按轮记录 pass 率、回归失败数、安全违规数、token 与延迟成本，四者同图观察，才能把真实提升从环境噪声里分出来。

4. **用「模型是否会做」这道题决定要不要动权重。** 若失败模式是流程遗漏、格式不符、工具选错，属支架问题，改提示与路由即可；只有当同一任务在正确提示下仍稳定失败、且已积累上千条带验证标签的轨迹时，权重级投入才划算。

5. **盯三个早期失败信号：指令膨胀、记忆腐化、验证器同质化。** 提示长度持续增长而分数不动，说明在打补丁；记忆条目数上升但命中率下降，说明在积累噪声；生成器与验证器的错误高度相关，说明回路已经开始自欺。

## 相关实体

- [[entities/dataflow-harness-pku-code-agent-data-pipeline|DataFlow-Harness]]
- [[entities/harness-engineering|Harness Engineering]]
- [[entities/harness-engineering-self-improvement-survey-lilian-weng|Harness Engineering & Self-Improvement Survey (Lilian Weng)]]
