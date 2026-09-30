---
title: "Formalizing Fermat's Last Theorem：Claude 用 Lean 端到端形式化证明费马大定理"
created: 2026-09-07
updated: 2026-09-30
type: entity
tags: [ai4math, formal-verification, lean, prove2me, multi-agent, anthropic, claude, theorem-proving]
sources: [raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026]
confidence: 0.9
---

# Formalizing Fermat's Last Theorem：Claude 用 Lean 端到端形式化证明费马大定理

Anthropic 于 2026-09-04 宣布首个完整的计算机可校验的费马大定理（FLT）证明：Claude 在 11 天内、以高度自主的方式，用 **Lean** 证明助手写完了 Wiles 证明的形式化版本。这是 AI 自动形式化（autoformalization）领域的一个里程碑式成果。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 背景：为什么 FLT 如此难形式化

费马在 1637 年于《算术》页边写下"我发现了真正绝妙的证明，但页边太窄写不下"，其后 350 余年无人证出。1908 年悬赏 10 万德国金马克，仅第一年就出现 621 份错误尝试。1993 年 Wiles 三场讲座后被发现存在关键缺口，又与 Taylor 苦战一年才在 1995 年发表正确证明——篇幅 129 页，依赖远超 1637 年时代的现代数学技术。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

形式化（把数学推理转成计算机可自动校验的形式）是一项公认难题：人类证明会跳过大量显然步骤，而 Lean 需要看到每一步；且形式化只能从已被形式化的少数数学出发，无法直接借力几百年的出版物积累。仅社区项目（2024 年 Kevin Buzzard 在帝国理工发起的 multi-year 努力）的形式化 blueprint 就有 86 页，原预计整个形式化需数年。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 关键成果与技术数据

Claude 用 11 天完成 FLT 形式化，沿途产出 30,300 个定理（最终证明用到 29,500 个），共 13 百万行 Lean 代码——是 Mathlib（社区主要数学定理库）规模的 5 倍以上，也是迄今构建的最大 Lean 证明。约 6 billion output tokens 消耗于一个接近 Claude Fable 5.1 的通用内部研究模型。证明只用了 Lean 的三个标准公理，并经 comparator 确认定理表述与 Mathlib 对 FLT 的表述一致。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

Claude 的证明遵循 Darmon-Diamond-Taylor 对 Wiles 证明的简化版本。人类输入仅限 Tianyi Peng（Anthropic 研究员，哥伦比亚大学课题组）偶尔的高层指令，如"Jacobian as a scheme 优先级高""尽快推 Mazur 定理"。多位 Claude agent 分工协作定义概念、证明中间定理、再用这些定理证明更难命题。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 为什么能成功：Prove2Me 平台与多 Agent Harness

值得注意：Claude 的数次初始尝试都失败了——agent 虽有早期成功，但很快丢失项目状态、停止有效协作，失败尝试贡献了最终证明中约 7% 的非样板代码行。**转用 Prove2Me 后才成功**。Prove2Me 是 Peng 与其哥伦比亚合作者设计的开放协作形式化平台，其设计要点：^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

- **定理语句 DAG**：维护定理陈述的有向无环图（DAG），供 agent 决定下一步证明什么，缓解记忆退化（memory degradation）并允许多 agent 并行。
- **编译提速**：把定理陈述与证明拆分到不同文件、独立维护链接，加速 Lean 编译、最小化资源消耗。
- **搜索复用**：为每个定理维护自然语言描述，简化证明路径。

配合基于 Claude Code 的多 Agent harness，一个 agent 团队在不到两周内完成证明（11 天主体工作）。这与 [[entities/claude-code-multi-agent-harness-source-analysis|Claude Code Multi-Agent Harness]] 的多 agent 编排模式高度相关——DAG 化任务分配是这群 agent 不"跑偏"的关键。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 对形式化验证与 AI 数学的意义

Anthropic 与评论者（Kevin Buzzard）认为这证明"自动形式化当代数学文献已成为可能"——可用来找出数学共同语料中的错误、减轻审稿/referee 负担，并以可校验方式核查 LLM 生成的数学。形式化也被视为人类对 AI 生成数学结果建立信心的主要途径：AI 产出越来越多"声称的证明"后，形式化可能是数学界跟上的唯一可行方式。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

一个低成本信号：用三个个人 Claude Max 订阅、完全经 Prove2Me 协作，agent 在三天内完成了 Vinogradov 三素数定理的形式化——说明在合适 scaffold 下，用消费级 AI 订阅协作形式化重大结果可达。这也呼应 [[entities/lean-scaling|Lean 软件扩展定律]] 所关注的形式化规模与成本问题。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md]

## 深度分析

