---
title: "AI 自我改进与自举"
created: 2026-06-11
updated: 2026-08-29
type: concept
tags: [agent, self-improvement, llm, memory, recursive-self-improvement, data-flywheel, skill-evolution, lossy-improvement]
description: "AI 自我改进与自举：递归优化、自对弈、数据飞轮、能力迭代"
---

# AI 自我改进与自举

> AI 学习正从训练阶段溢出到部署阶段——权重冻结下通过外部状态层（记忆 / 技能 / 数据飞轮 / Harness 编排）积累知识，是"毕业后的自学能力"。本概念页汇总 wiki 中 12 个相关实体的核心洞察，给出四类自举路径的统一视角。

## 一、核心定义

**AI 自我改进**（self-improvement）指 AI 系统在**不重新训练基础模型**的前提下，通过自身输出与外部反馈的循环，使后续表现优于过往表现的能力。

**自举**（bootstrapping）指系统从少量种子能力出发，**通过自我生成的样本 / 工具 / 经验**持续扩展能力边界——形成"用 X 改进 X" 的闭环。区别于传统监督学习需要人类标注，自举的反馈信号来自系统自身或环境。

两者交叉形成 4 类典型路径：

| 路径 | 反馈源 | 能力载体 | 典型代表 |
|------|-------|---------|---------|
| **递归自我改进** | 自对弈 / 自评分 | 训练数据 / 训练流水线 | Prime Intellect（nano-GPT Prime）、MIRA + MPA |
| **Skill 进化** | 用户反馈 / 任务成功率 | 可复用 Skill 三元组 (M, R, C) | Anthropic Skills 综述、Claude Code Self-Repair Hooks |
| **数据飞轮** | 生产环境真实使用日志 | 训练数据集 / 评估集 | OpenAI Tax AI Eval Loop、Anthropic 95% 数据分析自动化 |
| **世界知识自演化** | 环境探索结果 | 知识图谱 / 推理前缀 | World Knowledge Agent（Tencent-HKUSTGZ） |

## 二、关键约束：Lossy 风险与能力稳定性

递归自改进的核心警告来自 "Lossy self-improvement" 实体——**每一轮自举都会引入微小误差 / 噪声，多轮累积后会出现"漂移-坍缩"现象**：模型在某些指标上分数上升，但实际能力边界却在收缩。

**三类失效模式**：
1. **模式坍缩（mode collapse）**：自对弈生成的样本多样性下降，模型只会做"它会做的事"
2. **评估过拟合（eval gaming）**：模型学会通过评估器（evaluator），但不真改善任务表现
3. **数据自噬症（model autophagy disorder, MAD）**：用自己输出训练自己，导致能力分布向 LLM 已擅长区域倾斜

**对冲策略**（综合 12 个实体的实践）：
- **多源反馈混合**：自评分 + 用户反馈 + 任务成功率（OpenAI Tax AI 同时跑 eval + 用户票 + 业务指标）
- **保留种子数据**（"golden set"）作为能力下界
- **能力分布监控**：用 HumanEval / SWE-bench 等独立基准做月度对账

## 三、四条路径的工程化实践

### 3.1 递归自我改进 — Prime Intellect 范式

[Prime Intellect](entities/ai-recursive-self-improvement-nanogpt-prime-intellect) 公开了 nano-GPT Prime 实验：从一个 16M 参数的小模型出发，用**自对弈（self-play）生成的训练样本**迭代训练 7 轮，模型在 MATH benchmark 上分数持续上升。这是递归自改进的"教科书案例"，但其样本生成器也是 LLM——存在自噬风险。

[MIRA + MPA](entities/mira-mpa-deep-principle-ai4s-40-sota) 在材料科学领域用同样的递归思路做出 40 项实验 SOTA，**关键差异**是 MIRA 的反馈来自真实物理实验（DFT 计算），不是 LLM 自身——大幅降低自噬风险。

### 3.2 Skill 进化 — 沉淀可复用三元组

[Agent Skills 综述](entities/agent-skills-comprehensive-survey) 把"技能"形式化为 **S = (M, R, C)**：M = 主指令 / R = 检索 / C = 上下文。Anthropic 的生产 Skill 库就是这种三元组的大规模积累，**每次新任务完成后，Skill 通过反馈循环自动补全**。

工程实践上对应的两条路径：
- **被动沉淀**：从生产 trace 自动提取高频模式 → 升级为 Skill（见 [DeliAutoResearch SKILL](entities/deli-auto-research-skill-deepseek) 的 v1 → v2 演进）
- **主动编码**：开发者预先设计 Skill 模板，Agent 调用后用参数化方式扩展

