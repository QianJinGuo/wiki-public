---
title: "PyroDash: 成本感知的 token 级大小模型协同推理"
created: 2026-09-14
updated: 2026-09-14
type: entity
tags: [inference, cost-optimization, small-language-model, routing, post-training, grpo, open-source, llm-engineering]
confidence: 0.72
provenance_state: extracted
sources: [raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026]
---

# PyroDash: 成本感知的 token 级大小模型协同推理

火思动力（Pyromind）开源的 **PyroDash** 把「什么时候调用更贵的模型」从部署期的工程配置变成了**可训练、可调参、可复现的模型能力**：由小模型在生成过程中自己判断是否发出求助信号，一个请求最多触发一次大模型调用。论文为《PyroDash: Cost-Efficient Token-Level Small-Large Language Model Collaborative Inference》（arXiv 2607.20327），模型权重、数据集（EasyHard-24k）与评估代码均已开源。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

## 架构：小模型优先、至多一次、单向不返回

系统有三个部件：一个可训练的小模型、一个保持冻结的大模型，以及负责执行交接的协同引擎（Collaborate Engine, CE）。请求进入后 CE 先把问题连同固定系统提示交给小模型，小模型以流式方式生成；若能独立完成则直接返回，全程不产生任何大模型调用。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

一旦小模型判断自己接不住，就输出专属控制 token（论文记作 `τ_off`）；CE 检测到后立刻停止小模型解码、剥掉控制 token，把原始问题与此前形成的部分推理轨迹一次性打包交给冻结的大模型续写。控制权不再交回小模型，因此**一个请求最多只会产生一次大模型 API 调用**，交接是单向且不可回退的。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

论文给出的 GSM8K 案例说明了交接的语义：小模型先把柠檬树问题化简为年净收入算式 `7×1.5−3=7.5` 再发出 `τ_off`，大模型接手时看到的不再是自然语言题面而是一个已算好的中间量，于是只需列出 `7.5n > 90` 解得 n>12。交接不等于放弃——小模型已完成的工作作为上下文进入大模型续写，而不是被丢弃重来。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

对企业更实际的性质是：CE 拼接的是上下文，所以对大模型几乎没有要求——开源闭源皆可、不需要梯度、不暴露内部 logits，CE 需要的只是 token 检测、上下文打包和一个标准文本补全接口。被训练的那一端选 4B，是因为该尺寸能在端侧（如一台 Mac）上轻松跑起来。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

## 三阶段训练：把账单写进奖励函数

难点在于小模型如何知道该在哪一步举手——原始词表里根本没有 `τ_off`，更谈不上把它与「能力边界在这里」关联起来。PyroDash 用一条三阶段管线解决，且全程只训练小模型。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

第一阶段是控制 token 的嵌入学习：`τ_off` 的输入与输出嵌入初始化为一组锚点 token（句号、换行、结束符这类自然断点）嵌入的均值加高斯噪声（σ=0.1），把新 token 放在嵌入空间的「边界」附近；训练时嵌入层与输出头全量微调、主干挂临时 LoRA 联合更新，训完丢弃适配器。第二阶段是行为冷启动，在两份语料混合上做一轮 SFT，让同一模型同时学会独立推理与协同模式（无协同提示时不含 `τ_off`，保住原有独立推理行为，也避免鼓励无事就叫大模型）。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

第三阶段是分水岭：用**成本感知的 GRPO** 做策略对齐，训练时执行与推理完全相同的单向交接路径——一旦 rollout 出现 `τ_off`，训练环境复现打包、终止、调用冻结大模型、拼接联合输出的完整流程，再按最终答案是否正确以及这条联合轨迹的成本给奖励。奖励形式是准确度减去 λ 倍的归一化成本，归一化成本 = 本次协同推理绝对开销 ÷ 同题完全交给大模型的基线开销。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

成本口径是关键细节：不仅含小模型自己生成的 token，还含交接时小模型此前全部输出作为 prefill 进入大模型的输入成本，以及大模型后续解码成本——**小模型交接前多写的每个 token 都要在账单上被算两次**。归一化成本 <1 说明协同比纯大模型便宜，>1 说明冗余生成或无效交接反而抬高账单，该轨迹会在组内相对优势里吃亏。λ 越小越愿意用大模型换准确度，越大越强调压低调用与成本。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

