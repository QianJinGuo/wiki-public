---
title: "Codex、Claude Code 的推理 effort 本质就是往 prompt 里塞了一句话"
created: 2026-08-04
updated: 2026-10-01
type: entity
tags: [reasoning-effort, inference-scaling, rlvr, post-training, sft, codex, claude-code, deepseek, qwen3, kimi-k3]
sources:
  - raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话
confidence: 0.75
rating: v7c8
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Codex、Claude Code 的推理 effort 本质就是往 prompt 里塞了一句话

## 核心定位

本文基于 Sebastian Raschka 的长文，把 DeepSeek-R1 的 RLVR 一直拆到 Inkling 的连续值 effort conditioning，对比六份开源模型技术报告，建立推理档位（low/medium/high/max）背后的工程化理解框架。核心论点：**推理档位选择器的本质是一条自然语言 system prompt（如 "Reasoning effort: high"），它对应训练管线里一组具体的、公开可查的工程决策**，而非"多想一点"的黑箱魔法。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

## 推理模型的基础机制

### RLVR：只打分，不教步骤

推理模型行为的核心训练方法是 RLVR（Reinforcement Learning with Verifiable Rewards），来自 DeepSeek-R1：给问题 → 模型生成答案 → 外部验证器检查最终答案（数学用 SymPy/WolframAlpha，代码用编译器/单测）→ 对给 reward 1，错给 0。中间推理 trace **完全不参与评分**，训练只看最终答案（R_total = R_accuracy + R_format）。只靠"奖励结果"这一条，模型自己涌现出写中间步骤、回溯检查、自我修正的行为（Aha moment）。前提是领域必须能自动核对对错——数学和代码天然满足。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

### think 标签是界面需要，不是推理需要

`<think></think>` 标签对推理能力本身没有贡献——它是装饰性的，唯一作用是标记 reasoning trace 的起止位置，方便 ChatGPT/Codex 界面折叠隐藏。证据：不带标签重新训练 benchmark 几乎一样，换成任意符号效果相同。它是 RLVR reward 里的格式奖励训出来的约定，不是推理的机制基础。但标签圈出来的区域是全部工程操作的作用面：预填空标签关闭推理、标签中间截断控制长度、惩罚系数影响标签区域 token 数量。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

## 从"关不掉的话痨"到混合模式

第一代推理模型（DeepSeek-R1）只有一个档位——开，且关不掉（问 "hello" 也长篇推理），产品里难用。Qwen3 是转折点：Thinking Mode Fusion 在 SFT 阶段混合 `/think` 和 `/no_think` 两种样本，`enable_thinking=False` 在 chat template 层面预填空 `<think></think>` 块让模型直接答——硬开关保证模型不会"自作主张"开始推理。到 GPT-5.6 这一代，开关从二元变成多档（low/medium/high/max/ultra）。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

## 档位的实际载体与三条训练路径

### 档位 = system prompt 一句话

OpenAI 的 gpt-oss chat template 在每次请求前加 `Reasoning effort: low/medium/high`，界面下拉选择器就是把按钮映射成这句话。DeepSeek V4 Think Max 用 `Reasoning Effort: Absolute maximum with no shortcuts permitted`，Inkling 用连续值 `Thinking effort level: 0.8`，Kimi K3 在 API 暴露 `reasoning_effort` 参数。**档位的唯一载体就是这句自然语言指令。**^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

### 三条训练路径（六份技术报告的归纳）

随便拿未训练模型塞 "Reasoning effort: high" 不会生效——训练阶段必须把"这句话 → 这个行为"的映射编码进策略：

- **路径一：RLVR 阶段调长度惩罚** — 不同 system prompt 配不同 token 惩罚系数 λ，`R(e) = R_task − λ(e) × N_tokens`，说 low 时 λ 大逼模型写短。DeepSeek V4 和 Inkling 走此路。
- **路径二：RLVR 之后补 effort-conditioned SFT** — 收集"这个 prompt 要这么长的推理"配对样本，SFT 学会 label→长度映射。Qwen3 Thinking Mode Fusion 是此思路（二元开关版）。
- **路径三：多专家蒸馏** — 分别训 low/high/max 专家（不同长度数据、不同窗口、不同惩罚），on-policy distillation 合并进同一 checkpoint。DeepSeek V4 和 Kimi K3 明确走此路。

三条路径不互斥；推测 gpt-oss/GPT-5.6 是路径一+二组合（先 RLVR 做推理基础，再 SFT 植入档位响应），可能最成熟。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

## 换模型和调档位是两个独立轴

GPT-5.6 界面把两件事分清楚：选模型（Luna/Terra/Sol）是换权重文件（训练 scaling），调档位是模型不变只改推理 token 预算（推理 scaling）。同一模型不同档位是一条曲线，不同模型是不同曲线——**小模型高档位可以追平大模型低档位**。沿曲线走是推理 scaling，跨曲线是训练 scaling。规律：档位边际收益递减（high→max→ultra 提升收窄）。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

## 相关实体与概念

- [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] — 推理行为涌现的核心训练方法
- 推理模型 — reasoning trace 行为定义
- [[concepts/llm-pretraining-vs-sft|预训练 vs SFT]] — 路径二的训练阶段基础
- [[concepts/inference-optimization|推理优化]] — 推理 scaling 的成本/延迟权衡
- [[entities/deepseek-v4-training-methodology|DeepSeek V4 训练方法论]] — 路径一+三的实践
- [[entities/claude-code-extended-thinking-not-authentic|Claude Code Extended Thinking]] — effort 机制的关联讨论
- [[entities/llm-thonking-reasoning-effort-security-triage|LLM 推理 effort 安全分流]] — effort 在安全场景的应用
- [[entities/deploying-kimi-k3-on-aws|Kimi K3 部署]] — reasoning_effort 参数的工程实践
- [[entities/ai-agent-loops-claude-code-codex|Agent Loops]] — Codex/Claude Code 的工程上下文

