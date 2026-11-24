---
title: "DeepSeek点燃大模型效率之争 阶跃火速接棒 JetSpec让大模型解码速度 量子位"
created: 2026-08-01
updated: 2026-11-23
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md|原文存档]]

## 摘要

文章对比了几乎同期发布的两篇投机解码论文：DeepSeek 的 DSpark 与阶跃星辰参与发表的《JetSpec：Breaking the Scaling Ceiling of Speculative Decoding with Parallel Tree Drafting》。DSpark 关注高并发推理服务中的验证效率（展示生产系统仍有 Flash 模型 60%-85%、Pro 模型 57%-78% 的提速空间，Qwen3-8B/AIME25 上把平均接受长度从 DFlash 的 4.07 提到 5.01）；JetSpec 则从 Draft 生成本身入手，把因果性直接融入并行草稿头生成路径条件化的草稿树：在 Qwen3-8B 上相比标准自回归解码最高实现 9.64 倍端到端解码加速，MATH-500 上一次验证平均可接受 10.76 个 token，HumanEval、LiveCodeBench、MT-Bench 分别加速 7.12 倍、7.67 倍、4.58 倍。JetSpec 作者含阶跃 CEO 姜大昕与 CTO 朱亦博，一作 Lanxiang Hu 为 UCSD 博士生（阶跃实习期间完成），团队此前还合作过 PD 分离开山论文 DistServe；文章认为两者共同说明推理效率正成为 Agent 规模化落地的基础变量。^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md]

## 关键要点

- JetSpec 核心机制：因果并行草稿头生成路径条件化草稿树，深层节点依赖同一分支上更早生成的 token，把更大草稿预算转化为更长可接受前缀
- 关键数据：接受长度从投机预算 16 时的 7.23 提升到预算 128 时的 9.82（同预算下 DFlash 为 7.34、DDTree 为 8.66）；AIME25 上草稿深度 1 接受率约 99%、深度 8 仍保持约 50%，有效逐 token 接受率 alpha_eff 约 93%
- 理论背景：草稿成本下降后瓶颈转移到接受率——逐 token 接受率从 0.85 提到 0.95 即可显著提高最大理论加速比，这决定了 DSpark（吞吐导向、置信度调度验证）与 JetSpec（延迟导向、追求接受长度）的互补定位
- 团队背景：JetSpec 作者含阶跃 CEO 姜大昕、CTO 朱亦博（AI Infra 专家）；一作 Lanxiang Hu 就读 UCSD；阶跃与 UCSD 此前合作过 PD 分离代表作 DistServe，该架构已被 NVIDIA TensorRT-LLM、SGLang、vLLM 采用
- JetSpec 是阶跃 Flash 模型叙事（Step 3.5 Flash → Step 3.7 Flash，面向 Agent 场景的高效智能）在推理算法层面的拼图；项目开源在 GitHub（hao-ai-lab/JetSpec），论文编号 arXiv 2606.18394

## 深度分析

### 因果性是并行草稿的「通行证」，而非「加速器」

投机解码的经典困境在草稿成本骤降后发生了一次结构反转：EAGLE 一类自回归草稿用串行换质量，DFlash 用一次前向的多位置并行预测把草稿成本压到近乎为零^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:39-43]，但便宜本身不产生收益——并行候选若彼此不一致，验证阶段就会成批拒绝^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:43]。JetSpec 的核心判断是：当草稿几乎免费，瓶颈完全转移到接受率上，而接受率的敌人恰恰是并行化丢弃的因果依赖。它不做「先并行、后修正」的补救，而是把因果性直接融进生成路径——深层节点条件化于同一分支上更早生成的 token，让树状草稿天然携带前缀一致性^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:140-143]。同一认知在 DSpark 身上以另一种形态出现（Markov 修正头 + 置信度调度）^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:132-138]，Domino 亦以因果修正为卖点^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:57]：三项同期工作独立收敛于同一点，说明「因果一致性」已从实现细节升格为下一代投机解码的设计公理。

### 加速比公式把优化目标从「草稿长度」改写为「接受率」

