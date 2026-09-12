---
title: "Co-Evolving Harnesses and Models（arXiv 2609.09134）：模仿专家轨迹会破坏 model–harness fit，on-policy 纠错才让两者可组合"
description: "Salesforce 2026-09-08 arXiv 论文：7 个企业 Agent 任务上先进化 harness 再用专家轨迹 LoRA-SFT 弱模型，结果全线倒退 4-30 分（均值 -14.9）——模仿转移了知识却让弱模型套用无法执行的专家规划风格，破坏了 harness 与模型的匹配度。改用 on-policy expert correction（meta-MLE agent 定位失败轮次、专家只改写该轮）后均值 78.0%→79.7%，planning 失败占比仅 +0.7（模仿为 +13.5）"
created: 2026-09-12
updated: 2026-09-12
type: entity
tags: [agent, harness, harness-engineering, harness-evolution, co-evolution, on-policy-correction, imitation-learning, lora-sft, agentic-rl, post-training, enterprise-agent, model-harness-fit, gepa, arxiv, arxiv-2609-09134, salesforce]
sources: [raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134]
confidence: 0.85
provenance_state: extracted
related:
  - entities/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528
  - entities/不止autoresearch策略与harness共进化evotrainer跑通自主闭环
  - entities/agent-self-evolution-evaluator-bottleneck
  - entities/on-policy-distillation-vs-offline-distillation-loster
  - entities/xopd-on-policy-distillation-landscape-banana-2026
  - entities/gepa-reflective-prompt-evolution-iclr2026-oral
  - entities/seed-self-evolving-opd-long-horizon-agent-rl-tsinghua-zju-2026
  - concepts/harness-engineering
  - concepts/harness-tool-design-evolution
  - concepts/harness-component-expiry-build-to-delete
---

# Co-Evolving Harnesses and Models：模仿会破坏 model–harness fit，on-policy 纠错才让两者可组合

Salesforce 团队（Zhou Yu 等 11 人）2026-09-08 提交的 arXiv 论文，回答了一个此前被默认「两把杠杆可以叠加」的问题：**当一个 harness 已经被针对某个弱模型进化过之后，再对这个模型做权重微调，应该怎么做？** 答案反直觉——**用更强专家模型的完整成功轨迹做模仿式 SFT 会让性能全线倒退，7 个任务无一例外，平均 -14.9 分，最高 -29.9 分**；倒退的原因不是知识没学到，而是模型-框架匹配度（model–harness fit）被破坏。改为 on-policy expert correction 后，两个杠杆才真正可以组合（78.0% → 79.7%）。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

论文的核心判断可以浓缩成一句可迁移的设计原则：**harness 一旦针对某个模型优化过，它就与该模型的规划（planning）与执行行为耦合了；此后的权重更新必须「保持」而不是「无意中破坏」这种匹配。** 因此给弱模型的监督信号应当落在学生模型自己走过的状态（on-policy）上，而不是整条专家轨迹的批发式模仿。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 实验设置

- **任务套件**：7 个可客观验证的企业级 Agent 任务——payroll auditing（薪酬审计）、budget approval（预算审批）、stock alerting（股票告警）、IoT anomaly detection（异常检测）、browser automation（浏览器自动化）、website management（网站管理）、code refactoring（代码重构）。任务、环境与 harness 优化框架沿用 Yang et al. (2026)。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **Harness 进化方式**：GEPA 风格搜索（Agrawal et al., 2025），由 `gemini-3.1-pro-preview` 扮演 meta-agent，提出 failure-driven 的编辑，只有验证集性能提升才被保留；可编辑对象包括 system prompt、工具集、执行 hooks、上下文管理、sub-agent 配置。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **两处环境改动**：① 需要 MCP 工具时 Agent 不能直接改任务数据库；② tool-call 报错暴露给优化器。作者声明因此部分绝对值与 Yang et al. 不完全可比。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **模型对**：弱学生 `qwen3-coder-30b-a3b`，强专家 `gemini-3.1-pro-preview`；第二组消融换 `gemma-4-26b-a4b-it`（reasoning model）。模型适配用 LoRA-SFT（H100/H200），专家轨迹被转换成学生的 chat / tool-call 格式；训练+验证用于优化，test split 全程留出，报告三次运行的均值。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 发现一：进化后的 harness 向上迁移，暴露「模型侧余量」