## 实验结果

实验以 Qwen3.5-4B 为小模型、GLM-5.2-FP8 为大模型，在五个数学推理基准上评估，计价基准为小模型输入/输出 0.05/0.08 美元每百万 token、大模型 0.90/2.86 美元每百万 token。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

- **省钱端（λ=0.6）**：小模型平均每 100 道题才叫一次大模型，五基准估算总成本从纯大模型的 49.36 美元降到 1.78 美元（不到原来的三十分之一），平均准确率仅比纯大模型低约 3 个百分点。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]
- **效果端（λ=0.05）**：平均准确率 64.04%，反而比纯 GLM-5.2-FP8 的 57.68% 高出 6.36 个百分点，同时成本还低两成。准确率反超的原因是此时几乎每题都发生交接——大模型接手时看到的不是原始题面而是小模型已整理好的推理，改变了它进入问题时的状态。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]
- **中间档（λ=0.1）**：平均准确率高于 RouteLLM 与 GlimpRouter 两条主流路由基线，成本只有它们的十分之一到七分之一；两条基线都把超过 75% 的解码 token 交给大模型，而 PyroDash 只在必要的那一刻交出去一次。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]
- **消融**：只做完前两阶段监督微调的版本平均准确率停在 46.25%，加上第三阶段成本感知 GRPO 后跳到 64.04%——冷启动教会的是行为，强化学习学到的才是判断。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

## 与请求级路由的区别

主流做法（FrugalGPT 级联、RouteLLM 偏好数据训练的路由器、Hybrid LLM 难度预测）共同点是**决策发生在解码开始之前**：阅读输入→判断难度→把请求路由给某个模型。问题是真实任务的难度往往不写在输入里——一道题可能开头只是信息提取、中途才需要复杂判断，而请求级路由在解码前就把整道题分配掉了，之后无论生成过程中出现什么新信息都没有调整余地；级联虽可在打分后升级，但升级意味着从头重新生成，而不是从失败的位置继续。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

把决策移进生成过程的方案此前已有 CITER（成本感知 MLP 逐 token 决策）、Co-LLM（把交替解码学成隐式模型选择）、RelayLLM（小模型请求一段有界长度内容后交回控制权）、GlimpRouter（免训练、用每个推理步首 token 熵做不确定性信号）。粒度更细了，代价也随之出现：逐 token 评估路由器每步都有额外开销；反复切换要求服务循环同时协调两个模型并维护各自解码状态；在无状态商用 API 下每次重新介入都要把不断增长的上下文重发一遍，几段很短的大模型输出会变成数次完整的 prefill 计费。此外 Co-LLM 没有显式成本目标，RelayLLM 惩罚的是大模型 token 占比而不是真实账单里分开计价的 prefill 与 decoding。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

投机解码解决的是另一个问题：从 Leviathan 的原始方案到 SpecInfer、Medusa、MagicDec，这条线优化延迟与吞吐，且大模型要对每个草稿块打分、通常需要访问其输出概率甚至权重，大模型全程在场，小模型没有独立完成简单请求的机会。PyroDash 针对这些代价而来——决策内化在模型里（不挂外置路由器）、交接单向且至多一次（不来回切）、只用标准文本续写接口（不碰大模型内部）。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

## 深度分析

### 三个变化：决策位置、决策主体、优化目标

过去两年推理侧优化就三条路——缩小模型（量化、蒸馏）、加快解码（投机解码、PagedAttention）、分流任务（各类路由），三者共享同一默认前提：推理是一次性的、原子化的，一个请求由某一个模型从头到尾生成完，系统只负责选模型与让模型跑得更快更省。PyroDash 动摇的正是这个前提，可拆成三个变化：决策位置从请求之前移入生成过程之中；决策主体从外部路由器变成模型自身；优化目标从 token 占比、FLOPs、延迟这些代理指标换成按 prefill 与 decoding 分开计价的真实账单。三者合起来指向一个结论——**推理可以通过后训练持续优化，成本也可以成为训练目标的一部分**。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

