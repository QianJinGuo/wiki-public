---
title: "Self-Improvements in Modern Agentic Systems: A Survey — Agent 自我改进综述"
created: 2026-07-27
updated: 2026-09-10
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

## 相关实体

- [[entities/dataflow-harness-pku-code-agent-data-pipeline|DataFlow-Harness]]
- [[entities/harness-engineering|Harness Engineering]]
- [[entities/harness-engineering-self-improvement-survey-lilian-weng|Harness Engineering & Self-Improvement Survey (Lilian Weng)]]