只用一个弱 Qwen 进化出来的 harness，换更强的 Gemini 来用反而更有效：Qwen 从 base harness 的 **29.2%** 提升到进化 harness 的 **78.0%（+48.8）**；Gemini 在同一环境下从 84.4% 提升到 **93.6%（+15.6）**。也就是说，进化后的 harness 里已经藏着弱模型尚未兑现的能力，专家模型在同一个 harness 上比弱模型高 15.6 分——这正是「专家可以教弱模型」的自然诱因。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

轨迹层面的证据表明专家是「真在用」这些进化出来的组件：专家在 93.6%–100% 的 rollout 中触发了几乎每一项进化编辑，其中 domain-computation recipe 的使用率 94.2%，而 base 模型几乎不用（30.8%）。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 发现二：全轨迹专家模仿破坏 model–harness fit

看起来最自然的配方——**先进化 harness，再用专家轨迹微调弱模型**——在进化 harness 下**全线失败**：Qwen 均值从 78.0% 掉到 **63.1%（-14.9）**，7 个任务全部倒退，幅度从异常检测的 -4.2 到薪酬审计的 **-29.9**（网站管理 -20.3、浏览器自动化 -16.0）。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

两个消融把原因锁死在「模仿 × harness 进化的交互」上：

- **消融 I（同配方、基线 harness）**：在未进化的 base harness 下，同一批专家轨迹让 Qwen 从 29.2% 提升到 **35.5%（+6.3）**。同样的教学信号在 base harness 下有用、在进化 harness 下有害 → 问题不在模仿本身。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **消融 II（换模型家族）**：把弱学生换成 `gemma-4-26b-a4b-it`，在 Webarena 任务上复现同一模式——harness 进化让 Gemma 46.7%→55.6%（+8.9），同一 harness 也向上给 Gemini 带来 71.1%→81.1%（+10.0）；但用专家轨迹微调 Gemma 后掉到 **41.1%**，比它自己的进化-harness 基线低 14.5 分，甚至比它默认 harness 的基线还低 5.6 分。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

**机制诊断**：退化既不是知识丢失、也不是 scaffold 使用减少——SFT 之后两者都上升了（domain-computation recipe 使用率 30.8% → 76.1%）。真正变化的是**规划策略漂移**：弱模型抄了专家的规划策略，却缺乏执行该策略的能力，同时不再匹配那个围绕它「原生规划风格」进化出来的 harness。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 发现三：On-policy expert correction 让两个杠杆可组合

替代方案是**把教学信号放到 on-policy 状态上**：由 self-directed 的 MLE agent 端到端驱动一条数据合成流水线——从弱模型自己在进化 harness 下的 rollout 出发，定位每个失败 rollout **出错的那一个 turn**，由专家模型**只就地改写这一个 turn**，其余步骤原样保留；然后在「最小编辑后的轨迹」上 LoRA-SFT 弱模型。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

结果（Table 1 第 7 行）：均值 **78.0% → 79.7%（+1.7）**，7 个任务无一低于进化-harness 基线——网站管理 +5.6、股票告警 +2.2、代码重构 +2.2、异常检测 +1.7、浏览器自动化 +1.2；两个 harness 已接近饱和的任务在噪声范围内（预算审批 -0.6、薪酬审计 -0.4）。对比直接模仿的全线 -14.9，on-policy 纠错是「harness 尚有空间处加能力、harness 已近天花处保能力」。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

失败构成的分解（Table 3）解释了增益为何安全：

| 失败类型 | base @ h₀ | SFT-imit @ h₀ | base @ h* | SFT-imit @ h* | SFT-corr @ h* |
|---|---|---|---|---|---|
| 知识缺失 | 61.0% | 54.4% (-6.6) | 46.2% | 44.5% (-1.7) | 43.2% (-3.0) |
| 规划缺陷 | 0.9% | 11.5% (+10.6) | 1.1% | 14.6% (+13.5) | 1.8% (+0.7) |

即：on-policy 纠错拿到了与模仿同样的知识改善（46.2% → 43.2%），却没有模仿那步的规划坍塌（+0.7 vs +13.5）。整体硬失败率也从 28.9% 降到 26.8%。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 关键数据一览（Table 1，7 任务均值）