[Anthropic 95% 数据分析自动化](entities/anthropic-95pct-data-analysis-skill-stack-architecture) 是 Skill 进化的极端案例：21% 准确率 → 95% 准确率，核心就是为数据分析各子任务沉淀专用 Skill 库（Schema 推断 / 异常检测 / 可视化生成 等 30+ 技能）。

### 3.3 数据飞轮 — Eval Loop 与 Tax AI 范式

[OpenAI Tax AI](entities/xinzhiyuan-openai-tax-ai-self-improving-codex-eval-loop-20260606) 公开的 "Eval Loop" 范式：**用户真实调用 → LLM 输出 → 用户反馈 / 任务结果 → 收集为训练数据 → 下一版模型**。这是数据飞轮最纯净的形态。

关键设计原则（来自 [Agent 自我改进六条路](entities/agent-self-improvement-six-mechanisms)）：
- **Eval 集与生产环境分布保持一致**——不能"训练看 A，部署看 B"
- **反馈延迟容忍**——长程任务需要中间检查点而非只看最终结果
- **可解释评估**——不是"答对答错"二元信号，而是"哪个推理步骤错了"

### 3.4 世界知识自演化 — 推理前先探索

[World Knowledge Agent](entities/world-knowledge-agent-self-evolution-tencent-hkustgz) 提出"**推理前先探索**"模式：Agent 接到任务后不是直接解题，而是**先在环境中搜索 / 执行 / 观察**，把发现的事实结构化成可迁移知识（写进 memory），再基于这些知识做推理。这把"知识获取"和"知识使用"解耦——前者走世界模型，后者走语言模型。

## 四、相关实体

- [[entities/agent-self-improvement-six-mechanisms|Agent 自我改进的六条路]]
- [[entities/agent-skills-comprehensive-survey|Agent Skills 系统性综述：表示→获取→检索→进化]]
- [[entities/ai-recursive-self-improvement-nanogpt-prime-intellect|AI科研超越人类 — Prime Intellect递归自改进实验]]
- [[entities/anthropic-95pct-data-analysis-skill-stack-architecture|Anthropic 内部 95% 数据分析自动化：分析 Agent 技术栈 + Skill 框架（21%→95% 准确率）]]
- [[entities/claude-code-self-repair-hooks-memory-config|复制这套神仙配置，让Claude Code全自动修Bug！告别每天重复教AI写代码]]
- [[entities/deli-auto-research-skill-deepseek|DeliAutoResearch SKILL：DeepSeek陈德里的自主科研智能体框架]]
- [[entities/deli-auto-research-skill-v2-continual-learning-self-improvement|DeepSeek陈德里AI论文第二弹：从6分到8分，DeliAutoResearch SKILL又进化了]]
- [[entities/lossy-self-improvement|Lossy self-improvement]]
- [[entities/mira-mpa-deep-principle-ai4s-40-sota|MIRA + MPA：深度原理 AI Scientist 递归自训练打造材料基座模型，40 项实验全面 SOTA]]
- [[entities/problem-with-mathematically-proven-claims-about-llms|problem with mathematically proven claims about llms]]
- [[entities/the-shape-of-the-thing-mollick|The Shape of the Thing：AI 指数曲线·软件工厂·滚动颠覆（Mollick 2026-03）]]
- [[entities/world-knowledge-agent-self-evolution-tencent-hkustgz|World Knowledge：Agent推理前先探索环境生成可迁移知识]]

## 五、相关概念

- [[concepts/agent-memory-architecture|Agent 记忆架构]]（自举依赖长期记忆）
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]（Harness 是自举的载体）
- RLHF/DPO/GRPO 对齐（人类反馈也是反馈源）
- [[entities/llm-self-improvement-system-survey-zesearch-nlp-2026|LLM 自我提升系统综述（Yang 等 113 页）]]（本 wiki 2026-06 新增的 4 阶段闭环框架）

## 六、实践启示

1. **不要从"递归自改进"开始**——Lossy 风险高、调试困难。从 Skill 进化或数据飞轮开始，积累反馈通道
2. **每条路径都需要"独立评估器"**——不能用模型自评分当唯一反馈，否则必然走向上述三类失效
3. **保留"种子数据"**——任何自举系统都该有 5-10% 的"非自举"数据作为能力下界监控
4. **优先沉淀 Skill 而非改权重**——Skill 进化可解释、可回滚、调试成本低；改权重一旦失败损失巨大
5. **Eval 集与生产分布一致**——OpenAI Tax AI 反复强调的硬约束

## 所属 MOC

- [[moc/ai-skill-design|Ai Skill Design]]