### 为什么「几乎每题都交接」反而更准

λ=0.05 档的效果反超是本文最有信息量的数据点：当交接接近必然时，大模型的输入从原始题面变成了小模型整理过的推理（已化简的算式、已换元的线性方程、已写出的密度公式）。这实际上把协作关系从「谁来答」改写成「谁先答、答到哪一步」，大模型承担的是一次**带脚手架的续写**而非冷启动解题。副作用是成本口径必须把交接前的 prefill 计入，否则这套机制的账会算不平——奖励函数里那个「算两次」的细节正是该效应的定价。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

### 与「小模型是 Agent 未来」路线的位置

作者把 2025 下半年以来「AI 下半场」的讨论概括成一道选择题：继续预训练走向一个极庞大的中心化模型，或用大量不同能力层次的智能体组成最终的 ASI（Agent 蜂群）。他选后者，理由是「需要被 AI 解决的问题和场景无穷无尽、且不是静态的」，用有限参数模型泛化无限场景在 Transformer 架构上挑战太大。这一判断与英伟达研究院《小语言模型才是智能体 AI 的未来》、伯克利 BAIR「从模型到复合 AI 系统的转变」、以及 Gartner「可预见未来不存在一个可靠又经济、适用于所有场景的模型」的判断同向。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

值得注意的是作者并不认为基座演进会威胁这套架构：「基模越好，对 Worker Model 的压力越小」——PyroDash 是在基座之上打的一个补丁，基座越强反而会产生更多路由、协同与成本优化的强化学习需求。这一点与 [[entities/xunfei-spark-token-factory-model-routing-cost|讯飞星火 Token 工厂]] 的成本工程视角、以及 [[entities/state-of-routing-in-model-serving|模型服务路由现状]] 的请求级路由图景互补：前者做的是「把便宜模型用满」，PyroDash 做的是「让模型自己决定何时不用便宜模型」。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

### 边界与开放问题

评估集中在五个数学推理基准，且 `τ_off` 的语义是「这条推理轨迹超出当前能力」，对开放式生成、工具调用、长程 Agent 任务是否能定义同类「自然的断点」尚无公开证据。4B 小模型的端侧假设也意味着协作收益与模型能力差强相关：若小模型过弱，交接率会向 100% 漂移并退化为「多付一次 prefill 的大模型调用」；若过强，则 λ=0.6 档的成本优势可能变成误差放大。作者本人也把「生产场景里是否真的解决了特定问题，而不单纯是 Benchmark 上的分数」列为下一步的验证目标。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

## 实践启示

- **把成本写成训练期目标而非部署期旋钮**：λ 提供了一条从「省钱」到「效果」的连续可调谱系（同一套权重换 λ 即换性格），比在推理网关里手写路由阈值更容易复现与归因。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]
- **交接必须单向、有上限**：至多一次 + 不返回控制权，使系统的成本上界与状态复杂度都可枚举；反复切换的方案要在预填计费上付出隐性代价。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]
- **异构模型协作可以走「标准文本接口」**：CE 只依赖 token 检测与上下文打包，这使该范式对闭源大模型同样可用，也便于替换任一端的模型供应商。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]
- **数据构造用「模型相对难度」**：EasyHard-24k 的难易标签由基础小模型实跑结果定义（答对= easy，答错但能重构正确轨迹 = hard），比静态基准标签更贴合协作语义；同时必须用大模型重构正确思维链再插入交接位置，否则会把失败路径训回模型。^[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026.md]

## 相关实体

- [[entities/state-of-routing-in-model-serving]] — 请求级路由的整体图景，与本文的 token 级协作形成对照
- [[entities/cursor-router-production-model-routing-2026]] — 生产环境模型路由的工程实现
- [[entities/ibm-research-model-routing-optimization-2026]] — 路由优化研究视角
- [[entities/xunfei-spark-token-factory-model-routing-cost]] — token 成本工程实践
- [[concepts/inference-optimization]] — 推理优化总览（量化/解码加速/任务分流三条主线）
- [[concepts/ai-cost-optimization-framework]] — AI 成本优化框架

→ [[raw/articles/pyrodash-token-level-small-large-collaborative-inference-2026|原文存档]]