| # | 模型 | Harness | 均值成功率 | 相对变化 |
|---|---|---|---|---|
| 1 | qwen3-coder-30b-a3b | h₀ (base) | 29.2% ± 0.61 | — |
| 2 | qwen3-coder-30b-a3b | h* (evolved) | **78.0%** ± 0.97 | +48.8 |
| 3 | gemini-3.1-pro-preview | h₀ | 84.4% ± 0.85 | — |
| 4 | gemini-3.1-pro-preview | h* | **93.6%** ± 0.46 | +15.6（相对行 2） |
| 5 | qwen (SFT-imitation) | h* | 63.1% ± 0.81 | **-14.9** |
| 6 | qwen (SFT-imitation) | h₀ | 35.5% ± 1.06 | +6.3（相对行 1） |
| 7 | qwen (SFT-correction) | h* | **79.7%** ± 0.67 | +1.7（相对行 2） |

^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 可迁移的设计原则

- **一旦 harness 为某个模型优化过，harness 与权重更新就不再独立。** harness 会与该模型的规划/执行行为耦合，后续的权重更新必须保持这种匹配，而不是无意破坏它。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **harness 进化之后，监督信号应落在学生自己访问过的状态上（on-policy），而非整条专家轨迹。** 具体形式可以很轻——只定位并改写失败的那一个 turn。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **「向上迁移」是判断模型侧余量的廉价探针**：把一个强模型放进弱模型进化出的 harness，若它明显更强，说明 harness 的 scaffold 里还有弱模型未兑现的能力，值得做模型侧适配。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **成本友好**：单轮训练不到一小时，因此 on-policy 纠错可以安全地叠进「harness → 模型」的迭代共进化循环里。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **失败归因可作为回归探针**：把硬失败拆成「知识缺失 / 规划缺陷」两类并逐轮对比，能直接看出一次权重更新是在补能力还是在破坏匹配（本论文中 planning 桶从 1.1% 暴涨到 14.6% 就是模仿破坏匹配的信号）。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 局限与展望

作者明确承认 on-policy 纠错**并未完全消除弱模型与专家之间的差距**（推断受限于轻量 LoRA 的能力上限），只是在不引入反向退化的前提下拿到增量。此外数据来自 7 个企业任务、单架构家族的 3 次运行均值，作者自述「保持上升迁移结论的稳健性仍需更多模型对验证」。未来工作两条：① 把 RL 与 on-policy 专家指导结合，在仍有空间处继续推弱模型；② 让 harness 进化**知道**模型后续会被微调，从而把两条杠杆联合优化而不是串行执行。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 与库内相关工作的关系

- **与 EvoTrainer（策略与 harness 共进化）互补**：EvoTrainer 走的是 RL 路线做 policy+harness 共进化，本论文走的是「弱模型 LoRA-SFT + 专家 on-policy 纠错」，并给出了「不该怎么做」的负向结论（全轨迹模仿会破坏 fit）。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **与 on-policy distillation 家族同源**：本论文的 on-policy expert correction 在方法上属于「在学生自己访问的分布上给监督」这一大类，与库内 on-policy distillation 相关实体共享动机，但监督粒度更细——不是逐 token 蒸馏，而是**只改写失败的那一个 turn**。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]
- **对应「Agent 自进化的评估器瓶颈」**：论文用「知识缺失 / 规划缺陷」两桶分解来做失败归因，本质上是把评估从「分数」推进到「失败类型构成」，这正是自进化闭环里最难自动化的一环。^[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134.md]

## 相关

- [[entities/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528|Agent Lightning v1：harnessed agentic RL]]
- [[entities/不止autoresearch策略与harness共进化evotrainer跑通自主闭环|EvoTrainer：策略与 Harness 共进化]]
- [[entities/agent-self-evolution-evaluator-bottleneck|Agent 自进化：评估器瓶颈]]
- [[entities/on-policy-distillation-vs-offline-distillation-loster|On-policy vs Offline Distillation]]
- [[entities/xopd-on-policy-distillation-landscape-banana-2026|XOPD：on-policy distillation 全景]]
- [[entities/gepa-reflective-prompt-evolution-iclr2026-oral|GEPA：反射式 prompt 进化]]
- [[entities/seed-self-evolving-opd-long-horizon-agent-rl-tsinghua-zju-2026|SEED：长程 Agent RL 自进化 on-policy distillation]]
- [[concepts/harness-engineering|Harness Engineering]]
- [[concepts/harness-tool-design-evolution|Harness 工具设计演进]]
- [[concepts/harness-component-expiry-build-to-delete|Harness 组件保质期与 Build-to-Delete]]

→ [[raw/articles/co-evolving-harnesses-and-models-on-policy-correction-arxiv-2609-09134|原文存档]]