## 深度分析

### 表面相同的档位标签，背后管线可能完全不同

把六份技术报告并排看，最容易踩的坑是用档位标签反推模型行为。同样叫"三种模式"，DeepSeek V4 是把三个推理专家用不同 context window 和长度惩罚各自独立训完，再经 on-policy 蒸馏合并进单一 checkpoint；而 Nemotron 3 Ultra 的 medium-effort 是拿 GPT-OSS-120B 的输出当 teacher 蒸馏出来的便宜推理模式。表面一样，选择路径、工程约束和迭代成本完全不同。由此得到的推论：如果一份模型报告只宣布"提供 low/medium/high 三档"却不交代怎么训出来的，这个档位对你就是黑箱——你无法区分"low"是模型被训练成"少写"，还是"多写但被外部截断"，两种行为的失败模式截然不同。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

### 档位与预算正在解耦：行为模式和 token 上限是正交参数

Nemotron 做了一件其他方案都没做的事：把"学出来的推理行为"和"外部硬截断"拆成两套独立机制。它的截断训练数据构造很精巧——取一条正常推理 trace 在随机位置截断、保留原始最终答案，插入的 `</think>` 从 SFT loss 里 mask 掉，即模型不学"自己决定在这里结束"，只学"推理可在任意位置被外部中止，中止后仍要答对"。对照之下 Qwen3 从未显式训练截断，却在 Thinking Mode Fusion 之后涌现出同样能力——暗示模型一旦掌握 `<think>` 区域的起止概念，外部截断就是容易泛化的操作。若这种解耦成为标准，未来推理 API 可能拆成行为模式 + token 预算两个独立参数：大多数时候不需要降档省钱，在当前档位上加预算即可——code review 保持 high 但限 2000 token，和切成 low，效果可能完全不同。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

### 固定预算训练会侵蚀推理根基：Kimi K2.5 Toggle 的"先稳后快"

用固定 token 预算训练模型，它会过拟合短答案——变快、变便宜，但失去靠增加推理量攻克难题的能力。这是推理能力（需要长 trace 训练）与成本压缩（要求短 trace）之间的真实冲突。Toggle 的解法是在 RL 训练中交替预算阶段与无约束阶段：预算取该题目历史上所有正确 rollout 的长度分布分位数，且只在模型对该题准确率超过阈值后才激活——先确保做得对，再训做得短。效果是生成 token 削减 25%~30% 而 benchmark 几乎不掉，且可快可慢的能力还从数学/代码 RL 迁移到了 GPQA 和 MMLU-Pro。方法论启示：任何压缩推理成本的训练，都必须保住"无约束阶段"这条退路，否则短预算会反向侵蚀推理根基。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

### chat template 正从格式约定变成行为控制协议

GLM-5 把二元开关扩展成三种行为模式：交错思考（每次 tool call 前都插入推理块，而非想完一次就机械执行）、保留思考（多轮对话中复用之前的推理块，避免每轮从零重推）、按轮次开关（每轮独立控制推理与否）。当 chat template 成为推理行为的控制面之后，能往里塞的不只是"开不开"，还有"在哪开""开多久""要不要看历史"。另一个易误读的例子是 DeepSeek V4 那句 `Absolute maximum with no shortcuts permitted`——看着像 prompt 技巧，实际和 RLVR 阶段的惩罚配置绑定，换一个没经过该训练的模型照抄毫无效果。system prompt 永远只是触发器，不是杠杆；档位对应的行为在训练阶段就已被 reward 设计决定了。^[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话.md]

## 实践启示

1. **选模型先看报告透不透明。** 只宣传档位数量、不公开训练细节的模型，其档位就是黑箱——你不知道 low 是"被训练成少写"还是"多写但被截断"。六份公开配方的报告值得优先读。
2. **成本敏感的生产场景优先选支持外部硬截断的模型。** 在当前档位上加 token 预算上限，比降档更安全：推理质量由模式保证，成本由预算控制，两者不打架。
3. **别把最高档当默认。** 档位边际收益递减是跨模型规律——high→max→ultra 每档提升收窄，成本却线性甚至超线性增长；Inkling 在 effort 0.8 之后分数走平而 token 还在涨。
4. **用三五条显式规则做档位路由，别指望 Auto 模式。** GPT-5 的 Auto 效果不好已经下架；自动选档目前没有通用解。简单问答→low、多步推理→high、代码生成→max 的静态规则便宜、可控、出错可定位，与 [[entities/llm-thonking-reasoning-effort-security-triage|LLM 推理 effort 安全分流]] 是同一思路。
5. **把"换模型"和"调档位"当两个旋钮组合调优。** 小模型高档位可以追平大模型低档位（Luna 开 high 有时和 Terra 的 low 分数相当）；没有普适最优组合，按精度、成本、延迟权衡，预算紧张时先动便宜的旋钮。
6. **连续 effort 值留给程序消费，不要直接暴露给人。** 0.7 和 0.8 的差别用户答不上来；Inkling 自己最后也把连续值映射回了离散选择器。离散标签粗但直觉清晰，连续值更适合作为内部 router 的调参输入。

→ [[raw/articles/codexclaude-code-的推理-effort-本质就是往-prompt-里塞了一句话|原文存档]]