经典投机解码加速比由逐 token 草稿成本比 c、接受率 α 与草稿长度 γ 共同决定^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:45-47]。代入低草稿成本的现实数值后结论反直觉：c 再小，若 α 只有 0.85，最大理论加速比仍不足 5 倍；而把 α 从 0.85 提到 0.95，收益立现^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:49-51]。JetSpec 的实证曲线正是这条理论的注脚——AIME25 上深度 1 接受率约 99%、深度 8 仍有约 50% 的生存概率 q₈^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:146-148]，拟合出的有效逐 token 接受率 α_eff 约 93%，显著高于 DFlash^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:150-154]。这意味着工程上「往深了做梦」（预算 16→128，接受长度 7.23→9.82，同预算下超过 DFlash 的 7.34 与 DDTree 的 8.66^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:36]）之所以可行，前提是每个深度位点的接受率曲线足够平缓。优化焦点的迁移解释了 DSpark 与 JetSpec 的分野：前者承认接受率受限，用置信度调度把有限验证预算花在刀刃上（吞吐导向）^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:53-55]；后者用充足 FLOPs 买接受率，把草稿预算兑换成更长可接受前缀（延迟导向）^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:61]。

### 双雄并立是同一条 SLO 边界的两侧，不是竞品

把两篇论文放在同一坐标系里看，它们不是相互替代而是覆盖了吞吐—延迟帕累托前沿的两端：DSpark 面向高并发、预算受限服务，在不增加每请求验证成本的前提下抬升整体吞吐^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:55]；JetSpec 面向低并发、SLO 驱动的场景，用空闲 FLOPs 换每用户生成速度（图 3 中多种代码/数学任务的接受长度约 10–11 token）^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:127]。文章给出的下一步也顺理成章：一个动态服务框架按并发与预算在两种模式间切换，同时推进边界两端^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:155-159]。对推理平台而言，这暗示未来的草稿策略应是可插拔的运行时组件，而不是引擎里的一个固定开关。

### 推理效率成为 Agent 时代的基础变量，论文署名即产业卡位

文章的产业判断是：模型能力仍重要，但决定 Agent 能否规模化落地的基础变量正在变成推理效率——Agent 高频、长链路、持续调用的负载形态，让「答得对」之外多了「出得快、出得便宜」的硬约束^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:24]。署名结构佐证这不是论文游戏：JetSpec 作者含阶跃 CEO 姜大昕、CTO 朱亦博（AI Infra 专家），一作 Lanxiang Hu 为 UCSD 博士生（阶跃实习期间完成），团队与 UCSD 前作正是 PD 分离开山论文 DistServe——该架构已被 TensorRT-LLM、SGLang、vLLM 三大主流引擎采纳^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:26]。DeepSeek 侧 DSpark 署名梁文锋^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:26]。两大基座厂掌门人同时出现在「草稿便宜之后怎么办」这一细分问题上，等于公开确认推理效率竞争已进入基座厂主赛道；JetSpec 同时是阶跃 Flash 模型叙事（Step 3.5→3.7 Flash，面向 Agent 的高效智能）在算法层的拼图^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:24]。

## 实践启示

1. 选型投机解码方案先问流量形态：高并发吞吐敏感选 DSpark 式「并行主干 + 置信度调度验证」^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:53-55]；低并发延迟敏感（交互式 Agent、编码助手）选 JetSpec 式「因果树草稿 + 大预算换接受长度」^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:61]。
2. 评估草稿方案时别只看端到端加速倍数，直接测「深度—接受率」曲线：深度 1 处约 99%、深度 8 处仍约 50% 的平缓衰减才是能把预算从 16 加到 128 的本钱^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:146-148]。
3. 排查「草稿开了但收益不及预期」时，按公式定位短板：先算 c 是否已足够低，再测 α——c 低而 α≈0.85 时加速比天花板不足 5 倍，此时优化接受率（换树草稿/加因果修正）比继续加长草稿更有效^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:45-51]。
4. 引擎层面优先选择已采纳 PD 分离的栈（vLLM、SGLang、TensorRT-LLM 均已支持 DistServe 架构）^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:26]，为后续接入 JetSpec 类并行树草稿与 DSpark 类预算调度留出组合空间。
5. 做 Agent 容量规划时把「解码速度」当作一阶变量而非锦上添花：长链路 Agent 的体验与成本由单次推理延迟和吞吐共同决定，推理效率竞赛（DSpark/JetSpec 同期发布）意味着这条曲线仍在快速移动，固定基准很快过时^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:24]。
6. 跟踪开源实现而非只读论文：JetSpec 项目页、arXiv 2606.18394 与 GitHub 仓库 hao-ai-lab/JetSpec 均已公开^[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md:161-165]，可直接在自有负载上复测 q-曲线与 α_eff 再做引入决策。

## 来源


- 原文: [[raw/articles/2026-06-30-DeepSeek点燃大模型效率之争-阶跃火速接棒-JetSpec让大模型解码速度-量子位.md|DeepSeek点燃大模型效率之争 阶跃火速接棒 JetSpec让大模型解码速度 量子位]]