**失败的结构性根源不在模型能力，而在状态外化。** Claude 的初始尝试并非败在数学推理——agent 早期能产出有效进展——而是"快速丢失项目状态、停止有效协作"，即长周期任务中典型的记忆退化（memory degradation）与协调崩塌。失败尝试的 ~7% 非样板代码行最终被回收进证明，说明这些失败并非全然浪费，但真正的分水岭是换用 Prove2Me 这类外部化状态结构。这与 [[entities/claude-code-multi-agent-harness-source-analysis|Claude Code Multi-Agent Harness]] 的核心结论一致：agent 系统的成败往往由 harness 决定，而非底层模型。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:116-128]

**三个平台设计要点对应三类 harness 通病。** 定理语句 DAG 解决"无共享任务图的并行必然跑偏"——每个 agent 以 DAG 为唯一真相源决定下一步证明什么，任务依赖显式化；定理陈述与证明分文件、独立维护链接，解决的是验证式编程中"改一行声明、全库重编译"的资源瓶颈；自然语言定理描述解决的是"已证明结果无法被检索复用"的问题，缩短证明路径。三者都是通用 harness 模式（任务 DAG、编译/构建加速、语义索引）在形式化领域的具体化。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:124-136]

**Lean 编译器充当了无限耐心的 verifier 与训练信号的双重角色。** 13 百万行代码、29,500 个中间定理的量级下，任何人类评审都不可能逐行核验，而 Lean 用三个标准公理提供机器级确定性——comparator 进一步确认定理表述与 Mathlib 的 FLT 表述一致，堵住"形式对了但证的不是同一个定理"的漏洞。原论文脚注中 Hales 的 Kepler 猜想证明（12 人评审后只能给出"99% 确信"）与 FLT 形成对照：形式化把验证从社会过程变成了计算过程。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:140-144]

**人类输入的角色是"方向仲裁"而非"解题"。** 全程人类贡献仅限 Tianyi Peng 的少数高层指令（"Jacobian as a scheme 优先级高""尽快推 Mazur 定理"）——本质是利用人类对 Wiles 证明结构的先验知识来调整优先级，而非参与任何具体推导。Claude 走的是 Darmon-Diamond-Taylor 简化路线，且承认其证明很可能"远比必要的更长"（对比简洁且经过充分评审的 Mathlib）——AI 形式化当前的优势是"证出来"，而非"证得优雅"。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:100-104]

**成本曲线的斜率比绝对水平更值得关注。** 6 billion output tokens 换 FLT（此前预计需数年），而三个消费级 Claude Max 订阅三天完成 Vinogradov 三素数定理形式化——说明形式化成本已从"机构级项目"降到"个人订阅+合适 scaffold"级别。这与 [[entities/lean-scaling|Lean 软件扩展定律]] 的规模-成本框架吻合：瓶颈正在从人类专家工时转移到 token 支出，而 token 成本仍在快速下降。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:140-160]

## 实践启示

- **给长周期多 agent 任务先建外部状态图，再谈模型能力。** 若没有 DAG 化的任务真相源，并行 agent 会重复劳动、互相冲突直至崩盘——先投入 scaffold 设计，往往比换更强模型收益更大。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:116-128]
- **把"验证器在环"做成架构而非事后检查。** Lean 即时判定每一步的正确性，agent 因此可以无限试错而不积累隐性错误；任何 AI 代码/推理管线都可类比：选一个机器可判定的 oracle 并让它在生成过程中持续反馈。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:86-94]
- **分离声明与实现以加速验证循环。** Prove2Me 把定理陈述与证明拆到不同文件、独立维护链接来提速编译——软件工程中的接口/实现分离在 agent 时代有了新动机：缩短反馈回路本身就是生产力。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:130-136]
- **失败产物可资产化。** ~7% 的最终证明代码来自失败尝试——多 agent 系统应设计失败回收机制（DAG 中可复用的中间定理天然支持这一点），而非把每次失败当作纯成本。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:116]
- **用人类做高层仲裁，用形式化工具做机器级验证。** 对 AI 产出的可校验声明（数学、代码、合规推理），最经济的信任管线是：人类只校准方向与表述，正确性交由编译器/形式化工具闭环——Buzzard 预言的"为每份面向人类的写作同时产出形式化证明"可能成为 AI 内容质量保障的标配模式。^[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026.md:148-152]

## 相关实体

- [[entities/lean-scaling|Lean Software Scaling Laws]]
- [[entities/formatheoria-cfsg-ai-formal-verification-lean-2026|FormaTheoria：AI 辅助有限单群分类形式化]]
- [[entities/agent-formal-verification-ai-code|Agent 形式验证]]
- [[entities/claude-code-multi-agent-harness-source-analysis|Claude Code Multi-Agent Harness]]

→ [[raw/articles/formalizing-fermats-last-theorem-claude-lean-anthropic-2026|原文存档]]
